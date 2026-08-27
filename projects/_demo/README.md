# _demo：新项目接入模板

本目录是**模板**（下划线前缀 = 非真实项目，同 `_shared`），不会被根 `docker-compose.yml` include。
接入新项目：复制本目录为 `projects/<name>/`，按下文改造。

## 接入步骤

1. **复制目录**：`cp -r projects/_demo projects/<name>`，删掉本 README
2. **改 compose.yml**：服务名/容器名改为 `<name>-api` 等（带项目前缀，全机唯一）；环境变量前缀 `DEMO_` 全量替换为 `<NAME>_`
3. **登记 include**：根 `docker-compose.yml` 的 `include:` 列表加一行 `- projects/<name>/compose.yml`
4. **登记编排变量**：根 `.env.example` 加项目段（`<NAME>_API_PORT`、`<NAME>_RESTART`），服务器 `.env` 同步填值
5. **配密钥**：`cp api/.env.example api/.env` 填入真实值（`.env` 已被 gitignore）
6. **建运行目录**：`mkdir -p web data logs`（被 gitignore，不建则由 docker 以 root 身份自动创建，后续 CI 直传可能遇权限问题）；服务器上还需 `sudo chown -R <部署账号> projects/<name>`，否则 CI 无法直传产物、无法留 web.old 回滚备份
7. **验证启动**：仓库根目录 `docker compose config --quiet` 通过后 `docker compose up -d <name>-api`

## 目录约定（与 .gitignore 对应）

```
<name>/
├── compose.yml            # 服务定义（被根 compose include）✅ 入库
├── nginx.conf             # 前端反代配置（可选，放项目根，勿放 web/ 里）✅ 入库
├── api/
│   ├── <name>-api         # CI 直传的二进制 ❌ 不入库
│   ├── config.prod.yaml   # 版本化配置 ✅ 入库
│   ├── .env               # 密钥，env_file 注入 ❌ 不入库
│   └── .env.example       # 密钥模板 ✅ 入库
├── web/                   # 前端产物（CI 直传，纯产物目录）❌ 整目录不入库
├── data/                  # 业务数据 ❌ 不入库（备份 = 拷此目录）
└── logs/                  # 运行日志 ❌ 不入库（无需备份）
```

注意 `api/` 在 .gitignore 里是"默认全忽略 + 例外放行"：`config.prod.yaml`、`.env.example` 之外的新文件如需入库，要在根 `.gitignore` 加 `!` 例外。

路径约定（`api/config.prod.yaml` 模板里有示例，键名随应用定、路径前缀勿改）：业务数据写 `/app/data/`，运行日志写 `/app/logs/`，与 compose 挂载一一对应。

## 参考

- 完整实现样例：`projects/otb/`（Go 二进制 + nginx 前端）
- 公共 nginx 片段：`projects/_shared/`（gzip/安全头/反代头，注意根 README 的继承坑说明）
