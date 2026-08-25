# _demo：新项目接入模板

本目录是**模板**（下划线前缀 = 非真实项目，同 `_shared`），不会被根 `docker-compose.yml` include。
接入新项目：复制本目录为 `projects/<name>/`，按下文改造。

## 接入步骤

1. **复制目录**：`cp -r projects/_demo projects/<name>`，删掉本 README
2. **改 compose.yml**：服务名/容器名改为 `<name>-api` 等（带项目前缀，全机唯一）；环境变量前缀 `DEMO_` 全量替换为 `<NAME>_`
3. **登记 include**：根 `docker-compose.yml` 的 `include:` 列表加一行 `- projects/<name>/compose.yml`
4. **登记编排变量**：根 `.env.example` 加项目段（`<NAME>_API_PORT`、`<NAME>_RESTART`），服务器 `.env` 同步填值
5. **配密钥**：`cp app/.env.example app/.env` 填入真实值（`.env` 已被 gitignore）
6. **验证启动**：仓库根目录 `docker compose config --quiet` 通过后 `docker compose up -d <name>-api`

## 入库约定（与 .gitignore 对应）

| 路径 | 入库 | 说明 |
|---|---|---|
| `compose.yml`、`app/config/`、`app/nginx.conf`、`app/.env.example` | ✅ | 编排与配置，版本化 |
| `app/.env` | ❌ | 密钥，由 env_file 注入；服务器上从 .env.example 复制填写 |
| `app/bin/`、`app/dist/` | ❌ | 产物，由应用仓库 CI 直传 |
| `data/` | ❌ | 运行数据，备份 = 拷此目录 |

## 参考

- 完整实现样例：`projects/otb/`（Go 二进制 + nginx 前端）
- 公共 nginx 片段：`projects/_shared/`（gzip/安全头/反代头，注意根 README 的继承坑说明）
