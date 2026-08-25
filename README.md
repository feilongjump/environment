# Environment

服务器统一运行环境编排（Docker Compose）：共享基础设施 + 各项目服务，一台服务器跑多个生产项目。

## 结构

```
├── docker-compose.yml        # 根编排：共享设施（MySQL/PostgreSQL）+ include 各项目
├── .env.example              # 全部环境变量模板（按项目分段）
└── projects/
    └── <name>/               # 每个项目一个目录
        ├── compose.yml       #   服务定义（被根 compose include）
        ├── app/              #   配置与产物（产物由应用仓库 CI 直传，不入库）
        └── data/             #   运行数据（不入库，备份 = 拷此目录）
```

新项目接入：`projects/<name>/` 照抄现有项目结构，根 `docker-compose.yml` 的 `include` 列表登记一行。

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
