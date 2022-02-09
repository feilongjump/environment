# Environment

## 安装

1、你需要克隆 `environment` 仓库到你的电脑上
```
git clone https://github.com/PretendTrue/environment.git
```

2、进入根目录，复制 `env-example` 一份并且更改文件名为 `.env`
```
cp env-example .env
```

3、进入 `nginx` 文件加下的 `sites` 文件加下，添加你的站点配置文件

4、然后你就可以开始愉快的使用了

## 使用

### 运行这些容器。
```
docker-compose up -d nginx php mysql
```

### 进入 `php` 容器使用
```
docker-compose exec php bash
```

### 关闭这些容器
```
docker-compose stop
```

### 删除所有容器
```
docker-compose down
```

### `Laravel` 项目
打开 `Laravel` 项目的 `.env` 文件 然后 配置 你的 `mysql` 的 `DB_HOST`:
```
DB_HOST=mysql
```

## 遇到的问题
### Different lower_case_table_names settings for server ('2') and data dictionary ('0')
* Docker Desktop Version: `4.4.3 ~ 4.4.4`
* MySQL Version: `8.0.27`
* Windows 10
> 据说在旧版本时，`lower_case_table_names` 默认值是为 0 的，但是在以上版本中，则将值修改为 2 了。

> 有说法可以将版本降至 `4.1.0` 便可解决（我没有尝试）。

> 可以尝试使用此方式，[链接在此](https://stackoverflow.com/questions/64153426/laradock-mysql-container-exits0-different-lower-case-table-names-settings-fo)

> `command: --lower-case-table-names=2` 直接设置值，在 Windows 10 下是可行的，但不清楚在 MacOS 和 Linux 是否会出现问题
