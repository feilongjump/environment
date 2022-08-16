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

### 运行某些些容器。
```
docker compose up -d nginx php mysql
```

### 进入 `php` 容器使用
```
docker compose exec php bash
```

### 关闭这些容器
```
docker compose stop
```

### 删除所有容器
```
docker compose down
```

### `Laravel` 项目
打开 `Laravel` 项目的 `.env` 文件 然后 配置 你的 `mysql` 的 `DB_HOST`:
```
DB_HOST=mysql
```

### `Elasticsearch`

1. 先将 `kibana` 文件夹下的 `kibana-example.yml` 修改成 `kibana.yml`

2. 当 `Elasticsearch` 和 `Kibana` 容器启动后，留意控制台输出的 `Kibana` 访问地址
    > eg: Go to http://0.0.0.0:5601/?code=135441 to get started.


    `0.0.0.0` 表示本地机器上的所有 IPv4 地址都可以访问该端口。当您访问 `http://localhost:5601?code=135441` 时，系统会提示您输入注册令牌。

    当启动 `Elasticsearch` 容器时，复制并粘贴屏幕上显示的注册令牌。如果屏幕上堆满了日志或令牌已过期，可以使用该 `elasticsearch-create-enrollment-token` 命令生成一个新的。
    ```
    docker compose exec -it elasticsearch elasticsearch-create-enrollment-token -s kibana
    ```
    复制令牌进行配置 Elastic。

3. 此时如果需要输入 `Kibana` 的服务验证码，根据提示进行输入命令即可
    ```
    docker compose exec -it kibana bin/kibana-verification-code
    ```

4. 接下来登录 `Elasticsearch` 账号即可，默认的 `Elasticsearch` 账号名: `elastic`，密码根据配置文件的设置的密码进行登录

> 结尾吐槽一下
>  使用 `docker-compose.yml` 管理 `Elasticsearch` 和 `Kibana` 容器，在开启 `xpack` 安全性的情况下，文件编写配置是真的复杂，
>  搞不来，只能用这种方式了，也可以解决开启 `xpack` 安全性的前提下进行使用

## 遇到的问题
### Different lower_case_table_names settings for server ('2') and data dictionary ('0')
* Docker Desktop Version: `4.4.3 ~ 4.4.4`
* MySQL Version: `8.0.27`
* Windows 10
> 据说在旧版本时，`lower_case_table_names` 默认值是为 0 的，但是在以上版本中，则将值修改为 2 了。

> 有说法可以将版本降至 `4.1.0` 便可解决（我没有尝试）。

> 可以尝试使用此方式，[链接在此](https://stackoverflow.com/questions/64153426/laradock-mysql-container-exits0-different-lower-case-table-names-settings-fo)

> `command: --lower-case-table-names=2` 直接设置值，在 Windows 10 下是可行的，但不清楚在 MacOS 和 Linux 是否会出现问题
