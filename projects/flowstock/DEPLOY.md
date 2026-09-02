# flowstock 部署档案（唯一事实源）

> 部署相关改动在 environment 仓库的 flowstock 专属会话中进行；任何会话开场先读本文件。服务器侧对应路径：`/var/environment/projects/flowstock/`。

## 现状

- **通道**：② GitHub Actions 全自动（push main 自动 + workflow_dispatch 手动）——新项目接入的参照样板
- **项目仓库**：`feilongjump/flow_stock`（本地 `C:\Users\long\Code\flow_stock`）
- **形态**：B 单二进制（前端 go:embed 内嵌，页面 + API 同端口）
- **服务器**：主服务器（`ssh env`）
- **服务/端口**：`flowstock-api` / `FLOWSTOCK_PORT=9420`（health：`/api/health`）
- **数据库**：共享 postgres，库 `flow_stock`
- **Secrets**（flow_stock 仓库）：`SSH_HOST` / `SSH_USER` / `SSH_KEY`（= `ci-flowstock@github-actions`，装在 deploy@主服务器，带 `restrict`）
- **服务器密钥**：`projects/flowstock/api/.env`（DATABASE_URL / JWT_SECRET / DB_DRIVER / PORT，600 deploy 属主）
- **回滚**：激活段内置 `flowstock.old` 备份；回滚 = `ssh env` 把 `.old` mv 回来 + `docker compose restart flowstock-api`
- **构建要点**：bun 构建前端 → 产物校验（`framework-` 标记防空壳包）→ CGO_ENABLED=0 交叉编译

## OPS 日志

- 2026-09-02 接入收尾，修三缺口：①服务器目录属主 root→deploy（CI mv 必失败）②Secrets 私钥错配（原存的是个人 key）→补发规范 CI key ③`api/.env` 从未创建→服务器生成 + 建库。run 33594835737 首次全链路成功，容器 healthy。
- 2026-09-02 workflow 激活段补 `flowstock.old` 回滚备份（对齐 SOP 回滚约定）。
