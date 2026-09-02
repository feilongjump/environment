# otb 部署档案（唯一事实源）

> 部署相关改动在 environment 仓库的 otb 专属会话中进行；任何会话开场先读本文件。服务器侧对应路径：`/var/environment/projects/otb/`。

## 现状

- **通道**：① 手动上传 + compose（**迁移通道②：用户已决定暂缓**，迁移时按 `projects/_demo/README.md` SOP 阶段 2 补 workflow 即可）
- **项目仓库**：otb-api / otb-web 仓库（本地 `C:\Users\long\Code\otb` 为非 git 工作副本）
- **形态**：A 前后端分离（`otb-api` Go 二进制 + `otb-web` nginx 静态）
- **服务器**：主服务器（`ssh env`）
- **服务/端口**：`otb-api` 9418 / `otb-web` 5918
- **数据库**：SQLite（未接共享库；接入需 compose 声明 `env_net`）
- **部署方式**：本机手动 scp 产物到 `projects/otb/api/` 与 `projects/otb/web/`，服务器 `docker compose up -d otb-api otb-web`；根 README 曾写"CI 直传"为愿景，非现实
- **回滚惯例**：上传前留 `otb-api.bak` / `web.old/`（服务器上已存在）
- **服务器密钥**：`projects/otb/api/` 下 `.env`（env_file 注入；模板 `.env.example`）
- **目录备注**：实际目录为 `api/`（根 `.env.example` 注释里写 `app/.env` 是笔误，以本文件为准）

## OPS 日志

- 2026-09-02 建档（全景侦查确认现状：手动通道、目录属主已是 deploy、回滚备份在位）。待办：迁移通道②（暂缓）。
