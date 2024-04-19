FROM nginx:alpine

# 构建环境时 - 环境变量
ARG TZ

# Alpine apk 设置国内源
# 更改系统镜像源（参考 https://www.zzxworld.com/posts/cn-mirrors-for-debian-12-linux）
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories

RUN apk add --update tzdata \
  && cp /usr/share/zoneinfo/$TZ /etc/localtime

COPY nginx.conf /etc/nginx/

WORKDIR /var/www

EXPOSE 80 443