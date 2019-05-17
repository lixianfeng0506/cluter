# mysql 集群技术

# 前言：
当你的业务到达一定的量，肯定需要一定数据的数据库来负载均衡数据库请求。在业务不达的情况下我们会使用主从赋值方法来实现数据库同步。一主多从同步，或双主双从等，但是虽然进行了读写分离，但是对于读的方法限制还是比较大的，所以解决数据同步问题就是数据库集群的意义。我们在这里使用mysql官方提供的myslq-cluster实现集群。

# mysql cluster中的几个概念解释：
为了简单，我后面简称mysql-cluster为mc。
1、mc已经包含了mysql，我下载的最新的mc7.5，官方说明包含的是mysql版本是5.7。所以不需要使用别的msyql的安装包安装数据库。同时注意mysql5.7的版本在安装的命令和配置上面和之前的版本有很大的不同，所以网上有很多mc7.5之前的版本，所包含的mysql版本不同，所以安装方法不同。
2、管理节点，mc管理节点负责管理、配置、监控整个集群。
3、数据节点，使用内存存放数据，保存进数据节点的数据都会自动复制并存储到其他数据节点。
4、mysql节点，也叫数据库节点，和我们平时使用的mysql相同，作为数据库使用。被数据节点访问。

因为虚拟机占用内存较大，只使用了3台服务器，在实际情况中最好将数据节点和mysql节点分开。在实际中负载均衡服务还需要做备份，因为万一负载均衡服务器宕机将会导致所有数据节点都无法访问，所以需要对负载均衡服务器备份，有条件的话，分开管理节点和负载均衡器。实验只实现整个数据库集群，负载均衡请参考之前的博客配置即可。
129的负载均衡可以参考 HAProxy实现mysql负载均衡 也可以使用的别的负载均衡方案。

# 下载mysql cluster：
我下载的版本是mysql-cluster-gpl-7.5.14-linux-glibc2.12-x86_64.tar.gz
https://cdn.mysql.com//Downloads/MySQL-Cluster-7.5/mysql-cluster-gpl-7.5.14-linux-glibc2.12-x86_64.tar.gz
注意看清是64位版本的，别下载错了

    
# 安装mysql cluster之前
安装之前，如果之前安装过mysql，那么需要删除相应的各种mysql文件，删除之前请停止mysql服务。并且不要忘记删除my.cnf这些配置文件。确保删除干净。不然可能会和后面的安装有冲突。如果是实验，关闭防火墙，实际中，防火墙打开对应端口，｛注意实际中需要使用的端口不只有3306端口，还有同步需要使用的1186端口！！！｝。保证服务器之前能互相访问，能ping通。保证固定的ip地址。保证没有别的程序占用需要使用的端口。如3306等。这些都确认完毕后再进行安装。需要linux基础的命令熟练，需要熟练安装mysql基本版本等操作，因为后序的一些操作我会简单描述，不做过多的说明了。


# 安装配置管理节点
将下载后的包上传至服务器/usr/local下
解压
tar xvf mysql-cluster-gpl-7.5.14-linux-glibc2.12-x86_64.tar.gz
将需要的文件取出

mv mysql-cluster-gpl-7.5.14-linux-glibc2.12-x86_64 /usr/local/mysql

cp /usr/local/bin/ndb_mgm* /usr/local/bin

cd /usr/local/bin

chmod +x ndb_mgm*

# 新建配置文件并且初始化管理节点

 mkdir /var/lib/mysql-cluster
 
mkdir /usr/local/mysql

 vi /var/lib/mysql-cluster/config.ini
 
[ndbd default]
NoOfReplicas=2
DataMemory=512M
IndexMemory=18M

[ndb_mgmd]
HostName=172.30.156.220

DataDir=/var/lib/mysql-cluster

[ndbd]

HostName=172.30.156.221

DataDir=/var/lib/mysql-cluster

[ndbd]

HostName=172.30.156.222

DataDir=/var/lib/mysql-cluster

退出并保存

# 使用配置文件初始化管理节点

/usr/local/bin/ndb_mgmd -f /var/lib/mysql-cluster/config.ini --initial
 
然后就能使用ndbd进去管理了(如果ndbd命令不行，就使用在/usr/local/bin目录下使用ndb_mgm命令)
 
ndbd
 
ndb_mgm>show（使用show命令查看管理情况，当数据节点配置完毕之后，我们再用这个命令查看和管理）
 
安装配置数据和mysql节点
 
以下的所有操作需要在所有的集群节点都要进行相同的操作
 
groupadd mysql
 
useradd -g mysql -s /home/mysql

新建文件夹并赋予权限

mkdir /var/lib/mysql-cluster

chown root:mysql /var/lib/mysql-cluster

将下载后的包上传至服务器/usr/local下

解压tar xvf mysql-cluster-gpl-7.5.4-linux-glibc2.5-x86_64.tar.gz

mv mysql-cluster-gpl-7.5.4-linux-glibc2.5-x86_64.tar.gz mysql

初始化数据库(这里要注意，如果你安装的版本和我的不同，数据库初始化的命令使不同的，很多之前的版本会使用：scripts/mysql_install_db --user=mysql来初始化，这个已经被mysql在新的版本中废弃了，所以需要使用下面的命令安装，如果你需要安装别的版本请参考mysql官网的对应版本的安装命令。)

mkdir /usr/local/mysql/data

chown root:mysql /usr/local/mysql/data 数据库文件存放文件

cd /usr/local/mysql

如果下方这个命令无法使用，那么就进入bin目录下使用./mysqld --initialize进行初始化，总之正常安装mysql如何初始化就如何进行安装就可以了，这里还可以设置安装数据库的data目录等参数这里就不多解释了，网上安装mysql5.7的教程很多。

./usr/local/mysql/bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data

chown -R root /usr/local/mysql

chown -R mysql /usr/local/mysql/data

chgrp -R mysql /usr/local/mysql

cp /usr/local/mysql/support-files/mysql.server /etc/rc.d/init.d/

chmod +x /etc/rc.d/init.d/mysql.server

chkconfig --add mysql.server

# 配置数据节点

vi /etc/my.cnf

`[mysqld]
ndbcluster
ndb-connectstring=172.30.156.220

[mysql_cluster]
ndb-connectstring=172.30.156.220`

其中的IP为管理节点的IP

启动集群节点上面的服务启动mysql（成功会有success） /etc/init.d/mysql.server start

启动mysql成功之后请自己登录进mysql内然后进行密码修改等操作，就和正常安装完成mysql的操作一样。需要注意的是，集群数据库的密码需要相同哦！

启动ndbd# /etc/init.d/ndbd --initial如果上述不行使用绝对路径的这个：# /usr/local/mysql/bin/ndbd --initial如果出现下述现象就成功了


2017-03-06 14:04:07 [ndbd] INFO     -- Angel connected to '192.168.75.129:1186' 
2017-03-06 14:04:07 [ndbd] INFO     -- Angel allocated nodeid: 2


最后当所有的节点配置完成，回到管理节点，使用上述说过的show查看，如下的类似显示，证明已经连接完成


ndb_mgm> show 
Cluster Configuration 
--------------------- 
[ndbd(NDB)]    2 node(s) 
id=2 (not connected, accepting connect from 192.168.75.128) 
id=3    @192.168.75.130  (mysql-5.1.63 ndb-7.1.23, starting, Nodegroup: 0)

[ndb_mgmd(MGM)]    1 node(s) 
id=1    @192.168.75.129  (mysql-5.7.16 ndb-7.5.4)

[mysqld(API)]    2 node(s) 
id=4 (not connected, accepting connect from any host) 
id=5 (not connected, accepting connect from any host)

 修改mysql密码统一，修改mysql的访问权限，使外部ip能远程访问mysql
 
 唯一需要注意的是，创建表的时候必须选择表的引擎为NDBCLUSTER，否则表不会进行同步
 
 如果使用sql创建表，命令为：CREATE TABLE student (age INT) ENGINE=NDBCLUSTER






 
 
