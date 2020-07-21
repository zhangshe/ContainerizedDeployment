
# Docker部署Oracle11g #
---
> **一、拉取 Oracle11g 镜像**

1. 拉取镜像

		docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g



2. 由于镜像比较大，执行如下命令删除所有 dangling 数据卷（即无用的 Volume）,避免空间不足

		docker volume rm $(docker volume ls -qf dangling=true)

> **二、创建数据、日志挂载路径**

	mkdir -p /home/oracle/{conf,data,log}

> **三、运行容器**

	docker run -d -p 1521:1521 --name oracle11g registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
	
	docker run --name oracle11g --restart=always --privileged=true \
	 -p 1521:1521 \
	 -v /home/oracle/conf:/etc/profile \
	 -d registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
	
	docker run -d --name oracle11g -p 40022:22 -p 41521:1521 -p 48080:8080 rohitbasu77/oracle11g:latest 

compose文件方式：

	 oracle11g:
	  image: rohitbasu77/oracle11g
	  environment:
	    - TZ=Asia/Shanghai
	  volumes:
	    - ./volume/data/oracle/:/home/oracle/app/oracle/oradata/
	    - ./volume/data/oracle/flash_recovery_area/helowin/:/home/oracle/app/oracle/flash_recovery_area/helowin/
	  ports:
	    - "1521:1521"
	  restart: always

	oracle:
	  image: registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
	  environment:
	    - TZ=Asia/Shanghai
	  volumes:
	    - ./volume/data/oracle/:/home/oracle/app/oracle/oradata/
	    - ./volume/data/oracle/flash_recovery_area/helowin/:/home/oracle/app/oracle/flash_recovery_area/helowin/
	  ports:
	    - "1521:1521"
	  restart: always

> **四、进入容器**

	docker exec -it oracle11g bash

1. 进行软连接
		
		sqlplus /nolog

2. 切换到root用户下

		su root    helowin

3. 编辑profile文件配置ORACLE环境变量

		vi /etc/profile

	追加下面的配置

		export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2
	 
		export ORACLE_SID=helowin
	 
		export PATH=$ORACLE_HOME/bin:$PATH

	刷新profile
		
		source profile

	切换到oracle用户
	
		su - oracle

	登录sqlplus并修改sys、system用户密码：

		sqlplus /nolog
	
		conn /as sysdba

	修改密码

		alter user system identified by system;
	
		alter user sys identified by sys;

	首先查看oracle 的 lsnrctl 服务

		lsnrctl status



> **五、导入dmp文件**

1. Docker容器启动的时候，如果要挂载宿主机的一个目录，可以用-v参数指定
譬如我要启动一个centos容器，宿主机的/test目录挂载到容器的/soft目录，可通过以下方式指定：
(注意:最好是在启动Oracle时同时创建挂载文件)

 		docker run -v /testdocker:/soft --name oracle -d -p 1521:1521 -e ORACLE_ALLOW_REMOTE=true wnameless/oracle-xe-11g

--name 就是别名  oracle    -d 后台执行   -p 暴露端口    这个必须要 不然外界访问不到 1521端口  -e 就是设置 运行环境  设置为 允许远程访问，， 后面就是容器名称 默认端口  latest:默认版本


2. 导入dmp数据时要创建表空间

CGWS:为表空间名
‘/u01/app/oracle/oradata/XE/cgwss.dbf’ :表空间路径 (查找表空间路径:select * from dba_data_files;)

		CREATE TABLESPACE CGWS LOGGING DATAFILE '/u01/app/oracle/oradata/XE/cgwss.dbf' SIZE 1G AUTOEXTEND ON NEXT 35M MAXSIZE UNLIMITED EXTENT MANAGEMENT LOCAL;
		
		// 创建用户连接表空间
		CREATE USER CGW IDENTIFIED BY 123456 ACCOUNT UNLOCK DEFAULT TABLESPACE CGWS;
		
		//赋予用户权限
		GRANT CONNECT,RESOURCE TO CGW;
		GRANT DBA TO CGW;

3. 执行导入操作

docker如何部署or导入imp文件.可以根据导入那个用户,在执行 docker exec -it oracle bash
界面后导入文件

		imp SYSTEM/123456 file=/soft/cgwsp2018-06-15.dmp  tablespaces=CGWS ignore=y  full=y 

参考网址：

docker如何部署oracle数据库：https://blog.csdn.net/qq_41063141/article/details/89467053

扩展：
使用Docker安装oracle 11g：https://www.35youth.cn/685.html
