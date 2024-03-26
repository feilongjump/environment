FROM golang:alpine as builder

# 设置必要的环境变量
ENV GOPROXY=https://goproxy.cn,direct \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

# 移动到工作目录：/app
WORKDIR /app

# 此步骤相当于进行缓存，在 go.mod 以及 go.sum 文件没有进行修改时，可以直接从缓存中获取
# 复制 go.mod 和 go.sum 并下载依赖
COPY go.mod go.sum ./
RUN go mod download

# 将代码复制到容器中
COPY . /app

# 将代码编译成二进制可执行文件 howio
RUN go build -o howio .


########## 创建一个小镜像 ########
FROM alpine:latest

# 构建环境时 - 环境变量
ARG TZ
ARG PORT

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 从 builder 镜像中进行复制
COPY --from=builder /app/env.production.toml /env.local.toml
COPY --from=builder /app/howio /howio

RUN chmod +x /howio

# 声明服务端口
EXPOSE $PORT

# 启动容器时运行的命令
CMD ["/howio"]
