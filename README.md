# Environment

服务器统一运行环境编排（Docker Compose）：共享基础设施 + 各项目服务，一台服务器跑多个生产项目。

## 结构

```
├── docker-compose.yml        # 根编排：共享设施（MySQL/PostgreSQL）+ include 各项目
├── .env.example              # 全部环境变量模板（按项目分段）
└── projects/
    ├── _demo/              # 新项目接入模板（含步骤 README，复制即用）
    ├── _shared/            # 跨项目公共片段（如 nginx-base.conf：gzip/安全头/公共反代头）
    │                       #   项目 compose 里挂载 ../_shared:/etc/nginx/shared:ro，
    │                       #   项目 nginx.conf 中 include /etc/nginx/shared/base.conf;
    └── <name>/               # 每个项目一个目录
        ├── compose.yml       #   服务定义（被根 compose include）
        ├── nginx.conf        #   前端反代配置（可选；改动后需 restart <name>-web）
        ├── api/              #   后端：CI 直传二进制（不入库）+ config.prod.yaml/.env.example（入库）+ .env（密钥，不入库）
        ├── web/              #   前端：CI 直传产物（整目录不入库，CI 就地替换内容）
        ├── data/             #   业务数据（不入库，备份 = 拷此目录）
        └── logs/             #   运行日志（不入库，应用内滚动切割，无需备份）
```

⚠️ nginx 继承坑：`proxy_set_header`/`add_header` 在 location 内一旦自定义，外层公共片段的同名指令对该 location 全部失效——项目 location 内不要重写这两类指令。

新项目接入：复制 `projects/_demo/` 为 `projects/<name>/` 并按其 README 改造；根 `docker-compose.yml` 的 `include` 列表登记一行。

## 项目清单

| 项目 | 服务 | 对外端口（.env 可调） | 产物来源 |
|---|---|---|---|
| otb | `otb-api` / `otb-web` | `OTB_API_PORT`(9418) / `OTB_WEB_PORT`(5918) | otb-api / otb-web 仓库 CI 直传，部署细节见 otb-api 仓库 `docs/deploy.md` |

## 操作纪律（重要）

- 所有 compose 命令**必须带服务名**：`docker compose up -d otb-api`、`docker compose restart mysql`
- **禁止裸 `docker compose down`**——会停掉本机全部项目（含共享数据库）
- 服务器路径 `/var/environment`；编排改动提交到本仓库后，服务器 `git pull` 同步

## 快速开始（新机器）

```bash
cp .env.example .env   # 按需填写各段变量
docker compose up -d mysql postgres        # 启动共享数据库
docker compose up -d otb-api otb-web       # 启动某项目（产物需先由 CI 部署到位）
docker compose ps                          # 查看状态
```

共享数据库接入：项目服务在自身 `compose.yml` 中声明 `networks: [env_net]` 即可通过 `mysql:3306` / `postgres:5432` 直连（OTB 当前用 SQLite，未接入）。
