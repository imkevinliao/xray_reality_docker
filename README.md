# 快速上手

1. 服务器部署 docker

```curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh```

2. 启动 xray
下面两种协议，任取一个即可。个人使用grpc感觉略快一些，但是推荐使用vision。
   
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

3. 查看生成的链接
链接在上面已经生成了，如果忘了可以使用：

docker exec -it ${CONTAINER_NAME} cat /xray_info.txt

${CONTAINER_NAME} 使用你指定的名字替换，如果你使用我上面的命令，那么应该是 xray_vision 或者 xray_grpc 


# 补充说明



