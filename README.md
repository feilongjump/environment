# Environment

服务器统一运行环境编排（Docker Compose）：共享基础设施 + 各项目服务，多台服务器跑多个生产项目。
本仓库是**部署相关事务的唯一入口与档案中心**：查"某项目怎么部署"、接入新项目、改部署，都在这里进行（一个项目一个 AI 会话）。

## 结构

```
├── docker-compose.yml        # 根编排：共享设施（MySQL/PostgreSQL）+ include 各项目
├── .env.example              # 全部环境变量模板（按项目分段）
└── projects/
    ├── _demo/              # 新项目接入模板（含步骤 README，复制即用）
    ├── _shared/            # 跨项目公共片段（如 base.conf：gzip/安全头/公共反代头）
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

新项目接入：按 `projects/_demo/README.md` 的**四阶段 SOP** 执行（决策 → 环境仓库侧 → 项目仓库侧 → 服务器验证；全程在本仓库的项目专属会话里指挥）。

## 项目清单

档案页 = 每项目唯一事实源（`projects/<name>/DEPLOY.md`，含 OPS 日志），改部署先读它：

| 项目 | 档案页 | 服务 | 对外端口 | 部署方式 |
|---|---|---|---|---|
| otb | [DEPLOY.md](projects/otb/DEPLOY.md) | `otb-api` / `otb-web` | 9418 / 5918 | 通道①手动上传 + compose（迁移②暂缓） |
| flowstock | [DEPLOY.md](projects/flowstock/DEPLOY.md) | `flowstock-api` | 9420 | 通道②push main 自动（Actions → deploy 用户），**新项目照抄样板** |
| retail-integration | [DEPLOY.md](projects/retail-integration/DEPLOY.md) | （未入 compose，systemd） | 8090 | 通道③项目 deploy.sh + systemd，双客户实例（moni/V21），计划迁② |

## 服务器与部署全景

| 服务器 | ssh 别名 | 在跑什么 | 部署通道 |
|---|---|---|---|
| **8.163.117.200**（主服务器） | `ssh env` | otb（compose）· flowstock（compose）· retail-integration **moni** 客户（systemd :8090）· 共享 postgres（moni 库 `retail_integration`） | ①②③ |
| **8.134.137.138**（V21 服务器） | `ssh v21` | retail-integration **V21** 客户（systemd :8090）· postgres（库 `retail_integration_v21`） | ③ |

三条部署通道：

- **① 手动上传 + compose**：otb（产物手动 scp 到 `projects/otb/`，服务器 compose 拉起）
- **② GitHub Actions 自动**：项目仓库 push main → CI 以 deploy 用户直传产物 → compose 拉起（**唯一全自动通道，新项目默认走此**；样例见 flow_stock 仓库 `.github/workflows/deploy.yml`）
- **③ 项目自带 deploy.sh + systemd + /opt**：retail-integration 多客户实例（moni@主服务器、V21@V21 服务器），**计划迁移到本仓库 compose，迁移前放置不理**

⚠️ **V21 服务器的 env 仓库故意停在旧提交**（9db6621）：新提交含 flowstock 的 include，该机无对应目录，盲目 `git pull` 会让 compose 报错。待 retail-integration 迁移 compose 时一并处理。

## 密钥与访问

**两条线，永不相交**：

- **个人线**（每台电脑一把）：本机 `id_ed25519_long_desktop` → 两台服务器 root；root 仅限密钥登录（主服务器已关密码登录）
- **CI 线**（每个项目一把）：私钥只存 GitHub Secrets，公钥装 `deploy@8.163` 的 `authorized_keys`，**必须带 `restrict` 前缀** + 注释 `ci-<项目名>@github-actions`

| key | 存在位置 | 用途 |
|---|---|---|
| `id_ed25519_long_desktop` | 本机 → 两台服务器 root | 个人管理（`ssh env` / `ssh v21`） |
| `id_rsa`（GitHub 上叫"豪华大鸡"） | 本机 → GitHub | **仅** GitHub 推送（2026-09-05 起从两台服务器撤除） |
| MI | 笔记本 → GitHub | 笔记本推送 |
| `flow_stock@github-actions` | GitHub Secrets + deploy@8.163 | flow_stock CI（已加 restrict） |
| `skp-7xv8...` | root@V21 服务器 | **来源不明，暂保留待查证** |
| `github-deploy` | 仅本机留档 | 已退役（原个人 key，已从 deploy 账户撤除） |

服务器账户分工：**root** = 管理（git pull、compose、sshd）；**deploy@8.163** = 纯 CI 通道（docker 组），个人不要用它登录。

**新机器初始化清单**（约 5 分钟）：

1. `git clone` 本仓库及各项目仓库
2. `gh auth login`
3. 生成个人 key：`ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_<机器名>`，公钥从已授权机器代办追加到两台服务器 `/root/.ssh/authorized_keys`
4. 复制 `~/.ssh/config` 的 `env` / `v21` 别名段

**给新项目发 CI key**：`ssh-keygen`（注释 `ci-<name>@github-actions`）→ `gh secret set` 写入项目仓库 → 公钥加 `restrict` 前缀追加到 deploy@8.163 → 删本地私钥副本。

## 操作纪律（重要）

- 所有 compose 命令**必须带服务名**：`docker compose up -d otb-api`、`docker compose restart mysql`
- **禁止裸 `docker compose down`**——会停掉本机全部项目（含共享数据库）
- 服务器路径 `/var/environment`；编排改动提交本仓库后，**主服务器**（`ssh env`）`git pull` 同步；V21 服务器按上文说明**不要** pull

## 快速开始（新服务器）

```bash
cp .env.example .env   # 按需填写各段变量
docker compose up -d mysql postgres        # 启动共享数据库
docker compose up -d otb-api otb-web       # 启动某项目（产物需先由 CI 部署到位）
docker compose ps                          # 查看状态
```

共享数据库接入：项目服务在自身 `compose.yml` 中声明 `networks: [env_net]` 即可通过 `mysql:3306` / `postgres:5432` 直连（OTB 当前用 SQLite，未接入）。
