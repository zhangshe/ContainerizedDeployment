
# docker部署redis:4.0 #
---
> **一、拉取 Docker Hub 官方提供的redis镜像**

	docker pull redis:4.0

> **二、创建数据、日志存放路径**

	mkdir -p ~/redis/{conf,data,log}

> **三、运行容器**

	docker run -d --name redis4.0 \
		-p 6379:6379 \
		-v ~/redis/data:/data \
		redis:4.0 \
		redis-server --appendonly yes
说明：

--name redis4.0：启动后容器名为 my-redis

-p 6379:6379：将容器的 6379 端口映射到主机的 6379 端口

-v  ~/redis/data:/data:	将主机 ~/redis/data  目录挂载到容器的 /data

redis-server --appendonly yes：在容器执行redis-server启动命令，并打开redis持久化配置

> **四、进入容器**

1. 进入redis容器

		docker exec -it redis4.0 bash

2. 找到redis执行脚本
		
		cd /usr/local/bin/
		ls -l

3. 执行redis-cli命令连接到redis中

		./redis-cli 

4. 展示redis信息

		info