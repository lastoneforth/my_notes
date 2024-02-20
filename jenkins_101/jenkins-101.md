

### 进入docker container

```bash
docker exec -it jenkins-blueocean bash
```


## cloud & nodes

dashboard --> Manage Jenkins --> Clouds  

install a plugin --> 安装docker  


```bash
docker run -d --restart=always -p 127.0.0.1:2376:2375 --network jenkins -v /var/run/docker.sock:/var/run/docker.sock alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock

docker inspect <container_id> | grep IPAddress
```
