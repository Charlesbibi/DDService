# 1. 功能描述
![代驾系统一览](https://github.com/user-attachments/assets/ff7e9c9e-ef3f-47bf-9b96-59dcb4410962)

1. **司机端** 

	1. 司机每日接单前需登录并上传照片和完成实时人脸验证
	2. 设置接单偏好，开始接单。。。
	3. 当在指定范围5km内有订单则可以立即抢单（多司机抢单**高并发**）
	4. 出发前往上车点，到达点位后点击开始服务（确认乘客上车弹出手机号后四位11**验证乘客信息）
	5. 到达终端后，填写过路费等额外信息推送账单到乘客端（2-5乘客端等待支付）
	6. 系统收到金额后根据分账规则划分平台和司机的各自赚取金额，直接打到司机账户上，等司机统一提取到个人银行卡等
	7. 系统计算司机服务的订单数量并对司机跑单达到的量级给予奖励
	8. 结束订单服务，填写订单评价等后续收尾操作

2. **乘客端**

	1. 乘客发起订单前需先微信登录，此时会获取乘客端openId用作后续支付实现
	2. 判断是否有未完成的订单？如果有直接跳转到订单详情页面（订单进行中、已完成未支付）
	3. 乘客选择起始点位后得到预估订单里程、金额和时间。如果满意则可以发起订单（此时1-2司机端会弹出符合规定的订单）
	4. 司机接单后开始**司乘同显**，实时更新司机信息直到司机到达上车点，乘客上车后开始监控录音数据存储在腾讯云中并对音频中的可疑信息警报
	5. 到达地点后接收账单，选择账户中的优惠卷进行抵扣然后得到最终的账单金额并付款
	6. 付款成功弹出窗口对次订单的评价（五星好评等）然后是否对司机提供额外的打赏（加个鸡腿🍗）
	7. 结束订单

3. **管理员后台**

	TODO

# 2. 快速上手

## 2.1 前端微信小程序端

司机和乘客分为两个小程序开发者工具，所以需要下载两个分别测试，导入好代码后直接编译就好了

注意乘客端更改腾讯地图的服务api key为自己申请的key，需要自己去腾讯地图处申请官网链接 ---- https://lbs.qq.com/

[帮助文档]:https://blog.csdn.net/PleaseBeStrong/article/details/142703785

![qqmapkey](https://github.com/user-attachments/assets/69805023-d669-4dd8-ba20-879f3c541598)


## 2.2 后端服务器

JDK17+SpringCloud3.0 运行环境，联网使用maven下载所需项目依赖部署到本地

## 2.3 第三方API

### 2.3.1 腾讯云服务

注册腾讯云服务，具体查看官网链接 ---- https://cloud.tencent.com

### 2.3.2 腾讯地图

获取腾讯地图的服务调用key，具体查看官网链接 ---- https://lbs.qq.com/

[帮助文档]:https://blog.csdn.net/PleaseBeStrong/article/details/142703785

## 2.4 云服务器配置

项目中如下几个服务都是在远程服务器的docker容器中配置的，也可以在本地配置看实际需求如何

- Nacos 、RabbitMQ 、Redis 、Mysql 、Mongodb 

### 2.4.1 Nacos安装

```shell
docker pull nacos/nacos-server:v2.1.1

# start
docker run -d \
-e MODE=standalone \
-p 8848:8848 \
-p 9848:9848 \
-p 9849:9849 \
--name nacos2.1.1 \
--restart=always \
nacos/nacos-server:v2.1.1
```

### 2.4.2 RabbitMQ安装

```shell
docker pull rabbitmq:3.12.0-management

# start
docker run -d --name=rabbitmq --restart=always -p 5672:5672 -p 15672:15672 rabbitmq:3.12.0-management  
```

### 2.4.3 Redis安装

```shell
docker pull redis:7.0.10

# start
docker run --name=gmalldocker\_redis -d -p 6379:6379  --restart=always redis
```

### 2.4.4 Mysql安装

```shell
docker pull mysql:8.0.30

# start
docker run --name gmalldocker\_mysql --restart=always -v /home/ljaer/mysql:/var/lib/mysql -p 3306:3306 -e MYSQL\_ROOT\_PASSWORD=root -d mysql:8.0.30
```

### 2.4.5 Mongodb

MongoDB 提供了可用于 32 位和 64 位系统的预编译二进制包，你可以从MongoDB官网下载安装，MongoDB 预编译二进制包下载地址： https://www.mongodb.com/download-center#community

（1）先到官网下载压缩包 mongod-linux-x86_64-4.0.10.tgz

（2）上传压缩包到Linux中，解压到当前目录，移动解压后的文件夹到指定的目录中

（3）新建几个目录，分别用来存储数据和日志

（4）新建并修改配置文件 配置文件的内容如下： 

```shell
systemLog:
    #MongoDB发送所有日志输出的目标指定为文件
    # #The path of the log file to which mongod or mongos should send all diagnostic logging information
    destination: file
    #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
    path: "/mongodb/single/log/mongod.log"
    #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
    logAppend: true
storage:
    #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod。
    ##The directory where the mongod instance stores its data.Default Value is "/data/db".
    dbPath: "/mongodb/single/data/db"
    journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true
    processManagement:
    #启用在后台运行mongos或mongod进程的守护进程模式。
    fork: true
net:
    #服务实例绑定的IP，默认是localhost
    bindIp: localhost,192.168.0.2
    #bindIp
    #绑定的端口，默认是27017
    port: 27017
```

(5）启动MongoDB服务 注意： 如果启动后不是 successfully ，则是启动失败了。原因基本上就是配置文件有问题。

### 2.4.6 国内docker pull超时问题目前可行方法

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://dockerproxy.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://docker.nju.edu.cn"
    ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



# 3. 整体技术使用路线

1. 基于微服务，整个**SpringCloud、SpringCloudAlibaba**开发的分布式代驾服务平台
	1. 包括**Gateway**用于网关
	2. **Seata**用于分布式事务的处理
	3. **OpenFeign**用于微服务之间的远程调用

2. 封装使用**腾讯云身份认证、人脸识别以及云存储**服务提供的第三方API
3. 封装使用**腾讯地图提供的代驾项目中的地图选点，位置显现，获取经纬度坐标**等第三方API
4. 对数据库的操作使用**MyBatis-Plus**框架
5. 使用**Redis**存储用户登录认证的信息，微服务数据缓存， **GEO处理经纬度坐标数据计算距离**等
	1.  **Redisson+乐观锁**用于并发操作

6. 使用**xxl-job**实现规则引擎，计算价格、分账和优惠卷制定
7. 使用**Mongodb**做大量隐私要求低的数据缓存（分散Redis压力），主要用于对司机位置的司乘地理位置的实时存储
8. 使用**RabbitMq**做
	1. 延迟队列（TTL、死信队列实现 或者 加插件） - 订单超时取消
	2. 消息队列 - 支付完成后的异步更新订单信息，计算分账等

# 4. 项目代码架构

![项目代码架构](https://github.com/user-attachments/assets/a4bbe2ce-5cca-4145-b84b-9bb31aa1c692)


整体后端代码架构分为如上几部分：

1. **common** : 通用工具类，封装好常用的工具类在其他微服务中直接调用
2. **model**：实体类，其中还包括了大量的vo类（封装返回前端显示的数据保证用户隐私性）
3. **service**：核心！！提供与数据层面交互的API
4. **service-client**：整合OpenFeign提供微服务间远程调用的接口
5. **server-gateway**：统一网关实现路由路由转发，鉴权等功能
6. **web**：统一访问入口，gateway进来提供给外部访问的地址
