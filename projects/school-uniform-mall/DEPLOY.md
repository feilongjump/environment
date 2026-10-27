# school-uniform-mall 部署档案（唯一事实源）

> 部署相关改动在 environment 仓库的本项目专属会话中进行；任何会话开场先读本文件。服务器侧对应路径：`/var/environment/projects/school-uniform-mall/`。

## 现状

- **通道**：② GitHub Actions 全自动（push main 自动 + workflow_dispatch 手动兜底；2026-10-18 接入当日验证后开启）
- **项目仓库**：`feilongjump/school-uniform-mall`（本地 `C:\Users\long\Code\school-uniform-mall`）
- **形态**：B 单二进制（管理后台前端 go:embed 内嵌进 `sum`，页面 + API 同端口；小程序产物不入服务器，走微信平台）
- **服务器**：**阿里云轻量应用服务器 47.115.207.44**（ssh 别名 `swas`，root；`admin` 为控制台登录账户）
- **服务/端口**：`sum-web`（nginx，**80/443，公网唯一入口** `https://qibu888.cn`，health：`/api/health`）→ 容器内网反代 → `sum-api`（8080，宿主仅 `127.0.0.1:9422` 调试）；防火墙放行 22/80/443，9422 可关
- **域名/证书**：`qibu888.cn`（2026-10-27 备案完成）；Let's Encrypt ECC 证书，acme.sh 装宿主 root（crontab 自动续期，webroot `/var/www/acme`，续期 reloadcmd `/root/reload-sum-web.sh` 自动重启 sum-web）；acme.sh 经 `git -c core.autocrlf=false archive` 打包传入（避 Windows CRLF）
- **数据库**：共享 postgres（env_net 直连 `postgres:5432`，库 `school_uniform_mall`；首次启动自动 goose 迁移 + 种子超管 admin/admin123，**上线后立即改密**）
- **Secrets**（school-uniform-mall 仓库）：`SSH_HOST=47.115.207.44` / `SSH_USER=deploy` / `SSH_KEY`（= `ci-school-uniform-mall@github-actions`，装在 deploy@轻量服务器，带 `restrict`）
- **服务器密钥**：`projects/school-uniform-mall/api/.env`（JWT_SECRET / DATABASE_DSN，600 deploy 属主）；根 `.env` 为 **640 root:deploy**（deploy 用户跑 compose 需读编排变量）
- **回滚**：激活段内置 `sum.old` 备份；回滚 = `ssh swas` 把 `.old` mv 回来 + `docker compose restart sum-api`
- **构建要点**：bun 构建前端 → 产物校验（`assets/` 标记防空壳包）→ CGO_ENABLED=0 交叉编译；CI 前置 verify 作业（web typecheck+lint、go test）
- **时区**：compose 已挂 zoneinfo（避 alpine 无 tzdata 跑 UTC 的坑，见根 README）——与 flowstock/otb 不同，勿对齐他们
- **微信**：config.prod.yaml `wechat.mock: true`；域名/备案/HTTPS 已就绪（2026-10-27），正式上线仅剩：微信后台登记合法域名 `https://qibu888.cn`、小程序 `TARO_APP_API=https://qibu888.cn/api/app` 构建、后端切 `mock: false` 填真实凭据

## 轻量服务器（swas）环境要点（2026-10-18 初始化）

- Ubuntu 24.04，2C / 1.6G 内存 / 40G 盘；apt 装 docker.io 29 + docker-compose-v2；补 2G swap（fstab 持久）
- **GitHub HTTPS 不通**（GnuTLS -110）：`/var/environment` 从本机经 **git bundle** 同步——本机 `git bundle create %TEMP%\env.bundle develop` → scp 到 `/root/env.bundle` → 服务器 `git pull /root/env.bundle develop`；GitHub SSH（22 端口）可达，日后需要服务器自主 pull 可配只读 deploy key
- **Docker Hub 直拉超时**：`/etc/docker/daemon.json` 配 registry-mirrors（daocloud / 1ms.run / dockerproxy），改后 `systemctl restart docker`
- 未启用项目（otb/flowstock/retail-integration）`api/.env` 已放空壳（compose 解析需要文件存在，V21 机同款做法）
- 根 `.env` / 项目 `api/.env` 密钥均服务器本地生成（openssl rand），不入库

## OPS 日志

- 2026-10-18 首次接入（四阶段 SOP，全程在 environment 会话指挥）。期间发现原定主服务器 8.163.117.200 整机不可达（全端口 + ICMP），改接入用户提供的轻量应用服务器 47.115.207.44。服务器初始化：装 docker、2G swap、bundle 克隆本仓库、根/项目 .env 生成、postgres:18 起库 school_uniform_mall、deploy 用户 + restrict CI key、GitHub Secrets 三项。排障记录：①本机 ssh 未指定 IdentityFile 导致公钥拒认（新别名 swas 已入 ~/.ssh/config）②daemon.json 首次经 printf 写坏 + systemd 重启限流误报，重写并 reset-failed 解决 ③根 .env 640 root:deploy 后 deploy 方可跑 compose。首次部署 run 39089479340 成功：sum-api healthy、/api/health 200、迁移 v6、超管种子。同日开启 push main 自动部署（run 39089790200 由 push 触发验证）。
- 2026-10-18 push 自动部署第二跑（39089790200）因项目仓库 TestGenPickupCode 偶发失败（百次采样全不重复断言，碰撞率≈0.5%/跑）；已在项目仓库改粗熵检查（≥90/100 不同值）修复，run 39089987222 全绿，自动通道复验通过。
- 2026-10-18 应用户要求试切 SUM_PORT=80（干净 URL），说明 HTTPS 时代 80 的角色（301 跳转 + ACME 验证）与 443（正式服务）分工后，用户决定暂用 9422，当日回切。结论留档：域名/备案/HTTPS 就绪时，nginx 进驻 80/443，sum-api 撤回容器内网；小程序届时以 TARO_APP_API=https://域名/api/app 切换（合法域名登记到域名级）。
- 2026-10-27 **HTTPS 上线**（域名 qibu888.cn 备案完成当日）：新增 sum-web（nginx:alpine，80 跳转+ACME / 443 证书终止反代 sum-api，挂 _shared 公共片段），acme.sh 签发 Let's Encrypt ECC 并装宿主 crontab（webroot 模式续期，reloadcmd 自动重启 sum-web）；sum-api 端口改绑 127.0.0.1:9422，公网入口收敛到 https://qibu888.cn。外网验证：/api/health 200、301 跳转、证书受信、后台页面正常。排障记录：①外网探测 closed ≠ 防火墙未开（无监听即 refused，先起 python 临时监听再探测才作数）②acme.sh 三次装不上——scp 丢执行位 / Windows 克隆 CRLF / git archive 默认仍按 autocrlf 转 CRLF，最终 `git -c core.autocrlf=false archive` 解决 ③多层引号转义连续踩坑（printf 写坏 daemon.json、compose --format、acme reloadcmd），复杂命令一律走 stdin 管道 `tr -d '\r' | bash -s`。待用户：SWAS 防火墙关 9422；微信后台登记合法域名；小程序按 TARO_APP_API 重新构建上传。
