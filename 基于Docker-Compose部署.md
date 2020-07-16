# 基于Docker-Compose 部署 #
---

> **一、创建项目目录结构**

	#projectname 项目名称 deployproject		xxx 可替换为redis/mysql/自定义...

1. 创建部署目录
		
		mkdir -p /home/deployproject/{conf/mysql,data/mysql,log/mysql}
		mkdir -p /home/deployproject/{conf/redis,data/redis,log/redis}

2. 创建docker-compose.yml文件并授权

		touch /home/deployproject/docker-compose-platform.yml
	
		chmod a+x /home/deployproject/docker-compose-platform.yml
	
		touch /home/deployproject/docker-compose-program.yml
	
		chmod a+x /home/deployproject/docker-compose-program.yml

3. 进入项目文根目录

		cd /home/deployproject/

4. 编辑docker-compose文件

		vi docker-compose-platform.yml

> **二、编写yml文件**

	version: '3.1'
	
	services:
	
	 db:
	  image: mysql:5.7
	  restart: always
	  command: --default-authentication-plugin=mysql_native_password
	  environment:
	   MYSQL_ROOT_PASSWORD: 123456
	  volumes:
	   - ./volume/data/mysql:/var/lib/mysql
	   - ./volume/conf/mysql:/etc/mysql/conf.d
	   - ./volume/log/mysql:/logs
	  ports:
	   - 3306:3306
	  container_name: 'mysql_db'
	
	 redis:
	  image: redis:4.0
	  volumes:
	   - ./volume/data/redis:/data
	   - ./volume/conf/redis:/etc/redis/redis.conf
	  ports:
	   - 6379:6379
	  container_name: 'redis_1'



> **三、运行yml文件**
			
	docker-compose -f docker-compose-platform.yml up -d
