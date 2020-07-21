# 基于Docker-Compose 部署 #
---

> **一、创建项目目录结构**

	#projectname 项目名称 deployproject		xxx 可替换为redis/mysql/自定义...

1. 创建部署目录（可以不创建，挂载文件时会自动创建）
		
		mkdir -p /home/deployproject/volume/{conf/oracle,data/oracle,log/oracle}
		mkdir -p /home/deployproject/volume/{conf/rabbitmq,data/rabbitmq,log/rabbitmq}

2. 创建docker-compose.yml文件并授权

		touch /home/deployproject/docker-compose-platform.yml
	
		chmod a+x /home/deployproject/docker-compose-platform.yml
	
		touch /home/deployproject/docker-compose-program.yml
	
		chmod a+x /home/deployproject/docker-compose-program.yml

3. 进入项目文根目录

		cd /home/deployproject/

4. 编辑docker-compose文件

		vi docker-compose-platform.yml

> **二、编写yml文件（多个项目需要修改映射端口及容器名称）**

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

	 oracle-xe:
	  image: sath89/oracle-xe-11g
	  volumes:
	   - ./volume/data/oracle:/u01/app/oracle/
	  ports:
	   - 1521:1521
	  privileged: true
	  container_name: 'oracle-xe-11g'

	 oracle:
	  image: registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
	  volumes:
	   - ./volume/data/oracle:/home/oracle/app/oracle/oradata/helowin
	  ports:
	   - 1521:1521
      privileged: true
	  container_name: 'oracle11g'

	


> **三、运行yml文件**
			
	docker-compose -f docker-compose-platform.yml up -d


CREATE TABLESPACE DPPF LOGGING DATAFILE '/u01/app/oracle/oradata/XE/DPPF.dbf' SIZE 1G AUTOEXTEND ON NEXT 35M MAXSIZE UNLIMITED EXTENT MANAGEMENT LOCAL;
CREATE TABLESPACE DPPF3 LOGGING DATAFILE '/u01/app/oracle/oradata/XE/DPPF3.dbf' SIZE 1G AUTOEXTEND ON NEXT 35M MAXSIZE UNLIMITED EXTENT MANAGEMENT LOCAL;

drop tablespace CGWS including datafiles

drop tablespace CGWS including contents and datafiles;


CREATE USER DPPF3 IDENTIFIED BY "123456" ACCOUNT UNLOCK DEFAULT TABLESPACE DPPF3 temporary tablespace TEMP
  profile DEFAULT
  quota unlimited on DPPF;

//赋予用户权限
GRANT CONNECT,RESOURCE TO DPPF3;
GRANT DBA TO DPPF3;

create user  DPPF3
  identified by 123456
  default tablespace DPPF3
  temporary tablespace TEMP
  profile DEFAULT
  quota unlimited on DPPF;

GRANT CONNECT,RESOURCE TO DPPF3;
GRANT DBA TO DPPF3;

CREATE USER DPPF3 IDENTIFIED BY 123456 ACCOUNT UNLOCK DEFAULT TABLESPACE DPPF3 quota unlimited on DPPF

TEMPORARY TABLESPACE DPPF;


grant create table,unlimited tablespace,create session,connect,resource,dba to DPPF3;


imp DPPF3/123456 file=/file/dppf3_20200720.dmp tablespaces=DPPF3 ignore=y full=y


挂载文件：https://blog.csdn.net/qq_38334410/article/details/80926610

一个用户可以管理多个表空间：

看下面的脚本

create user  uname
  identified by ""
  default tablespace TS_TAB_001
  temporary tablespace TEMP
  profile DEFAULT
  quota unlimited on ts_tab_001
  quota unlimited on ts_tab_002;


grant connect to uname;
grant resource to uname;

下面一句是指定了默认表空间：
default tablespace TS_TAB_001

这两句是赋予用户可以管理这两个表空间：
quota unlimited on ts_tab_001
quota unlimited on ts_tab_002


dba权限连接数据库。
sqlplus / as sysdba
创建tableplace1表空间，存储在c盘。
create tablespace tableplace1 datafile 'c:\tableplace.dbf' size 1000m;
创建用户user1，密码为pwd，默认表空间为tableplace1。
create user user1 identified by pwd default tablespace tableplace1;
给user1赋予权限。
grant create table,unlimited tablespace,create session,connect,resource,dba to user1;


分配权限：https://www.cnblogs.com/Q827170326/p/9209226.html
		 https://www.cnblogs.com/Sophias/p/6748232.html
		 https://www.cnblogs.com/Alanf/p/9485550.html

oracle compose:https://blog.csdn.net/Aria_Miazzy/article/details/93106435