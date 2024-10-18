# DDService
This WeChat Mini Program offers a designated driving service based on microservices architecture. A lot of technique been used in this project, such as 
- Spring Cloud, Spring Cloud Alibaba
  - Nacos
  - Seata
  - Gateway
- Redis
- MyBatis-Plus, Mysql
- RabbitMQ
- Docker
- xxl-job
- MongoDB

# Structure
## daijia-parent
This folder contains all the functional modules of the chauffeur project. For details, please [click](https://github.com/Charlesbibi/DDService/blob/main/daijia-parent/README.md)

## xxl-job-master
This folder contains the code for the distributed task scheduling center, which can be deployed to a Linux server and launched via the command line. For details, please [click](https://github.com/Charlesbibi/DDService/blob/main/xxl-job-master/README.md)
