FROM mysql

# 构建环境时 - 环境变量
ARG TZ

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

EXPOSE 3306