# _demo：新项目接入 SOP（四阶段全流程）

本目录是**模板**（下划线前缀 = 非真实项目，同 `_shared`），不会被根 `docker-compose.yml` include。

**铁律**：接入全程在**本仓库（environment）的会话**里指挥，一个项目一个专属会话，会话开场先读该项目的 `projects/<name>/DEPLOY.md` 档案页。项目仓库只被动接收文件（CI workflow、指路牌）——阶段 2 虽然写的是项目仓库的文件，但仍在**本仓库会话**里直接读写项目仓库目录完成（指挥权与文件物理位置分离）。项目仓库的 `docs/deploy.md` 指路牌会把走错门的人引回这里。

## 阶段 0：决策（本仓库会话）

| 决策项 | 选项 | 参考 |
|---|---|---|
| 项目名 `<name>` | 全小写；目录/服务/容器/变量统一前缀 | 根 README 项目清单（避免撞名撞端口） |
| 形态 | **A** 前后端分离（api 二进制 + web 静态 + nginx 容器）/ **B** 单二进制（前端 go:embed 内嵌，页面+API 同端口） | A 样例 `projects/otb/`；B 样例 `projects/flowstock/` |
| 端口 | 查根 `.env` 已占用端口再分配 | A 需两个（api/web），B 一个 |
| 数据库 | 共享 postgres / mysql / SQLite | 共享库走 env_net 直连 `postgres:5432` / `mysql:3306`，首次建库 |
| CI 触发 | **全项目统一**：push main 自动 + `workflow_dispatch` 手动兜底 | 以 flow_stock 仓库 workflow 为准 |

## 阶段 1：环境仓库侧（本仓库会话直接执行）

1. **复制模板**：`cp -r projects/_demo projects/<name>`，删掉本 README
2. **改 compose.yml**：服务名/容器名 `<name>-api`（形态 A 另加 `<name>-web`，全机唯一）；环境变量前缀全量替换为 `<NAME>_`
3. **登记 include**：根 `docker-compose.yml` 的 `include:` 列表加一行
4. **登记变量**：根 `.env.example` 加项目段（`<NAME>_API_PORT`、`<NAME>_RESTART`…），服务器 `.env` 同步填值
5. **应用密钥**：按 `api/.env.example` 注释生成真实值（如 `openssl rand -hex 32`）
6. **发 CI key**：每项目一把，见下方"发钥流程"
7. **提交推送本仓库**；主服务器同步：`ssh env` → `cd /var/environment && git pull`
8. **服务器准备**（`ssh env`）：
   - `mkdir -p projects/<name>/{api,web,data,logs}`（形态 B 无 web）
   - `chown -R deploy:deploy projects/<name>`（CI 直传必需，**漏了 CI 必失败**）
   - `cp api/.env.example api/.env` 填真实值（600，deploy 属主）
   - 共享库建库（一次）：`docker exec postgres psql -U postgres -c "CREATE DATABASE <db>;"`

⚠️ **首次部署顺序铁律**：阶段 1 全部就绪（尤其目录属主、`api/.env`、发钥）之后，才允许阶段 2 的首次推送——否则 CI 直传失败或容器起不来，首部署即半成品。

## 阶段 2：项目仓库侧（仍在**本仓库会话**做：直接写项目仓库目录并提交推送）

1. 写 `.github/workflows/deploy.yml`：**照抄 flow_stock 仓库同路径文件**，按形态改"构建"与"激活"两段
2. 产物合同：构建 → scp 先落 `/tmp/<name>-ci`（不直接覆盖运行中文件）→ ssh 激活（mv 到位 + `chmod 0755`）→ `docker compose up -d <name>-api` → restart → ps
3. **回滚约定（激活段内置）**：放新产物前先留旧版本——形态 A 留 `web.old/` 与 `api/<binary>.bak`（otb 惯例）；形态 B 留 `api/<binary>.old`。回滚 = `ssh env` 把旧文件 mv 回来 + `docker compose restart <name>-api`
4. 放**指路牌**：项目仓库 `docs/deploy.md`（模板见文末）
5. Secrets 核对（项目仓库）：`SSH_HOST`（8.163.117.200）/ `SSH_USER`（deploy）/ `SSH_KEY`（阶段 1 发的私钥）
6. 提交推送项目仓库 main → 首次部署自动触发

## 阶段 3：服务器验证（`ssh env`）

- `docker compose ps <name>-api` → **healthy**
- `curl http://127.0.0.1:<端口>/`（或 health 端点）
- 在 `projects/<name>/DEPLOY.md` 追加一行 OPS 日志（日期 + 做了什么 + 结果）

## 发钥流程（阶段 1 第 6 步展开；本仓库会话执行）

```powershell
# 私钥只进 GitHub Secrets，不留本地
ssh-keygen -t ed25519 -f $env:TEMP\ci-<name> -N "" -C "ci-<name>@github-actions"   # 引号被吃就加 --% 前缀
gh secret set SSH_KEY  --repo feilongjump/<项目仓库> --body (Get-Content $env:TEMP\ci-<name> -Raw)
gh secret set SSH_HOST --repo feilongjump/<项目仓库> --body "8.163.117.200"   # 该仓库首次才需要
gh secret set SSH_USER --repo feilongjump/<项目仓库> --body "deploy"          # 该仓库首次才需要
# 公钥上服务器（restrict 前缀强制；由会话通过 ssh env 代办）：
#   echo "restrict <公钥内容>" >> /home/deploy/.ssh/authorized_keys
Remove-Item $env:TEMP\ci-<name>, $env:TEMP\ci-<name>.pub -Force   # 删本地副本
```

## 指路牌模板（阶段 2 第 4 步，写入项目仓库 `docs/deploy.md`）

```markdown
# 部署

部署由 environment 仓库统一管理（通道②：push main 自动部署）。

- 档案页（唯一事实源）：`C:\Users\long\Code\environment\projects\<name>\DEPLOY.md`
- 部署相关改动请在 environment 仓库的本项目专属会话中进行，勿在本仓库改部署配置。
```

## 目录约定（与 .gitignore 对应）

```
<name>/
├── compose.yml            # 服务定义（被根 compose include）✅ 入库
├── nginx.conf             # 前端反代配置（可选，放项目根，勿放 web/ 里）✅ 入库
├── DEPLOY.md              # 部署档案页（唯一事实源 + OPS 日志）✅ 入库
├── api/
│   ├── <name>-api         # CI 直传的二进制 ❌ 不入库（旧版留 .bak/.old 供回滚）
│   ├── config.prod.yaml   # 版本化配置 ✅ 入库
│   ├── .env               # 密钥，env_file 注入 ❌ 不入库
│   └── .env.example       # 密钥模板 ✅ 入库
├── web/                   # 前端产物（CI 直传，纯产物目录）❌ 整目录不入库（旧版留 web.old/）
├── data/                  # 业务数据 ❌ 不入库（备份 = 拷此目录）
└── logs/                  # 运行日志 ❌ 不入库（无需备份）
```

注意 `api/` 在 .gitignore 里是"默认全忽略 + 例外放行"：`config.prod.yaml`、`.env.example` 之外的新文件如需入库，要在根 `.gitignore` 加 `!` 例外。

路径约定（`api/config.prod.yaml` 模板里有示例，键名随应用定、路径前缀勿改）：业务数据写 `/app/data/`，运行日志写 `/app/logs/`，与 compose 挂载一一对应。

## 参考

- 形态 A 完整样例：`projects/otb/`（Go 二进制 + nginx 前端；手动上传，待迁移通道②）
- 形态 B 完整样例：`projects/flowstock/` + flow_stock 仓库 `.github/workflows/deploy.yml`（全自动，新项目照抄）
- 公共 nginx 片段：`projects/_shared/`（gzip/安全头/反代头，注意根 README 的继承坑说明）
