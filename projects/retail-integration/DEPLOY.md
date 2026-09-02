# retail-integration 部署档案（唯一事实源）

> 部署相关改动在 environment 仓库的 retail-integration 专属会话中进行；任何会话开场先读本文件。

## 现状

- **通道**：③ 项目自带 deploy.sh + systemd + /opt（**计划迁移到本仓库 compose，迁移前放置不理**；迁移时在本目录补 compose.yml 并走 SOP）
- **项目仓库**：`feilongjump/retail-integration`（本地 `C:\Users\long\Code\retail-integration`，无 CI/Actions）
- **多客户实例**（同一二进制，每客户一台服务器一份配置）：

| 实例 | 服务器 | systemd 服务 | 端口 | 数据库 |
|---|---|---|---|---|
| moni | 主服务器（`ssh env`） | `retail-integration.service` | 8090 | 共享 postgres，库 `retail_integration` |
| V21 | V21 服务器（`ssh v21`） | 同名 service | 8090 | postgres，库 `retail_integration_v21` |

- **部署方式**：项目仓库 `deploy.sh`（sed 渲染 unit 模板的实例名/路径 → 上传 `/opt/retail-integration/`）→ `systemctl restart retail-integration`；unit 模板见项目仓库，注意"直接 cp 到 systemd 不会工作（占位符未替换）"
- **日志**：`/var/log/retail-integration/{stdout,stderr}.log`（非 docker logs）
- ⚠️ **V21 服务器 env 仓库锁定旧提交 9db6621，勿 `git pull`**（新提交含 flowstock include，该机无对应目录会炸），待本项迁移 compose 时一并处理
- ⚠️ root@V21 有一把来源不明 key（注释 `skp-7xv8...`），暂保留待查证

## OPS 日志

- 2026-09-02 建档（全景侦查确认双实例现状）。9-1 两实例刚部署过（unit/二进制 mtime）。
