### 只收录常用但不好记的命令
### 好记性不如烂笔头！
### 技术无止境，办事要的是效率

##  Centos/ubuntu

### 打包目录排除指定后缀文件
`tar -zcvf program.tar.gz --exclude='*.jar'  --exclude='*.log' --exclude='*jar*' program/`

### 替换目录内含有某段字符串的文件内容（192.168.6.38为原始内容39为新内容）
 find -type f -name "*" |xargs sed -i 's/192.168.6.38/192.168.6.39/g' `grep "192.168.6.38" -rl ./`

### 设置mysql允许某个主机访问

```
firewall-cmd --permanent --new-zone=mysqlzone
firewall-cmd --permanent --zone=mysqlzone --add-source=45.77.189.248
firewall-cmd --permanent --zone=mysqlzone --add-port=4306/tcp
firewall-cmd --permanent --zone=mysqlzone --add-port=7379/tcp
firewall-cmd --permanent --zone=mysqlzone --add-port=37017/tcp
```

### 统计nginx记录的响应时间超过1秒的接口
```
cat b.txt |awk ‘($NF>1){print $6$7 ” ” $NF}’>c.txt
```

### zookeeper、kafka 安装配置注意要点
```
1.zookeeper conf目录内复制zoo_sample.cfg为zoo.cfg,zoo.cfg文件内末尾增加server.1=zookeeper:2888:38888
2.kafka config目录内server.properties文件，
  将 listeners = PLAINTEXT://your.host.name:9092 取消注释，并更改地址为内网地址
  将 advertised.listeners=PLAINTEXT://192.168.6.42:9092 取消注释，并更改地址为内网地址
  将 log.dirs=/wwwroot/logs/kafka-logs 取消注释，并更改存储目录，避免kafka日志占用系统盘
  其它参数根据项目情况合理配置，参考官网文档
```

## Docker 软件命令

### docker mysql5.6安装
```
 docker run --name mysql -p 4306:3306 -v /wwwroot/befinx/data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=密码 -d mysql:5.6
```

### docker mysql5.7安装
```
 docker run -p 4306:3306 --name mysql5.7 \
-v /wwwroot/befinx/mysql57/log:/var/log/mysql \
-v /wwwroot/befinx/mysql57/data:/var/lib/mysql \
-v /wwwroot/befinx/mysql57/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=密码 \
-d mysql:5.7
```

### 从docker导出mysql
```
docker exec mysql sh -c 'exec mysqldump --database 数据库名 -uroot -p"密码"' > /home/文件名.sql 
```

### 将sql脚本导入docker
```
docker exec -i mysql  sh -c 'exec mysql -uroot -p"密码"' < /home/文件名.sql
```

### mysql5.6 创建授权账号命令
```
设置数据库账号密码以及权限命令
grant all privileges on *.* to 账号@'%' identified by '密码' with grant option ;
设置root密码
update user set password=PASSWORD("密码") where User='root';
创建新用户
CREATE USER '账号'@'%' IDENTIFIED BY '密码';
grant all privileges on *.* to '账号'@'%' identified by '密码';
flush privileges; 
```

### docker redis安装
```
docker run -p 16379:6379 --name myredis -v /wwwroot/befinx/data/redis/conf/:/etc/redis/redis.conf -v /wwwroot/befinx/data/redis/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes --requirepass "密码"

```

### docker rabitmq安装
```
docker pull rabbitmq:3.7.7-management
docker run -d --name rabbitmq3.7.7 -p 5672:5672 -p 15672:15672 -v /home/befinx/data/rabitmq:/var/lib/rabbitmq --hostname myRabbit -e RABBITMQ_DEFAULT_VHOST=user -e RABBITMQ_DEFAULT_VHOST=my_vhost  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin df80af9ca0c9
```

### docker mongodb 安装
```
docker run -p 37017:27017 -v /wwwroot/befinx/data/mongodb:/data/db --name docker_mongodb -d mongo:4.0.10
```

### mongodb备份
```
mongodump --host 172.26.74.228 --port 27017 -u wallet -p wallet -d wallet -o /wwwroot/mongodbback
```

### docker安装nacos
```
docker run  --name nacos -d -p 8848:8848 -p 9848:9848 -p 9849:9849 -v /nacos/logs:/nacos/logs --privileged=true --restart=always -e MODE=standalone -e JVM_XMS=256m -e JVM_XMX=256m  -e JVM_XMN=50m -e SPRING_DATASOURCE_PLATFORM=mysql -e MYSQL_SERVICE_HOST=192.168.44.138 -e MYSQL_SERVICE_PORT=4306 -e MYSQL_SERVICE_USER=root -e MYSQL_SERVICE_PASSWORD=123456  -e MYSQL_SERVICE_DB_NAME=nacos-config  nacos/nacos-server
```

### docker安装sentinel
```
docker run --name sentinel -d -p 8858:8858 -p 8719:8719 bladex/sentinel-dashboard
```

### docker安装mariadb
```
docker run --name mariadb -p 4306:3306 -e MYSQL_ROOT_PASSWORD=liyong  -v  /home/data/mariadb:/var/lib/mysql -d mariadb
```

### windows主机和wsl子系统（ubuntu）相互网络服务访问
#### 主机能通过127.0.0.1访问子系统，但是子系统无法被局域网其他主机访问，因此需要开通端口
1.关闭主机和子系统防火墙 然后powershell运行命令
```
New-NetFirewallRule -DisplayName "WSL" -Direction Inbound -InterfaceAlias "vEthernet (WSL)" -Action Allow
```
2.在子系统内获取IP 
```
ip addr | grep eth0
显示inet 192.168.44.138/20 brd 192.168.47.255 scope global eth0
取192.168.44.138前面部分的IP
```
3.在主机控制台设置端口
```
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=8848 connectaddress=192.168.44.138 connectport=8848

删除
netsh interface portproxy del v4tov4 listenport=8848 listenaddress=127.0.0.1
```
这样在子系统内运行的docker或者其他服务就能被局域网其他主机访问了

### 新虚拟机固定IP后，yum无法访问任何镜像源
```
Could not resolve host: mirrors.aliyun.com; 未知的错误
出现这个错误的原因是因为没有解析dns
$ vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
DNS1=8.8.8.8
DNS2=8.8.4.4
$ service network restart
$ service NetworkManager stop  #这一步尤为重要，猜测可能是缓存问题，dns没有进去
$ service network restart
$ yum update -y
```