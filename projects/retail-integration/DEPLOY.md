# retail-integration 部署档案（唯一事实源）

> 部署相关改动在 environment 仓库的 retail-integration 专属会话中进行；任何会话开场先读本文件。

## 现状

- **通道**：③ → ② **迁移进行中（2026-09-04 启动，V21 先行）**。本目录 compose.yml 已就位并入根 include；moni 待其服务商切换（百胜→易神）明朗后退役或同剧本补迁（主服务器目录随 git 对齐，**不 up**）
- **项目仓库**：`feilongjump/retail-integration`（本地 `C:\Users\long\Code\retail-integration`）
- **多客户实例**（同一二进制，每客户一台服务器一份配置）：

| 实例 | 服务器 | 运行方式 | 端口 | 数据库 |
|---|---|---|---|---|
| moni | 主服务器（`ssh env`） | systemd `retail-integration.service`（暂留通道③） | 8090 | 共享 postgres，库 `retail_integration` |
| V21 | V21 服务器（`ssh v21`） | **compose `retail-integration-api`**（2026-09-04 已切换；systemd 已 stop/disable，unit 与 /opt 保留至 9-11 备回滚） | 8090 | postgres（env_net `postgres:5432`），库 `retail_integration_v21` |

- **CI 设计（通道②变体，已与用户确认）**：push main **只构建+测试不部署**；`workflow_dispatch` 表单必选发布目标（moni/V21/both）——MEMORY.md 铁律第 1 条（目标必须显式指定）的流程强制版。dispatch 可选任意 ref（分支/tag）= 灰度与回滚（旧 tag 重发）
- **服务器布局（V21）**：`/var/environment/projects/retail-integration/api/` = `retail-server`（CI 直传，留 `.old` 回滚）+ `config.yaml`（服务器侧不入库，`database.host=postgres`）+ `configs/`；日志 `./logs` 挂 `/var/log/retail-integration`
- **secrets**（GitHub `feilongjump/retail-integration`）：`V21_SSH_HOST` / `V21_SSH_USER` / `V21_SSH_KEY`（moni 接入时再加 `MONI_*` 成对）；V21 服务器建 `deploy` 用户（docker 组）+ restrict CI key
- **切换步骤（V21，阶段 3）**：`systemctl stop && systemctl disable retail-integration` → `docker compose up -d retail-integration-api` → 验证 healthz/8090/cron 日志 → 保留 `/opt/retail-integration` 与 unit 文件一周再清。（**已于 2026-09-04 执行**，见 OPS 日志）
- **回滚（V21）**：`docker compose stop retail-integration-api` → `systemctl start retail-integration`（原 /opt 布局原样保留期间，秒级可退）
- **日志**：容器内路径不变 `/var/log/retail-integration/app.log`（宿主 `projects/retail-integration/logs/`）；systemd 时代另有 stdout/stderr.log（切换后消失，属预期）
- ⚠️ root@V21 有一把来源不明 key（注释 `skp-7xv8...`），暂保留待查证

## OPS 日志

- 2026-09-02 建档（全景侦查确认双实例现状）。9-1 两实例刚部署过（unit/二进制 mtime）。
- 2026-09-04 排查"deploy.sh 连接被拒"：根因 = 密钥会话把 `id_rsa` 从两台服务器撤除（README 密钥表 9-05 起仅 GitHub），而 deploy.sh 用裸 IP 连接不命中 `~/.ssh/config` 的 `Host v21` 别名块（IdentityFile 不生效），默认名密钥全数失效 → Permission denied。修复：`Host env 8.163.117.200` / `Host v21 8.134.137.138` 别名块加 IP 匹配（裸 IP 与别名同待遇）。同日 V21 实例补发成功（9-4 11:54 中断的那次）：binary `28b8ff5…`，**OBSERVER_MODE=false 转正**（EZR AppId 仍 `V21_test` 测试参数，转正待确认项未清）。
- 2026-09-04 通道③→② 迁移启动并完成阶段 1+2（V21 先行，moni 待服务商切换明朗）。已完成：compose.yml 入库并登记 include；V21 解除 9db6621 锁定（预建 otb/flowstock `api/.env` 空壳后 pull 至 450d3e5，compose 解析通过，postgres 无扰动）；V21 布局 `projects/retail-integration/api/`（预置当日 binary + config.yaml，DB host 已改 `postgres`）；建 deploy 用户（docker 组）+ restrict CI key，secrets `V21_SSH_HOST/USER/KEY` 已入 GitHub；项目仓库加 `/healthz` 探针 + deploy.yml（push main 仅构建；dispatch 必选目标；含 systemd 未切换闸门防 8090 撞口）。事故记录：pwsh→ssh 双层引号绞碎曾在 V21 `.env` 留下毒行 `n#` 致 compose 解析失败，已清除——**复杂远程脚本一律走"本地写文件→scp→bash"**。
- 2026-09-04 阶段 3 切换完成（用户提供 EZR 生产参数：AppId `V21` / AppSystem `E3C` / `https://open-tp.ezrpro.com`，`V21_test` 测试参数退役）。执行：参数写入 `api/config.yaml` → 新 binary（main@1314452，含 /healthz）就位留 `.old` → systemd stop/disable → compose up。验证：容器 healthy、healthz=200、coupon 游标跨 systemd→容器连续推进、同步周期零错误。**抓到一个真坑**：alpine 无 tzdata 致 TZ 失效容器跑 UTC，E3Cloud 时间戳校验拒收（同步全失败）——compose 挂宿主 `/usr/share/zoneinfo:ro` 修复（commit c26ec54）。**遗留**：① moni 仍通道③待命；② 9-11 后清理 V21 的 /opt 与 unit 文件；③ otb/flowstock 容器实测也在跑 UTC（同根因，未动，待用户决策）；④ 项目仓库 feat/tenant-boundary 分支落后 main 3 个提交（healthz+CI），提示用户合并。
