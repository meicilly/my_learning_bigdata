### Zookeeper集群模式的搭建
######[Zookeeper的官网](https://zookeeper.apache.org/)
###### 在zookeeper集群中请求的端口号是2181，集群之间数据同步的端口号是3888，集群选举端口号只有leader会监听2888.flower会随机打开一个端口向2888请求数据。只有leader才能写，写完同步到flower。
```shell
#1.对zookeeper的安装包进行解压
tar -zxvf zookeeper-3.4.5.tar.gz
#2。修改解压后文件夹的名字
mv zookeeper-3.4.5 zookeeper
#3.进入conf目录，基于模板配置文件生成配置文件
cd zookeeper/conf
cp zoo_sample.cfg zoo.cfg
#4.在zoo.cfg文件中的配置
tickTime=2000
#服务器与服务器之间心跳检测的间隔时间,单位为毫秒
initLimit=10
#集群中leader服务器与follower服务器初始连接心跳次数,即多少 个 2000 毫秒
syncLimit=5
#leader与follower之间连接完成后,后期检测发送和应答心跳次数
dataDir=/opt/software/zookeeper/data
dataLogDir=/opt/software/zookeeper/log
clientPort=2181
#客户端连接zookeeper服务器的端口,Zookeeper会监听这个端口,接收客户端的访问请求
maxClientCnxns=128
#单个客户端 IP 可以和 zookeeper 保持的连接数
server.1=192.168.233.16:2888:3888
server.2=192.168.233.17:2888:3888
server.3=192.168.233.18:2888:3888
#5。创建data目录和log的目录
mkdir data
mkdir log
#6.在data下创建myid文件用来进行选举
each "1" > myid
# 7.把zookeeper包分别拷贝到另外的两台机器上 并修改myid文件
# 8.启动zookeeper
zkServer.sh start 
```