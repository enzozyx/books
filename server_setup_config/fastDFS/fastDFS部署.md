## fastDFS 部署

[TOC]



### 准备

### 环境

Center OS 7 64位

### 配置hosts

- 配置linux hosts

> vi /etc/hosts
>
> // 增加
>
> 192.168.3.10 enzo.server.com

- windows配置

> 修改C:\Windows\System32\drivers\etc\hosts
>
> // 增加
>
> 192.168.3.10 enzo.server.com

### 配置基础依赖libfastcommon

- 下载

```
sudo mkdir -p /opt/software_packages
cd /opt/software_packages/
wget https://github.com/happyfish100/libfastcommon/archive/V1.0.7.tar.gz
```

- 解压编译

```
//依赖安装
yum install cmake perl
cd /opt/software_packages/
//监理软连接 (libfastdfs 安装到/usr/lib64, fastDFS 的lib目录安装到了/usr/local/lib)
# ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
# ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so
# ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so
# ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so
```



### 下载fastDFS并安装

```shell
# 下载fastDFS
cd /opt/software_packages/
wget https://github.com/happyfish100/fastdfs/archive/V5.05.tar.gz
# 安装
tar -zxvf V5.05.tar.gz
cd V5.05.tar.gz
./make.sh
./make.sh install

```



### fastDFS 脚本与工具环境变量问题配置

```shell
# 安装后的脚本与配置文件路径
## 服务 tracker 和storage
cd /etc/init.d/ 
ll | grep fdfs_
-rwxr-xr-x. 1 root root  1186 9月   6 20:36 fdfs_storaged
-rwxr-xr-x. 1 root root  1186 9月   6 20:36 fdfs_trackerd

## 配置文件路径
cd /etc/fdfs/ 
ll
-rw-r--r--. 1 root root 1461 9月   6 20:36 client.conf.sample
-rw-r--r--. 1 root root 7829 9月   6 20:36 storage.conf.sample
-rw-r--r--. 1 root root 7102 9月   6 20:36 tracker.conf.sample
## 命令行工具
cd /usr/bin
ll | grep fdfs
-rwxr-xr-x. 1 root root    321912 9月   6 20:36 fdfs_appender_test
-rwxr-xr-x. 1 root root    321688 9月   6 20:36 fdfs_appender_test1
-rwxr-xr-x. 1 root root    308560 9月   6 20:36 fdfs_append_file
-rwxr-xr-x. 1 root root    307992 9月   6 20:36 fdfs_crc32
-rwxr-xr-x. 1 root root    308592 9月   6 20:36 fdfs_delete_file
-rwxr-xr-x. 1 root root    309352 9月   6 20:36 fdfs_download_file
-rwxr-xr-x. 1 root root    308936 9月   6 20:36 fdfs_file_info
-rwxr-xr-x. 1 root root    322760 9月   6 20:36 fdfs_monitor
-rwxr-xr-x. 1 root root   1125656 9月   6 20:36 fdfs_storaged
-rwxr-xr-x. 1 root root    331848 9月   6 20:36 fdfs_test
-rwxr-xr-x. 1 root root    326968 9月   6 20:36 fdfs_test1
-rwxr-xr-x. 1 root root    465432 9月   6 20:36 fdfs_trackerd
-rwxr-xr-x. 1 root root    309544 9月   6 20:36 fdfs_upload_appender
-rwxr-xr-x. 1 root root    310576 9月   6 20:36 fdfs_upload_file
## 特别说明，fastDFS 工具bin内部配置执行工具的路径是 /usr/local/bin 但我们刚刚也看到了，实际的执行脚本文件都在usr/bin 下，所以我们要进行一下处理，两种方法：1.修改fdfs_tracker和fdfs_storage内部的路径 2.建立软连接
# 方式1
vi /etc/init.d/fdfs_trackerd 
## 批量替换指令
:%s+/usr/local/bin+/usr/bin

vi /etc/init.d/fdfs_storaged
## 批量替换指令
:%s+/usr/local/bin+/usr/bin

# 方式2
/usr/bin/fdfs_trackerd /usr/local/bin
/usr/bin/fdfs_storaged /usr/local/bin
/usr/bin/stop.sh /usr/local/bin
/usr/bin/restart.sh /usr/local/bin
```



### 配置tracker

```shell
# 配置tracker
## 1. 拷贝样例配置文件
cd /etc/fdfs
cp tracker.conf.sample tracker.conf
## 2. 修改配置
### 修改tracker 日志与数据库数据信息存放目录的更目录，启动后会自动创建data与log目录，
# the base path to store data and log files
base_path=/opt/datas/fastdfs
data目录：
storage_groups.dat：存储分组信息
storage_servers.dat：存储服务器列表
log目录：
trackerd.log： tracker server 日志文件
### 修改HTTP服务端口 
# HTTP port on this tracker server
http.server_port=8101
## 3. 创建base_path目录
mkdir -p /opt/datas/fastdfs
## 4.打开防火墙（iptables方式， center 7 使用firewall-cmd）
vi /etc/sysconfig/iptables
### 新增规则(22122 可以在刚刚的配置文件 属性：tracker server port: port)
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22122 -j ACCEPT
### 重启防火墙
service iptables restart
## 4.打开防火墙（firewall-cmd方式）
firewall-cmd --permanent --zone=public --add-port=22122/tcp  //永久
firewall-cmd --zone=public --add-port=22122/tcp  //临时
### 重启生效
firewall-cmd --reload

## 5.启动tracker
方式1：
/etc/init.d/fdfs_trackerd start
方式2：
service fdfs_trackerd start

## 6.检查是否已启动
[root@vm-server-a fdfs]# ps -ef | grep tracker
root      3448     1  0 10:32 ?        00:00:00 /usr/local/bin/fdfs_trackerd /etc/fdfs/tracker.conf
root      3456  1168  0 10:32 pts/0    00:00:00 grep --color=auto tracker
[root@vm-server-a fdfs]# netstat -ntlp | grep 22122
tcp        0      0 0.0.0.0:22122           0.0.0.0:*               LISTEN      3448/fdfs_trackerd  

## 7. 关闭tracker服务
/etc/init.d/fdfs_trackerd stop
或 service fdfs_trackerd stop
Stopping fdfs_trackerd (via systemctl):                    [  确定  ]
netstat -ntlp | grep 22122
ps -ef | grep tracker
root      3517  1168  0 10:33 pts/0    00:00:00 grep --color=auto tracker

## 查看一下tracker的目录
[root@vm-server-a fdfs]# cd /opt/datas/fastdfs/
[root@vm-server-a fastdfs]# ll
总用量 0
drwxr-xr-x. 2 root root 130 9月   7 10:33 data
drwxr-xr-x. 2 root root  26 9月   7 10:32 logs
[root@vm-server-a fastdfs]# cd data/
[root@vm-server-a data]# ll
总用量 8
-rw-r--r--. 1 root root  0 9月   7 10:32 storage_changelog.dat
-rw-r--r--. 1 root root 42 9月   7 10:33 storage_groups_new.dat
-rw-r--r--. 1 root root 44 9月   7 10:33 storage_servers_new.dat
-rw-r--r--. 1 root root  0 9月   7 10:33 storage_sync_timestamp.dat
[root@vm-server-a data]# cd ..
[root@vm-server-a fastdfs]# cd logs/
[root@vm-server-a logs]# ll
总用量 4
-rw-r--r--. 1 root root 1328 9月   7 10:33 trackerd.log

## 8. 设置tracker 开机启动
chkconfig fdfs_trackerd on
或 下面这中还想没生效
vi /etc/rc.d/rc.local
新增：
/etc/init.d/fdfs_trackerd start

```



### 配置fastDFS存储

```shell
# 修改配置 
cd /etc/fdfs
cp storage.conf.sample storage.conf
vi storage.conf

## 修改配置

# the storage server port
port=23000

# the base path to store data and log files
base_path=/opt/datas/fastdfs/storage

# path(disk or mount point) count, default value is 1
store_path_count=1

# store_path#, based 0, if store_path0 not exists, it's value is base_path
# the paths must be exist
store_path0=/opt/datas/fastdfs/file
#store_path1=/home/yuqing/fastdfs2

# subdir_count  * subdir_count directories will be auto created under each
# store_path (disk), value can be 1 to 256, default value is 256
subdir_count_per_path=256

# tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
tracker_server=enzo.server.com:22122

# Hour from 0 to 23, Minute from 0 to 59
sync_start_time=00:00

# storage sync end time of a day, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
sync_end_time=23:59

# the port of the web server on this storage server
http.server_port=8102


## 创建工作目录
mkdir -p /opt/datas/fastdfs/storage
mkdir -p /opt/datas/fastdfs/file

## 打开storage 服务端口
firewall-cmd --permanent --zone=public --add-port=23000/tcp    //永久
firewall-cmd --zone=public --add-port=23000/tcp    //临时

## 启动服务
[root@vm-server-a fdfs]# service fdfs_storaged start
Starting fdfs_storaged (via systemctl):                    [  确定  ]
[root@vm-server-a fdfs]# ps -ef | grep storaged
root      1178     1 89 11:12 ?        00:00:07 /usr/local/bin/fdfs_storaged /etc/fdfs/storage.conf
root      1180  1106  0 11:12 pts/0    00:00:00 grep --color=auto storaged
[root@vm-server-a fdfs]# netstat -ntlp | grep 23000
tcp        0      0 0.0.0.0:23000           0.0.0.0:*               LISTEN      1182/fdfs_storaged 

## 查看tracker与storage通信
[root@vm-server-a fdfs]# netstat -ntlp | grep fdfs
tcp        0      0 0.0.0.0:23000           0.0.0.0:*               LISTEN      1182/fdfs_storaged  
tcp        0      0 0.0.0.0:22122           0.0.0.0:*               LISTEN      991/fdfs_trackerd   
### tracker与storage 通信连接
	Storage 1:
		id = 192.168.3.10
		ip_addr = 192.168.3.10 (enzo.server.com)  ACTIVE

## 设置storage 开机自启动
chkconfig fdfs_storaged on

## 查看storage 目录
### 服务目录 data和log
[root@vm-server-a fdfs]# cd /opt/datas/fastdfs/storage 
[root@vm-server-a storage]# ll 
总用量 0
drwxr-xr-x. 3 root root 90 9月   7 11:12 data
drwxr-xr-x. 2 root root 26 9月   7 11:12 logs
[root@vm-server-a storage]# cd data/
[root@vm-server-a data]# ll
总用量 8
-rw-r--r--. 1 root root    4 9月   7 11:12 fdfs_storaged.pid
-rw-r--r--. 1 root root 1011 9月   7 11:12 storage_stat.dat
drwxr-xr-x. 2 root root   44 9月   7 11:12 sync
[root@vm-server-a data]# cd ../logs/ & ll
[1] 1225
总用量 8
-rw-r--r--. 1 root root    4 9月   7 11:12 fdfs_storaged.pid
-rw-r--r--. 1 root root 1011 9月   7 11:12 storage_stat.dat
drwxr-xr-x. 2 root root   44 9月   7 11:12 sync

### 文件存放目录 256 * 256
[root@vm-server-a data]# ls
00  06  0C  12  18  1E  24  2A  30  36  3C  42  48  4E  54  5A  60  66  6C  72  78  7E  84  8A  90  96  9C  A2  A8  AE  B4  BA  C0  C6  CC  D2  D8  DE  E4  EA  F0  F6  FC
01  07  0D  13  19  1F  25  2B  31  37  3D  43  49  4F  55  5B  61  67  6D  73  79  7F  85  8B  91  97  9D  A3  A9  AF  B5  BB  C1  C7  CD  D3  D9  DF  E5  EB  F1  F7  FD
02  08  0E  14  1A  20  26  2C  32  38  3E  44  4A  50  56  5C  62  68  6E  74  7A  80  86  8C  92  98  9E  A4  AA  B0  B6  BC  C2  C8  CE  D4  DA  E0  E6  EC  F2  F8  FE
03  09  0F  15  1B  21  27  2D  33  39  3F  45  4B  51  57  5D  63  69  6F  75  7B  81  87  8D  93  99  9F  A5  AB  B1  B7  BD  C3  C9  CF  D5  DB  E1  E7  ED  F3  F9  FF
04  0A  10  16  1C  22  28  2E  34  3A  40  46  4C  52  58  5E  64  6A  70  76  7C  82  88  8E  94  9A  A0  A6  AC  B2  B8  BE  C4  CA  D0  D6  DC  E2  E8  EE  F4  FA
05  0B  11  17  1D  23  29  2F  35  3B  41  47  4D  53  59  5F  65  6B  71  77  7D  83  89  8F  95  9B  A1  A7  AD  B3  B9  BF  C5  CB  D1  D7  DD  E3  E9  EF  F5  FB
[root@vm-server-a data]# cd 00/
Display all 256 possibilities? (y or n)
00/ 06/ 0C/ 12/ 18/ 1E/ 24/ 2A/ 30/ 36/ 3C/ 42/ 48/ 4E/ 54/ 5A/ 60/ 66/ 6C/ 72/ 78/ 7E/ 84/ 8A/ 90/ 96/ 9C/ A2/ A8/ AE/ B4/ BA/ C0/ C6/ CC/ D2/ D8/ DE/ E4/ EA/ F0/ F6/ FC/ 
01/ 07/ 0D/ 13/ 19/ 1F/ 25/ 2B/ 31/ 37/ 3D/ 43/ 49/ 4F/ 55/ 5B/ 61/ 67/ 6D/ 73/ 79/ 7F/ 85/ 8B/ 91/ 97/ 9D/ A3/ A9/ AF/ B5/ BB/ C1/ C7/ CD/ D3/ D9/ DF/ E5/ EB/ F1/ F7/ FD/ 
02/ 08/ 0E/ 14/ 1A/ 20/ 26/ 2C/ 32/ 38/ 3E/ 44/ 4A/ 50/ 56/ 5C/ 62/ 68/ 6E/ 74/ 7A/ 80/ 86/ 8C/ 92/ 98/ 9E/ A4/ AA/ B0/ B6/ BC/ C2/ C8/ CE/ D4/ DA/ E0/ E6/ EC/ F2/ F8/ FE/ 
03/ 09/ 0F/ 15/ 1B/ 21/ 27/ 2D/ 33/ 39/ 3F/ 45/ 4B/ 51/ 57/ 5D/ 63/ 69/ 6F/ 75/ 7B/ 81/ 87/ 8D/ 93/ 99/ 9F/ A5/ AB/ B1/ B7/ BD/ C3/ C9/ CF/ D5/ DB/ E1/ E7/ ED/ F3/ F9/ FF/ 
04/ 0A/ 10/ 16/ 1C/ 22/ 28/ 2E/ 34/ 3A/ 40/ 46/ 4C/ 52/ 58/ 5E/ 64/ 6A/ 70/ 76/ 7C/ 82/ 88/ 8E/ 94/ 9A/ A0/ A6/ AC/ B2/ B8/ BE/ C4/ CA/ D0/ D6/ DC/ E2/ E8/ EE/ F4/ FA/ 
05/ 0B/ 11/ 17/ 1D/ 23/ 29/ 2F/ 35/ 3B/ 41/ 47/ 4D/ 53/ 59/ 5F/ 65/ 6B/ 71/ 77/ 7D/ 83/ 89/ 8F/ 95/ 9B/ A1/ A7/ AD/ B3/ B9/ BF/ C5/ CB/ D1/ D7/ DD/ E3/ E9/ EF/ F5/ FB/ 


# 关闭服务
service fdfs_storaged stop
```



### 测试上传

```shell
# 修改client 配置
cd /etc/fdfs
cp client.conf.sample client.conf
vi client.conf

## 修改配置
# the base path to store log files
base_path=/opt/datas/fastdfs/logs

# tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
tracker_server=enzo.server.com:22122

#HTTP settings
http.tracker_server_port=8101

## 创建base_path
mkdir -p /opt/datas/fastdfs/logs

## 测试
cd /opt/sofeware_packages
/usr/bin/fdfs_upload_file /etc/fdfs/client.conf V1.0.7.tar.gz 
group1/M00/00/00/wKgDCluR8POAdzyUAAEdvO0ZHFk.tar.gz
```



### 安装nginx

```shell
# nginx 1.提供文件服务
# 配合 fastDFS 实现文件访问的重定向

## 安装前置依赖
yum install gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel
## 下载nginx 
cd /opt/software_packages
wget -c https://nginx.org/download/nginx-1.12.1.tar.gz
tar -zxvf nginx-1.12.1.tar.gz
cd nginx-1.12.1
./configure
make
make install
## nginx bin目录
/usr/local/nginx/sbin/nginx
/usr/local/nginx/sbin/nginx 启动
/usr/local/nginx/sbin/nginx -V
/usr/local/nginx/sbin/nginx -t
/usr/local/nginx/sbin/nginx -s reload/stop
# 设置开机启动
vi /etc/rc/local
# 开放端口 80
firewall-cmd --permanent --zone=public --add-port=80/tcp
firewall-cmd --reload
## storage服务器配置文件请求服务


```





### 配置fastDFS与nginx

```shell
#下载 fastFDFS nginx module
cd /opt/software_packages/
wget https://github.com/happyfish100/fastdfs-nginx-module/archive/5e5f3566bbfa57418b5506aaefbe107a42c9fcb1.zip
# 解压
yum install unzip
unzip 5e5f3566bbfa57418b5506aaefbe107a42c9fcb1.zip
# 改名
mv fastdfs-nginx-module-5e5f3566bbfa57418b5506aaefbe107a42c9fcb1/ fastdfs-nginx-module-master
# 关闭nginx服务
/usr/local/nginx/sbin/nginx -s stop
# 进入nginx 解压包
cd /opt/software_packages/nginx-1.12.1
## 预编译是加入模块配置
./configure --add-module=../fastdfs-nginx-module-master/src
## 编译安装
make && make install
## 查看安装信息
[root@vm-server-a nginx-1.12.1]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.12.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC) 
configure arguments: --add-module=../fastdfs-nginx-module/src/

# 配置etc
## 将fastdfs-nginx-module 源码中配置文件拷贝到/etc/fdfs钟
cd /opt/software_packages/fastdfs-nginx-module/src
cp mod_fastdfs.conf /etc/fdfs/
# 修改模块配置信息
# connect timeout in seconds
# default value is 30s
connect_timeout=0

# FastDFS tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
# valid only when load_fdfs_parameters_from_tracker is true
tracker_server=enzo.server.com:22122

# the port of the local storage server
# the default value is 23000
storage_server_port=23000

# if the url / uri including the group name
# set to false when uri like /M00/00/00/xxx
# set to true when uri like ${group_name}/M00/00/00/xxx, such as group1/M00/xxx
# default value is false
url_have_group_name = true

# store_path#, based 0, if store_path0 not exists, it's value is base_path
# the paths must be exist
# must same as storage.conf
store_path0=/opt/datas/fastdfs/file

## 赋值fastDFS 部分配置文件到 /etc/fdfs
cd /opt/software_packages/fastdfs-5.05/conf/
[root@vm-server-a conf]# ll
总用量 84
-rw-rw-r--. 1 root root 23981 11月 22 2014 anti-steal.jpg
-rw-rw-r--. 1 root root  1461 11月 22 2014 client.conf
-rw-rw-r--. 1 root root   858 11月 22 2014 http.conf
-rw-rw-r--. 1 root root 31172 11月 22 2014 mime.types
-rw-rw-r--. 1 root root  7829 11月 22 2014 storage.conf
-rw-rw-r--. 1 root root   105 11月 22 2014 storage_ids.conf
-rw-rw-r--. 1 root root  7102 11月 22 2014 tracker.conf
[root@vm-server-a conf]# cp anti-steal.jpg http.conf mime.types /etc/fdfs/

##配置nginx
    server {
        listen       8102;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        location /group[0-9]/M00 {
            ngx_fastdfs_module;
        }
1. 增加location
location /group[0-9]/M00 {
  ngx_fastdfs_module;
}
2. listen 端口与storage.http_server配置端口一致
## 启动nginx
/usr/local/nginx/sbin/nginx 
ngx_http_fastdfs_set pid=12006
## 开放端口
[root@vm-server-a conf]# firewall-cmd --permanent --zone=public --add-port=8102/tcp
success
[root@vm-server-a conf]# firewall-cmd --reload
success
## 再次访问文件




```



### java客户端



### 权限控制



