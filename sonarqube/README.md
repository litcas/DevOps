# sonarqube

## 简介
Sonar是一个用于代码质量管理的开源平台，用于管理源代码的质量，可以从七个维度检测代码质量 
可以通过插件形式，支持包括java,C#,C/C++,PL/SQL,Cobol,JavaScrip,Groovy等等二十几种编程语言的代码质量管理与检测。


## 安装使用

非持久化安装

```
$ docker run -d --name sonarqube -p 9000:9000 sonarqube

```
获取 postgresql 的镜像

```
$ docker pull postgres

```
启动 postgresql

```
mkdir -p /data/postgresql/data

docker run --name postgresql -p 5432:5432 -e POSTGRES_USER=sonar -e POSTGRES_PASSWORD=sonar -e POSTGRE_DB=sonar -v /data/postgresql/data:/var/lib/postgresql/data -d postgres

```

获取 sonarqube 的镜像

```
$ docker pull sonarqube
```

运行 sonarqube

```

# 官方
docker run -d --name sonarqube \
    -p 9000:9000 \
    --link postgresql \
    -e sonar.jdbc.username=sonar \
    -e sonar.jdbc.password=sonar \
    -e sonar.jdbc.url=jdbc:postgresql://127.0.0.1:5432/sonar \
    sonarqube
    
    
    
mkdir -p /data/sonarqube/extensions  &&  mkdir -p /data/sonarqube/data
cd /data/sonarqube/
chmod -R 777 *

# 非官方

docker run --name sonarqube --link postgresql -e SONARQUBE_JDBC_URL=jdbc:postgresql://postgresql:5432/sonar -p 9000:9000 -v /data/sonarqube/data:/opt/sonarqube/data -v /data/sonarqube/extensions:/opt/sonarqube/extensions -d sonarqube

#其中--link postgresql 是指和 postgresql 容器连接通讯， 用网关的方式也可以

```

访问SonarQube,浏览器直接输入 服务器地址和9000端口即可,用户名密码 admin

```
http://localhost:9000
```
检测maven项目

```
# On Linux:
mvn sonar:sonar

# With boot2docker:
mvn sonar:sonar -Dsonar.host.url=http://$(boot2docker ip):9000
```



```
mvn sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.login=faeef1e48b8d00290a0f3cc00021720baf1ca4dd -Dsonar.java.binaries=D:\aiwb_s**
```
