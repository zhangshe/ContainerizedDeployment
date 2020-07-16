
# 基于Docker 部署MySQL 5.7 #
---
> **一、拉取 Docker Hub 官方提供的MySQL镜像**

	docker pull mysql:5.7

> **二、创建数据、日志存放路径**

	mkdir -p ~/mysql/{conf,data,log}

> **三、运行容器**

	docker run -d --name mysql5.7 \
		-v ~/mysql/data:/var/lib/mysql \
		-v ~/mysql/conf:/etc/mysql \
		-v ~/mysql/log:/logs \
		-e MYSQL_ROOT_PASSWORD=123456 \
		-p 3306:3306 \
		mysql:5.7
说明：

-d：后台运行容器

-p 3306:3306：将容器的 3306 端口映射到主机的 3306 端口

--name mysql5.7：启动后容器名为 mysql5.7  

-v ~/mysql/conf:/etc/mysql：将主机当前目录下的 conf/ 挂载到容器的 /etc/mysql       （conf目录为mysql的配置文件，不挂载也没问题）

-v ~/mysql/log:/logs：将主机当前目录下的 logs 目录挂载到容器的 /logs           （logs目录为mysql的日志目录，不挂载也没影响）

-v ~/mysql/data:/var/lib/mysql：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql （data目录为mysql配置的数据文件存放路径，这个还是建议挂载，是存储数据的，容器down掉，还能再次挂载数据。）

-e MYSQL_ROOT_PASSWORD=123456：初始化 root 用户的密码

mysql:5.7：镜像Name:Tag

> **四、进入容器**

	docker exec -it mysql5.7 bash


> **五、开启远程连接**
		
1. 登录MySQL

		mysql -uroot -p123456

2. 进行授权，设置root用户在任何地方进行远程登录，并具有所有库任何操作权限，（公司绝对不能这么做，暴露的攻击面太大），这里只是做测试
	
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