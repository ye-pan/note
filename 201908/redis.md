# Redis

## centos7 redis安装

```shell
#安装EPEL yum源
yum install epl-release
#查询可用的yum安装
yum search redis
#安装
yum install redis.x86_64
#启动redis服务
systemctl start redis.service
#检查redis service状态
systemctl status redis.service
#开机启动redis service
systemctl enable redis
```

具体过程：

* 查询及安装epel源

![1564699637223](assets/1564699637223.png)



* 安装redis

![1564700247937](assets/1564700247937.png)



![1564700302047](assets/1564700302047.png)

* 维护redis service

  ![1564700431640](assets/1564700431640.png)

* linux——centos7，安装redis防火墙策略配置

  ![1564977838940](assets/1564977838940.png)

## redis

### redis数据结构

| 结构类型 | 结构存储的值                     | 结构的读写能力 |
| -------- | -------------------------------- | -------------- |
| String   | 可以是字符串，整数或者浮点数     |                |
| list     | 一个链表                         |                |
| set      | 包含字符串的无序容器，每个值唯一 |                |
| hash     | 无序散列表                       |                |
| zset     | 有序容器                         |                |

### 持久化选项

redis提供了两种持久化方法来讲数据存储到硬盘。主要是避免内存数据丢失。

1. 快照，它可以将存在于某一时刻的所有数据都写入硬盘里面
2. 只追加文件，它会在执行写命令时，将被执行的写命令复制到硬盘里面

### 使用redis构建应用

1. 使用redis来记录日志
2. 