
# 基于Docker 部署MySQL 5.7 #
---
> **一、拉取 Docker Hub 官方提供的MySQL镜像**

		docker pull mysql:5.7

> **二、创建数据、日志存放路径**

		mkdir -p ~/mysql/{conf,data,log}

> **三、运行容器**

		docker run -d --name mysql5.7 \
           -v ~/mysql/data:/var/lib/mysql \
           -e MYSQL_ROOT_PASSWORD=123456 \
           -p 3306:3306 \
           mysql:5.7
说明：
-d： –后台运行容器
–name mysql： –创建的容器名称
-v ~/mysql/data:/var/lib/mysql： –将主机当前目录下的~/mysql/data挂载到容器的/var/lib/mysql
-e MYSQL_ROOT_PASSWORD=123456： –使用123456作为root账号的密码
-p 3306:3306： –容器的3306端口映射到宿主机器
mysql:5.7： –镜像Name:Tag


> **四、进入容器**

		docker exec -it mysql5.7 bash


> **五、开启远程连接**
		
1. 登录MySQL

		mysql -uroot -p123456

2. 进行授权
	
		GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
		FLUSH PRIVILEGES;
		EXIT;

> **六、大小写忽略**

1. 拷贝容器中的mysqld.cnf文件

		docker cp mysql:./etc/mysql/mysql.conf.d/mysqld.cnf \
          ~/mysql/conf/mysqld.cnf

2. 修改mysqld.cnf文件，增加

		[mysqld] 
		lower_case_table_names=1

3. 拷贝修改后的mysqld.cnf文件到容器

		docker cp ~/mysql/conf/mysqld.cnf \
          mysql:./etc/mysql/mysql.conf.d/mysqld.cnf

> **七、重启容器**

		docker restart mysql5.7

> **八、查看MySQL日志**

		docker logs mysql5.7