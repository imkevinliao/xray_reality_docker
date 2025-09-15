# 科学上网
hysteria备用：https://github.com/imkevinliao/hysteria_docker
# 新版本
懒人只需要两步骤（复制粘贴回车重复两次）：

1. 部署 docker （敲完命令然后去喝杯茶 等待docker部署）
```
curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh
```

2. 一键科学上网（敲完命令然后去喝杯茶 等待vless链接生成）：
```
docker run -d --name reality --restart=always -p 12345:443 -e PORT=12345 kevinstarry/reality:latest && sleep 3 && docker exec -it reality cat info.txt
```

懒人到这里就部署完了，下面的两行命令可以瞄一下

查看节点信息 
```
docker exec -it reality cat info.txt
```

卸载：一键三连（彻底移除，干干净净）
```
docker stop reality && docker rm reality && docker rmi kevinstarry/reality
```

懒人到这里就结束了，后面的是给想了解更清楚地用户阅读的

----------------------------------------

自定义参数：修改服务器暴露的 PORT 和 伪装域名 (伪装域名可选，不填会从指定的域名列表中随机挑选一个)

例如 PORT 10086 伪装域名 www.apple.com
```
PORT=10086 && NAME="reality" && \
docker run -d --name ${NAME} --restart=always \
  -p $PORT:443 \
  -e PORT="$PORT" \
  -e DOMAIN="www.apple.com" \
  kevinstarry/reality:latest && \
  sleep 3 && docker exec -it ${NAME} cat info.txt
```



新版本是在原来的版本上改进的，主要改进和调整如下：

1. 两个协议合并，共用一个端口
2. 增加 geodata 每日自动更新
3. 路径优化 所有文件配置操作 都在 /app 路径下 更干净也更清晰
4. 节点信息放置在：/app/info.txt 

```
环境变量定义

如果不指定 PORT 默认使用 443，指定 PORT 则将 PORT 作为参数传递给容器（生成链接），当然你也可以选择手动修改链接 PORT

ENV PORT=""        暴露的主机端口，生成链接需要使用
ENV UUID=""        与先前一致
ENV DOMAIN=""      与先前一致
ENV PRIVATEKEY=""  与先前一致
ENV PUBLICKEY=""   与先前一致
ENV HOST=""        与先前一致
ENV COMMENT=""     与先前一致
```

--------------------------------

# 旧版本
只是留个纪念，没必要存在了，新版本完全是对旧版本的全面升级与改进
## 1. 启动 xray 容器
下面两种协议，任取一个复制后，直接执行命令即可。（当然你可以一起创建 >_< ）

reality vision 最稳 （速度差不多） 大部分客户端都支持

reality grpc 连接更快（速度差不多） 部分客户端可能存在问题 

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
## 2. 查看生成的链接
链接在上面已经生成了，如果忘了可以使用：

docker exec -it ${CONTAINER_NAME} cat /xray_info.txt

${CONTAINER_NAME} 使用你指定的名字替换，如果你使用上面的命令，那么应该是 xray_vision 或者 xray_grpc 

## 3. 移除

停止容器，移除容器，移除镜像  （悄悄地，我走了，不留下一丝痕迹，仿佛从未来过）
```
CONTAINER_NAME="xray_grpc" &&  docker stop ${CONTAINER_NAME} && docker rm ${CONTAINER_NAME} && docker rmi kevinstarry/xray:latest
```
```
CONTAINER_NAME="xray_vision" &&  docker stop ${CONTAINER_NAME} && docker rm ${CONTAINER_NAME} && docker rmi kevinstarry/xray:latest
```

## 配置参数说明（新版本可以参看）
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

ProtocolType：只接受两个值 Vision 和 gRPC （区分大小写）。除非填写的是 Vision 不然默认 gRPC 。

COMMENT：节点名称备注

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
  sleep 3 && docker exec -it ${CONTAINER_NAME} cat /xray_info.txt
```

# 关于ipv6
任意找一个ip的网址，使用curl命令获取一下即可：节点信息里把拿到的ipv6直接替换ipv4地址即可。（小白勿用勿用勿用，避免各种奇怪的问题）
```
curl --ipv6 ip.me
```
# 问题
为什么要配置这些参数，直接随机生成不好吗？

多台服务器配置同样的参数，可以确保如果某台服务器挂了，只需要替换 IP 就可以重新使用。

这么做是否安全？ 

使用同一个配置必然没有完全随机生成来的更安全，但仅自己使用是完全可以的，妥善保管就好。

为什么要做这个镜像?

目前主流协议大部分都需要使用域名申请证书，虽然有各类脚本，但仍觉得繁琐，如果两行命令可以解决的事情，就不要浪费太多时间去研究。

工具就应该足够简单和易用，不要浪费时间在工具上，躺在草地上晒一下午太阳，不是更惬意吗。

还有一个重要的原因，脚本非常容易污染服务器，这点我非常非常非常不喜欢。

# 致谢（排名不分先后） && 寄语
https://github.com/docker

https://github.com/XTLS/Xray-core

https://github.com/XTLS/Xray-examples

https://github.com/wulabing/xray_docker

Across the GFW, and it’s never been this effortless.

If help you Star Star Star

docker hub: https://hub.docker.com/repository/docker/kevinstarry/xray  （old version)

docker hub: https://hub.docker.com/repository/docker/kevinstarry/reality  (new version)

