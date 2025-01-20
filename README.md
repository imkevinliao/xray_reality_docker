# 快速上手

## 1. 服务器部署 docker

```curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh```

## 2. 启动 xray
下面两种协议，任取一个即可，直接复制运行即可，唯一需要注意的是端口也许你想用自己设置的。

新手提示：USER_PORT 是主机端口，需要服务器放开该端口（一般大厂都需要主动开放，小厂通常所有端口都是开放的），USER_PORT 可以自行修改即可

xray vision:
```
USER_PORT=61111 && CONTAINER_NAME="xray_vision" && \
docker run -d --name ${CONTAINER_NAME} --restart=always \
  -p ${USER_PORT}:443 \
  -e ProtocolType="Vision" \
  -e HOST_PORT="${USER_PORT}" \
  kevinstarry/xray:latest && \
  sleep 1 && docker exec -it ${CONTAINER_NAME} cat /xray_info.txt
```

xray grpc:
```
USER_PORT=62222 && CONTAINER_NAME="xray_grpc" && \
docker run -d --name ${CONTAINER_NAME} --restart=always \
  -p ${USER_PORT}:443 \
  -e ProtocolType="gRPC" \
  -e HOST_PORT="${USER_PORT}" \
  kevinstarry/xray:latest && \
  sleep 1 && docker exec -it ${CONTAINER_NAME} cat /xray_info.txt
```
## 3. 查看生成的链接
链接在上面已经生成了，如果忘了可以使用：

docker exec -it ${CONTAINER_NAME} cat /xray_info.txt

${CONTAINER_NAME} 使用你指定的名字替换，如果你使用我上面的命令，那么应该是 xray_vision 或者 xray_grpc 


# 补充说明
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

HOST：这个是服务器的公网 ip 地址

不指定则会使用 curl 从下面的列表中逐个获取，成功获取则终止，都通常不需要指定
```
["api.ipify.org", "ifconfig.me", "ip.me", "ipinfo.io/ip","ip.sb"]
```

下面两个通常是必填参数：

HOST_PORT:这个是主机端口，实际上随便什么端口都可以，只是需要将该参数传递给容器，不然链接生成时候端口不知道是哪一个

ProtocolType：只接受两个值 Vision 和 gRPC （区分大小写），如果是其他值会默认选择 gRPC，所以可以理解为接受 Vision 和 其他值。

## 为什么要单独配置这些变量
考虑到有些用户有多台服务器，部署完全一样的 vless 链接，如果某个服务器挂了，可以直接把 ip 替换就能继续使用

是否安全？ 当然使用同一个配置肯定没有完全随机生成安全，但是通常情况下自己使用完全可以，自己妥善保管就好。

## 以Vison协议举例
例如我要配置 2333 作为服务器端口

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


