# school-uniform-mall 部署档案（唯一事实源）

> 部署相关改动在 environment 仓库的本项目专属会话中进行；任何会话开场先读本文件。服务器侧对应路径：`/var/environment/projects/school-uniform-mall/`。

## 现状

- **通道**：② GitHub Actions（接入完成后 push main 自动 + workflow_dispatch 手动）
- **项目仓库**：`feilongjump/school-uniform-mall`（本地 `C:\Users\long\Code\school-uniform-mall`）
- **形态**：B 单二进制（管理后台前端 go:embed 内嵌进 `sum`，页面 + API 同端口；小程序产物不入服务器，走微信平台）
- **服务器**：主服务器（`ssh env`）
- **服务/端口**：`sum-api` / `SUM_PORT=9422`（容器内 8080；health：`/api/health`）
- **数据库**：共享 postgres（env_net 直连 `postgres:5432`，库 `school_uniform_mall`；首次启动自动 goose 迁移 + 种子超管 admin/admin123，**上线后立即改密**）
- **Secrets**（school-uniform-mall 仓库）：`SSH_HOST` / `SSH_USER` / `SSH_KEY`（= `ci-school-uniform-mall@github-actions`，装在 deploy@主服务器，带 `restrict`）
- **服务器密钥**：`projects/school-uniform-mall/api/.env`（JWT_SECRET / DATABASE_DSN，600 deploy 属主）
- **回滚**：激活段内置 `sum.old` 备份；回滚 = `ssh env` 把 `.old` mv 回来 + `docker compose restart sum-api`
- **构建要点**：bun 构建前端 → 产物校验（`assets/` 标记防空壳包）→ CGO_ENABLED=0 交叉编译；CI 前置 verify 作业（web typecheck+lint、go test）
- **时区**：compose 已挂 zoneinfo（避 alpine 无 tzdata 跑 UTC 的坑，根 README 有说明）——与 flowstock/otb 不同，勿对齐他们

## OPS 日志

- 2026-09-08 首次接入（四阶段 SOP）：建目录/compose/DEPLOY，登记 include 与 SUM_ 变量，发 CI key，服务器准备，首次部署。
