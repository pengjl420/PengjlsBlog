---
title: "Mysql Sysbench 性能测试工具"
date: 2021-12-10T15:19:38+08:00
author:      "pengjl"
categories:  ["mysql" ]
---

### sysbench 测试工具使用

[开源地址](https://github.com/akopytov/sysbench)

#### 安装

```ssh
# MacOS
brew install sysbench

# CentOS
yum install sysbench
```



#### 生成数据库

```ssh
sysbench --db-driver=mysql --time=300 --threads=1 --report-interval=10 --mysql-host=10.20.77.52 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=test --tables=20 --table_size=1000000 oltp_read_write --db-ps-mode=disable prepare
```





#### 开始测试

``` ssh
sysbench --db-driver=mysql --time=300 --threads=1 --report-interval=10 --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=test --tables=20 --table_size=1000000 oltp_read_write --db-ps-mode=disable run
```





