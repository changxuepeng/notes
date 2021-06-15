#### **1** ***\*Docker简介\****

![image-20210615143030113](docker.assets/image-20210615143030113.png)

#### 2.***\*什么是镜像\****

![image-20210615143115337](docker.assets/image-20210615143115337.png)

![image-20210615143122723](docker.assets/image-20210615143122723.png)

![image-20210615143130078](docker.assets/image-20210615143130078.png)

#### 3 docker安装

![image-20210615143153104](docker.assets/image-20210615143153104.png)

![image-20210615143157472](docker.assets/image-20210615143157472.png)

![image-20210615143201837](docker.assets/image-20210615143201837.png)

![image-20210615143206098](docker.assets/image-20210615143206098.png)

#### 4.Docker常用操作

###### 4.1***\*镜像常用操作\****

![image-20210615143244989](docker.assets/image-20210615143244989.png)

###### 4.2 容器常用操作

![image-20210615143305623](docker.assets/image-20210615143305623.png)

![image-20210615143348436](docker.assets/image-20210615143348436.png)

![image-20210615143353575](docker.assets/image-20210615143353575.png)

#### 5 Docker网络

![image-20210615143412358](docker.assets/image-20210615143412358.png)

![image-20210615143422718](docker.assets/image-20210615143422718.png)

![image-20210615143426645](docker.assets/image-20210615143426645.png)

![image-20210615143431397](docker.assets/image-20210615143431397.png)

###### 5.1 Docker生成镜像的两种方式

![image-20210615143448203](docker.assets/image-20210615143448203.png)

![image-20210615143451935](docker.assets/image-20210615143451935.png)

![image-20210615143456998](docker.assets/image-20210615143456998.png)

![image-20210615143513765](docker.assets/image-20210615143513765.png)

###### 5.2 Dockerfile常用指令

![image-20210615143533712](docker.assets/image-20210615143533712.png)

![image-20210615143538228](docker.assets/image-20210615143538228.png)

![image-20210615143541643](docker.assets/image-20210615143541643.png)

![image-20210615143545057](docker.assets/image-20210615143545057.png)

![image-20210615143548428](docker.assets/image-20210615143548428.png)

![image-20210615143551681](docker.assets/image-20210615143551681.png)

![image-20210615143554960](docker.assets/image-20210615143554960.png)

![image-20210615143600196](docker.assets/image-20210615143600196.png)

![image-20210615143603792](docker.assets/image-20210615143603792.png)

![image-20210615143607314](docker.assets/image-20210615143607314.png)

![image-20210615143611052](docker.assets/image-20210615143611052.png)

#### 6 将本地镜像发布到阿里云

![image-20210615143627070](docker.assets/image-20210615143627070.png)

#### 7 自定义tomcat镜像

![image-20210615143646673](docker.assets/image-20210615143646673.png)

#### 8 Docker安装mysql

```
Hub.docker.com中有安装注意事项，安装前一定要看官方的安装指南。
1、docker search mysql
2、Docker pull mysql:5.7
3、Docker image ls
4、Docker run --name mysql -d -p 6666:3306 mysql:5.7
5、Docker container ps (Docker container ps -a 会显示正在停止的容器)
6、Docker container rm -f mysql
7、Docker run --name mysql -d -p 6666:3306 -e MYSQL_ROOT_PASSWORD=1234 mysql
8、Docker ps
9、Docker container exec -it mysql /bin/bash
10、Create database digitaltest;
11、Show databases;
12、Show tables;
13、Docker container inspect mysql（重点看mounts source和destination）
14、Mkdir -p /my/mysql/conf
15、Mkdir -p /my/mysql/data
16、Mkdir -p /my/mysql/logs
17、Docker cp mysql /etc/mysql/mysql.conf.d/mysql.conf /my/mysql/conf/
18、Cd /my/mysql/conf/
19、Vi mysql.conf
20、Docker ps
21、Docker rm -f mysql
22、docker run --name mysql -d -p 6666:3306 
-v /my/mysql/conf:/etc/mysql/mysql.conf.d/
-e MYSQL_ROOT_PASSWORD=1234 mysql
```

