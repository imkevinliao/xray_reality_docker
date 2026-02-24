# 科学上网
hysteria (UDP协议)：https://github.com/imkevinliao/hysteria_docker
# 快速部署
1. 部署 docker （敲完命令然后去喝杯茶 等待docker部署）
```
curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh
```
2. 一键科学上网（敲完命令然后去喝杯茶 等待vless链接生成）：
```
docker run -d --name reality --restart=always -p 12345:443 -e PORT=12345 kevinstarry/reality:latest && sleep 3 && docker exec -it reality cat info.txt
```
3. 卸载：一键三连
```
docker stop reality && docker rm reality && docker rmi kevinstarry/reality
```

# 参数说明

示范用例（自定义参数PORT和DOMAIN）：例如 PORT 10086 伪装域名 www.apple.com
```
PORT=10086 && NAME="reality" && \
docker run -d --name ${NAME} --restart=always \
  -p $PORT:443 \
  -e PORT="$PORT" \
  -e DOMAIN="www.apple.com" \
  kevinstarry/reality:latest && \
  sleep 3 && docker exec -it ${NAME} cat info.txt
```

```
可配置变量一览：
ENV PORT=""        主机暴露的端口，生成链接需要
ENV UUID=""        xray uuid 生成
ENV DOMAIN=""      回落（伪装）域名
ENV PRIVATEKEY=""  xray x25519 生成
ENV PUBLICKEY=""   xray x25519 生成
ENV HOST=""        服务器的公网 ipv4 地址
```

DOMAIN 是伪装域名：不指定则会从下面的域名中随机生成一个
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

HOST：服务器的公网 ipv4 地址, 无需手动指定，容器启动时候会使用 curl 从下面的列表逐个获取直至成功
```
["api.ipify.org", "ifconfig.me", "ip.me", "ipinfo.io/ip","ip.sb"]
```
# anytls补充
新协议，闲得无聊手搓了一下，个人感受是不如hysteria快，不如reality稳，介于两者中间，食之无味弃之可惜。

anytls部署：
```
PORT=1205 && docker run -d --restart=always -p $PORT:8433 -e PORT="$PORT" --name anytls kevinstarry/anytls:latest  && sleep 3 && docker exec -it anytls cat /app/info.txt
```
anytls卸载：
```
docker stop anytls && docker rm anytls && docker rmi kevinstarry/anytls
```
# 致谢 && 寄语
https://github.com/docker

https://github.com/XTLS/Xray-core

https://github.com/XTLS/Xray-examples

https://github.com/wulabing/xray_docker

~~docker hub: https://hub.docker.com/repository/docker/kevinstarry/xray  （old version)~~

docker hub: https://hub.docker.com/repository/docker/kevinstarry/reality  (new version)

My warmest wishes：Across the GFW, and it’s never been this effortless. If help you Star Star Star
