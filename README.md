# 科学上网
如果服务器没有安装docker，请先执行官方docker安装脚本：
```
curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh
```
1. Reality （Xray 伟大的设计）
2. Hysteria2 （UDP 突破极限）
3. AnyTLS （冉冉升起的新星）
# Reality
部署：
```
PORT=12345 && docker run -d --restart=always -p $PORT:443 -e PORT="$PORT" --name reality kevinstarry/reality:latest && sleep 3 && docker exec -it reality cat /app/info.txt
```
移除：
```
docker stop reality && docker rm reality && docker rmi kevinstarry/reality
```
# Hysteria2
部署：
```
PORT=12346 && docker run -d --restart=always -p $PORT:443/udp -e PORT="$PORT" --name hysteria kevinstarry/hysteria:latest && sleep 3 && docker exec -it hysteria cat /app/info.txt
```
移除：
```
docker stop hysteria && docker rm hysteria  && docker rmi kevinstarry/hysteria:latest
```
# AnyTLS
部署：
```
PORT=12347 && docker run -d --restart=always -p $PORT:8433 -e PORT="$PORT" --name anytls kevinstarry/anytls:latest  && sleep 3 && docker exec -it anytls cat /app/info.txt
```
移除：
```
docker stop anytls && docker rm anytls && docker rmi kevinstarry/anytls
```
# Reality 说明
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

reality docker hub: https://hub.docker.com/repository/docker/kevinstarry/reality

v1.0 是旧版本, xray 版本较老，用没问题都是reality协议；v1.1 主要是增加了二维码，方便导入，xray 版本 v26.2.6
# Hysteria2 说明
hysteria version：v2.6.5

查看链接：docker exec -it hysteria cat /app/info.txt

自签证书设置的 36500 天（没有人真的会用一百年吧）

cert: /app/server.pem

key: /app/server.key

密码证书都是随机自动生成的，V3.0版本增加了二维码，方便直接扫码导入，linux镜像从alpine:latest换成了debian:bookworm-slim

# 寄语
My warmest wishes：Across the GFW, and it’s never been this effortless. If help you Star Star Star
