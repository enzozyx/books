# maven工程配置

[TOC]



## 安装maven插件



## 创建maven工程



## 配置源代码管理

### subversion



### git



## 配置构件触发器





## 配置构建前置触发



```shell
# 因为nexus 并没有做snapshot 或是版本升级的管理，所以每次服务器更新，在构建下载依赖前，需要手动将本地项目依赖删除，实时从nexus私服请求最新的依赖包，如果使用snapshot或是release进行了版本升级，该操作可省(如果不存在私服jar依赖，该步骤也可以省略)
rm -rf /opt/repository/com/huajie/bim
rm -rf /root/.jenkins/workspace/fj-server-news/target/
```



## 构建

- pom root:  pom文件相对工作空间的路径
- goal： 一般我们构建目标为package ，如果存在同步的nexus 版本管理，可以考虑在此使用deploy（不推荐，个人认为jenkins更新迭代远比版本的更替更细致，jenkins偏向开发的发展路线轨迹记录，实际软件版本的更替线路不应和jenkins线路进行概念的混合，不然会造成版本堆积，增加版本管理复杂度，当然如果需要细致化的版本管理，就另当别论啦）



## 配置构建后置触发

```shell
# 拷贝编译包 至工作目录
mkdir -p /opt/servers/bak
cp /root/.jenkins/workspace/fj-server-news/target/server-news.jar /opt/servers/bim-server-news/
# 更新服务 BUILD_ID设置是为了防止jenkins将附属进程关闭，导致启动的服务也随之关闭
# 复杂的服务启动，应该编写模块化的shell脚本，如start stop bak update 等，单独编写，然后根据需要调用，可以避免代码复杂性以及管理的混乱
BUILD_ID=NOT_KILL_ME_BIM_SERVER_NEWS_8095
/opt/servers/bim-server-news/updateServer.sh
```

