# 快速上手
只需复制粘贴回车重复操作两次即可！（awesome）

## 1. 部署 docker

```
curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh
```

## 2. 启动 xray 容器
 
下面两种协议，任取一个复制后，直接执行命令即可。（当然你可以一起创建 >_< ）

xray vision:
```
USER_PORT=61111 && CONTAINER_NAME="xray_vision" && \
docker run -d --name ${CONTAINER_NAME} --restart=always \
  -p ${USER_PORT}:443 \
  -e ProtocolType="Vision" \
  -e HOST_PORT="${USER_PORT}" \
  kevinstarry/xray:latest && \
  sleep 3 && docker exec -it ${CONTAINER_NAME} cat /xray_info.txt
```

xray grpc:
```
USER_PORT=62222 && CONTAINER_NAME="xray_grpc" && \
docker run -d --name ${CONTAINER_NAME} --restart=always \
  -p ${USER_PORT}:443 \
  -e ProtocolType="gRPC" \
  -e HOST_PORT="${USER_PORT}" \
  kevinstarry/xray:latest && \
  sleep 3 && docker exec -it ${CONTAINER_NAME} cat /xray_info.txt
```

补充：USER_PORT 是主机端口(建议自己设置一个端口)，注意需要服务器放开该端口（一般大厂都需要主动开放，小厂通常所有端口都是开放的）。

系统端口（Well-known Ports）：0-1023。这些端口通常由系统或特定的服务使用，例如 HTTP（端口 80）、HTTPS（端口 443）、FTP（端口 21）等。

注册端口（Registered Ports）：1024-49151。这些端口通常由应用程序和服务使用，它们可以被程序注册以确保它们的端口号不会与其他应用冲突。

动态或私有端口（Dynamic or Private Ports）：49152-65535。这些端口通常由操作系统动态分配给客户端应用程序。

## 3. 查看生成的链接
链接在上面已经生成了，如果忘了可以使用：

docker exec -it ${CONTAINER_NAME} cat /xray_info.txt

${CONTAINER_NAME} 使用你指定的名字替换，如果你使用上面的命令，那么应该是 xray_vision 或者 xray_grpc 

## 4. 移除

停止容器，移除容器，移除镜像  （悄悄地，我走了，不留下一丝痕迹，仿佛从未来过）
```
CONTAINER_NAME="xray_grpc" &&  docker stop ${CONTAINER_NAME} && docker rm ${CONTAINER_NAME} && docker rmi kevinstarry/xray:latest
```
```
CONTAINER_NAME="xray_vision" &&  docker stop ${CONTAINER_NAME} && docker rm ${CONTAINER_NAME} && docker rmi kevinstarry/xray:latest
```

# 说明
## 配置参数
可配置环境变量展示与说明：
```
ENV UUID=""
ENV DOMAIN=""
ENV PRIVATEKEY=""
ENV PUBLICKEY=""
ENV HOST=""

# ENV ProtocolType= "Vision" Or "gRPC"
ENV ProtocolType=""
ENV HOST_PORT=""
ENV COMMENT=""
```
 
UUID 由 xray uuid 生成

PRIVATEKEY PUBLICKEY 由 xray x25519 生成

DOMAIN 是伪装域名：如果不指定则会从下面的域名中随机生成一个
```
    fake_domains = [
        "addons.mozilla.org",
        "www.amd.com",
        "www.samsung.com",
        "www.swift.com",
        "www.cisco.com",
        "www.apple.com"
    ]
```

HOST：这个是服务器的公网 ipv4 地址 （当然你可以使用域名，如果域名解析到了你的服务器公网ip）

不指定则会使用 curl 从下面的列表中逐个获取，直到成功获取，通常无需指定
```
["api.ipify.org", "ifconfig.me", "ip.me", "ipinfo.io/ip","ip.sb"]
```

下面两个通常是必填参数：

HOST_PORT:这个是主机（宿主机）端口，实际上随便什么端口都可以，只是需要将该参数传递给容器，不然链接生成时候端口不知道是哪一个

ProtocolType：只接受两个值 Vision 和 gRPC （区分大小写），如果是其他值会默认选择 gRPC，所以可以理解为接受 Vision 和 其他值。

COMMENT：顾名思义，节点名称备注

## 以Vison协议举例
例如要配置 2333 作为服务器端口

使用 Vision 协议

容器名称叫：xray_vision_example

伪装域名使用：www.cisco.com

UUID 使用：6d299974-944d-44f2-acc1-5d520f386ae3

PRIVATEKEY 使用：4K3tYDfQYRdb-Ud62sdHrAsoHfWfoWc53HazfB9ssh0

PUBLICKEY 使用：sUsiFWX5bnnnR2ULfyBWHMItvlTsUpgU2oLXSmDobEo

```
USER_PORT=2333 && CONTAINER_NAME="xray_vision_example" && \
docker run -d --name ${CONTAINER_NAME} --restart=always \
  -p ${USER_PORT}:443 \
  -e DOMAIN="www.cisco.com" \
  -e UUID="6d299974-944d-44f2-acc1-5d520f386ae3" \
  -e PRIVATEKEY="4K3tYDfQYRdb-Ud62sdHrAsoHfWfoWc53HazfB9ssh0" \
  -e PUBLICKEY="sUsiFWX5bnnnR2ULfyBWHMItvlTsUpgU2oLXSmDobEo" \
  -e ProtocolType="Vision" \
  -e HOST_PORT="${USER_PORT}" \
  kevinstarry/xray:latest && \
  sleep 1 && docker exec -it ${CONTAINER_NAME} cat /xray_info.txt
```

# 关于ipv6
任意找一个ip的网址，使用curl命令获取一下即可：
```
curl --ipv6 ip.me
```
节点：把拿到的ipv6直接替换ipv4地址即可。（ipv6目前生态还不行，有些线路可能v6直，v4绕，通常差距也不会很大）

## 为什么要单独配置这些变量
考虑到有些用户有多台服务器，部署完全一样的 vless 链接，如果某个服务器挂了，可以直接把 ip 替换就能继续使用

是否安全？ 当然使用同一个配置肯定没有完全随机生成安全，但是通常情况下自己使用完全可以，自己妥善保管就好。

## 为什么要做这个镜像
目前主流协议大部分都需要使用域名申请证书，虽然有各类脚本，但仍觉得繁琐，如果两行命令可以解决的事情，就不要浪费太多时间去研究。

docker buildx build amd64和arm64两个版本，基本可以覆盖所有linux，如果 xray reality 工具或者协议没有重大更新，那么本镜像也不必更新。

工具就应该足够简单和易用，不要浪费时间在折腾工具上，宁可躺在草地上晒一下午太阳，也不要浪费一分钟去折腾服务器。这就是本镜像的初衷。

docker hub: https://hub.docker.com/repository/docker/kevinstarry/xray
# 致谢
排名不分先后：

https://github.com/docker

https://github.com/XTLS/Xray-core

https://github.com/XTLS/Xray-examples

https://github.com/wulabing/xray_docker

# 寄语
Reality 让人感觉仿佛是回到了 SSR 的那个时代，致敬 “逗比”。如果帮助到你，Star Star Star

