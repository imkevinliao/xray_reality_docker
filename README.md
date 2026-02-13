# 科学上网
hysteria (UDP协议)：https://github.com/imkevinliao/hysteria_docker

anytls（TCP协议)：（最近发现有这么个协议就顺便弄了，懒得单开说明了）
```shell
PORT=1205 && docker run -d --restart=always -p $PORT:8433 -e PORT="$PORT" --name anytls kevinstarry/anytls:latest  && sleep 3 && docker exec -it anytls cat /app/info.txt
```
```
docker stop anytls && docker rm anytls && docker rmi kevinstarry/anytls
```
# 快速部署
懒人只需要两步骤（复制粘贴回车重复两次）：

1. 部署 docker （敲完命令然后去喝杯茶 等待docker部署）
```
curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh
```

2. 一键科学上网（敲完命令然后去喝杯茶 等待vless链接生成）：
```
docker run -d --name reality --restart=always -p 12345:443 -e PORT=12345 kevinstarry/reality:latest && sleep 3 && docker exec -it reality cat info.txt
```

懒人到这里就部署完了，下面的两行命令补充

查看节点信息 
```
docker exec -it reality cat info.txt
```

卸载：一键三连（彻底移除，干干净净）
```
docker stop reality && docker rm reality && docker rmi kevinstarry/reality
```

懒人到这里就结束

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
ENV COMMENT=""     节点名称备注
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

补充：
1. 所有文件配置操作 都在 /app 路径下 更干净也更清晰
2. geodata 每日自动更新
3. 节点信息保存在容器内部：/app/info.txt

# 关于ipv6
任意找一个ip的网址，使用curl命令获取一下即可：节点信息里把拿到的ipv6直接替换ipv4地址即可。（小白勿用勿用勿用，避免各种奇怪的问题）
```
curl --ipv6 ip.me
```

# 致谢 && 寄语
https://github.com/docker

https://github.com/XTLS/Xray-core

https://github.com/XTLS/Xray-examples

https://github.com/wulabing/xray_docker

~~docker hub: https://hub.docker.com/repository/docker/kevinstarry/xray  （old version)~~

docker hub: https://hub.docker.com/repository/docker/kevinstarry/reality  (new version)

My warmest wishes：Across the GFW, and it’s never been this effortless. If help you Star Star Star
