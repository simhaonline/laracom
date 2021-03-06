# demo-service/Dockerfile

# 使用 golang 官方镜像提供 Go 运行环境，并且命名为 buidler 以便后续引用
FROM golang:alpine as builder

# 启用 Go Modules
ENV GO111MODULE on

# 更新安装源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装 git
RUN apk --no-cache add git

# 设置工作目录
WORKDIR /app/laracom-demo-service

# 将当前项目所在目录代码拷贝到镜像中
COPY . .

# 下载依赖
RUN go env -w GOPROXY=https://goproxy.io,direct && go mod tidy

# 构建二进制文件，添加来一些额外参数以便可以在 Alpine 中运行它
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o laracom-demo-service

# 下面是第二阶段的镜像构建，和之前保持一致
FROM alpine:latest

# 更新安装源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装相关软件
RUN apk update && apk add --no-cache bash ca-certificates curl

# 和上个阶段一样设置工作目录
RUN mkdir /app
WORKDIR /app

# 这一步不再从宿主机拷贝二进制文件，而是从上一个阶段构建的 builder 容器中拉取，神奇吧
COPY --from=builder /app/laracom-demo-service/laracom-demo-service .
COPY --from=builder /app/laracom-demo-service/demo.json .

# HEALTHCHECK --interval=3s --timeout=3s CMD curl -fs http://healthchecker:8081/health || exit 1

EXPOSE 9091
EXPOSE 8181

# 启动演示服务
CMD ["./laracom-demo-service", "--server_address=:9091"]