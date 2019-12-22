# centos操作系统

## 基本操作

### centos7防火墙关闭

firewall-cmd --state

systemctl stop firewalld.service

systemctl disable firewalld.service

![image-20191221223148407](assets/image-20191221223148407.png)

### 服务程序维护

关闭防火墙(centos7 以前)

service iptables stop 关闭防火墙

chkconfig iptables off 更新到配置

Centos7关闭防火墙：

systemctl stop firewalld.service

systemctl disable firewalld.service

 

修改日期： date -s 18/10/2017

修改时间： date -s 10:18:00 

### 进程操作

netstat –apn 其中最后一栏是PID/Program name

ps -ef 查看进程信息

### 关机管理

Shutdown -r now 立即关机，root用户权限

Linux共有七种运行等级

- run level 0：关机
- run level 3：纯文本模式
- run level 5：含有图形接口模式
- run level 6：重新启动

 关机命令：

​    shutdown -h now

​    poweroff

   init 0

### 文件操作

打包压缩命令：压缩目录

zip格式

   zip -r ./filename.zip ./filename/*

   unzip *.zip

tar命令  .tar.gz 

​    tar --exclude=*/logs/* --exclude=*/work/* --exclude=*.war* --exclude=*.tar.gz --exclude=*.zip* --exclude=*/hqtrans.log --exclude=*/.svn/* --exclude=*.tgz --exclude=*.tar -zcvf BFall.tar.gz ./*

​    tar -zvxf *.tar.gz

分目录统计文件大小：du -s    root用户权限

把当前一个文件copy到远程另外一台主机上：

scp /home/daisy/full.tar.gz root@172.19.2.75:/home/root

把文件从远程主机copy到当前系统：

scp [root@](http://linmaogan.blog.163.com/blog/static/382639372009101062147913/root@172.19.2.75/home/root)/full.tar.gz 172.19.2.75:/home/root/full.tar.gz home/daisy/full.tar.gz

### 创建用户并指定目录

useradd -d /users/g -m g

passwd g

## 系统信息查看

Linux查看系统编码：locale

修改系统编码：LANG="charset"

查看操作系统位数：

getconf LONG_BIT

uname -m

Arch

查看操作系统版本：cat /proc/version

查看系统CPU情况：cat /proc/cpuinfo

查看cpu位数：getconf LONG_BIT

Linux查看CPU信息：

* 查看物理CPU的个数：cat /proc/cpuinfo |grep "physical id"|sort |uniq|wc –l
* 查看逻辑CPU的个数：cat /proc/cpuinfo |grep "processor"|wc –l
* 查看CPU是几核：cat /proc/cpuinfo |grep "cores"|uniq
* 查看CPU的主频：cat /proc/cpuinfo |grep MHz|uniq 
* uname -a
* 查看当前操作系统内核信息：cat /etc/issue | grep Linux
* cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
* cat /proc/cpuinfo | grep physical | uniq -c
* getconf LONG_BIT
* 如何获得CPU的详细信息：cat /proc/cpuinfo
* cat /proc/cpuinfo | grep flags | grep ' lm ' | wc –l
* 逻辑CPU个数：cat /proc/cpuinfo | grep "processor" | wc -l
* 物理CPU个数：cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
* 每个物理CPU中Core的个数：cat /proc/cpuinfo | grep "cpu cores" | wc -l
* 查看CPU信息命令：cat /proc/cpuinfo
* 查看内存信息命令：cat /proc/meminfo

 查看硬盘信息命令：fdisk -l

## 常见问题及解决

文件格式为dos系统，命令文件出现如图

![img](assets/clipboard.png)

用vim打开执行: set ff 可以看到文件格式为dos，执行：set ff=unix 修改，保存退出

![img](assets/clipboard.png)

