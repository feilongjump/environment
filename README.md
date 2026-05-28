# Environment

本地开发基础设施配置，支持多种服务。

## 目录结构

```
├── docker-compose.yml      # Docker Compose 配置
├── .env.example           # 环境变量模板
├── .gitignore             # 忽略敏感文件和备份
└── README.md              # 使用说明
```

## 快速开始

### 启动服务

```bash
docker-compose up -d
```

### 查看容器状态

```bash
docker-compose ps
```

### 连接信息

从 `.env.example` 复制为 `.env` 后配置以下变量：

**MySQL**
- Host: `localhost`
- Port: `3306`
- Username: `root`
- Password: `MYSQL_ROOT_PASSWORD`（需在 `.env` 中设置）
- Database: `MYSQL_DATABASE`（需在 `.env` 中设置）

**PostgreSQL**
- Host: `localhost`
- Port: `5432`
- Username: `POSTGRES_USER`
- Password: `POSTGRES_PASSWORD`（需在 `.env` 中设置）
- Database: `POSTGRES_DB`（需在 `.env` 中设置）

### 停止服务

```bash
docker-compose down
```

### 重置（删除所有数据）

```bash
docker-compose down -v
```