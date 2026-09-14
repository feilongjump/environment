# Environment

本地开发基础设施配置，支持数据库、Nginx 反向代理和多服务部署。

采用**"基础设施（git 追踪）+ 用户服务（gitignore）"**分层设计，
本项目本身保持稳定，你的后端服务定义在个人 override 文件中，不污染 git 历史。


## 目录结构

```
environment/
├── docker-compose.yml               # 基础设施编排（git 追踪）
├── docker-compose.override.yml      # 用户自定义服务（git 忽略）
├── .env.example                     # 环境变量模板
├── .gitignore
├── README.md
│
├── nginx/
│   ├── conf.d/
│   │   ├── .gitkeep
│   │   └── default.conf.example     # 站点配置 demo
│   └── sites/
│       └── .gitkeep                 # 前端构建物放这里
│
└── server/
    └── .gitkeep                     # 后端二进制放这里
```


## 核心设计

### 两层文件分工

| 文件 | 追踪 | 作用 |
|------|------|------|
| `docker-compose.yml` | ✓ git | 基础设施（MySQL、PostgreSQL、Nginx），基本不变 |
| `docker-compose.override.yml` | ✗ gitignore | 你的后端服务（api、auth 等），自由增删 |

执行 `docker-compose up` 时，Docker 自动合并这两个文件。

### 三个数据目录（均不提交）

| 目录 | 用途 | 来源 |
|------|------|------|
| `nginx/sites/<name>/` | 前端构建产物 | `npm run build` 输出，手动复制 |
| `nginx/conf.d/*.conf` | Nginx 站点配置 | 从 `default.conf.example` 复制修改 |
| `server/<name>/` | 后端二进制 | `go build` 输出，手动复制 |


## 快速开始

### 1. 初始化环境

```bash
# 复制环境变量模板
cp .env.example .env

# 编辑 .env，配置密码等（.env 不会被 git 追踪）
```

### 2. 启动基础设施

```bash
# 启动 mysql、postgres、nginx
docker-compose up -d

# 查看容器状态
docker-compose ps
```

### 3. 定义你的后端服务

新建 `docker-compose.override.yml`（如果不存在）：

```yaml
services:
  api:
    image: alpine:3.20
    container_name: api
    restart: no
    working_dir: /app/api
    command:
      - sh
      - -c
      - "apk add --no-cache ca-certificates && ./api"
    volumes:
      - ./server:/app:ro
    expose:
      - "8080"
    networks:
      - env_net
```

### 4. 添加前端构建物

```bash
# 在前端项目目录构建
npm run build

# 复制到 environment 的 nginx/sites/ 下
cp -r dist/* /path/to/environment/nginx/sites/a/
```

### 5. 添加后端二进制

```bash
# 编译 Go 程序（注意：需要 Linux 版本）
# Windows PowerShell:
$env:GOOS="linux"; go build -o api
# Linux/macOS:
GOOS=linux go build -o api

# 复制到 environment 的 server/ 下
mkdir -p /path/to/environment/server/api
cp api /path/to/environment/server/api/
```

### 6. 配置 Nginx 站点

```bash
# 复制 demo 配置
cp nginx/conf.d/default.conf.example nginx/conf.d/a.conf

# 编辑 a.conf：
#   - server_name a.localhost
#   - root /usr/share/nginx/sites/a
#   - proxy_pass http://api:8080
```

### 7. 启动所有服务

```bash
docker-compose up -d
```

### 8. 访问应用

浏览器打开：

- `http://a.localhost` → 前端页面（Vue/React SPA）
- `http://a.localhost/api/xxx` → 反代到 api 容器

> 根据 RFC 6761，所有 `*.localhost` 自动解析为 `127.0.0.1`，无需修改 hosts。


## 连接信息

### MySQL

- Host: `localhost`
- Port: `3306`（在 `.env` 中配置）
- Username: `root`
- Password: `MYSQL_ROOT_PASSWORD`（需在 `.env` 中设置）
- Database: `MYSQL_DATABASE`（需在 `.env` 中设置）

### PostgreSQL

- Host: `localhost`
- Port: `5432`（在 `.env` 中配置）
- Username: `POSTGRES_USER`（默认 `postgres`）
- Password: `POSTGRES_PASSWORD`（需在 `.env` 中设置）
- Database: `POSTGRES_DB`（需在 `.env` 中设置）

### Nginx

- 访问地址：`http://<站点名>.localhost`
- 宿主机端口：`${NGINX_PORT}`（默认 80）


## 常用操作

### 启动 / 停止

```bash
docker-compose up -d         # 启动
docker-compose ps            # 查看状态
docker-compose logs -f nginx # 查看 nginx 日志
docker-compose down          # 停止所有容器
docker-compose down -v       # 停止并删除数据卷（重置）
```

### 重启单个服务

```bash
docker-compose restart nginx   # 重启 nginx（配置或前端变更）
docker-compose restart api     # 重启 api（二进制变更）
```

### 调试

```bash
# 进入容器
docker-compose exec nginx sh
docker-compose exec api sh

# 查看实时日志
docker-compose logs -f --tail=100 api
```


## 添加新站点 / 新服务

以添加 `b.localhost`（前端）+ `auth` 服务（端口 8081）为例：

### 1. 建目录

```bash
mkdir -p nginx/sites/b
mkdir -p server/auth
```

### 2. 复制二进制和前端构建物

```bash
# 后端（Linux 版本）
cp path/to/auth server/auth/auth

# 前端
cp -r path/to/b-frontend/dist/* nginx/sites/b/
```

### 3. 写 Nginx 配置

```bash
cp nginx/conf.d/default.conf.example nginx/conf.d/b.conf
```

编辑 `b.conf`，修改三处：
- `server_name b.localhost;`
- `root /usr/share/nginx/sites/b;`
- `proxy_pass http://auth:8081;`

### 4. 在 override 文件加服务定义

编辑 `docker-compose.override.yml`，追加：

```yaml
  auth:
    image: alpine:3.20
    container_name: auth
    restart: no
    working_dir: /app/auth
    command:
      - sh
      - -c
      - "apk add --no-cache ca-certificates && ./auth"
    volumes:
      - ./server:/app:ro
    expose:
      - "8081"
    networks:
      - env_net
```

### 5. 启动

```bash
docker-compose up -d
```

### 6. 访问

`http://b.localhost`


## 开发迭代流程

| 修改内容 | 操作 | 是否需重启容器 |
|---------|------|--------------|
| 前端代码 | `npm run build` 后复制 dist | 否（Nginx 实时读取挂载文件） |
| 后端代码 | 重新编译并复制二进制 | 是（`docker-compose restart <服务名>`） |
| Nginx 配置 | 编辑 conf 文件 | 是（`docker-compose restart nginx`） |
| override 文件 | 添加新服务 | 是（`docker-compose up -d`） |


## 注意事项

- **后端必须是 Linux 二进制**：在 macOS/Linux 上用 `GOOS=linux go build`；在 Windows 上 PowerShell 用 `$env:GOOS="linux"; go build`
- **不要提交个人配置**：`docker-compose.override.yml`、`.env`、`nginx/sites/` 内容、`server/` 内容、`nginx/conf.d/*.conf` 都在 `.gitignore` 中
- **Git 协作**：每个人的 override 文件互不影响，各自维护本地服务
