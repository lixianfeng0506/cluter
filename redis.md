# cluter

redis 3.0 + 正式开始支持集群。

# redis 集群介绍

1. 集群是一个提供在多个Redis间节点间共享数据的程序集。

2. redis不支持处理多个keys的命令，因为这需要在不同节点间移动数据，从而不像redis那样的性能，在搞负载的情况下可能会导致不可预料的错误

3. redis集群通过分区来提供一定程度可用性，在实际环境中档某个节点宕机或不可达的情况下继续处理命令，redis集群的优势。

自动分割数据到不同的节点上。

整个集群的部分节点失败或者不可达情况下能够继续处理命令。

# redis 集群的数据分片

redis 集群没有使用一致的hash，而是引入了哈希槽的概念。

Redis 集群有16384个哈希槽,每个key通过CRC16校验后对16384取模来决定放置哪个槽.集群的每个节点负责一部分hash槽,举个例子,比如当前集群有3个节点,那么:

节点 A 包含 0 到 5500号哈希槽.

节点 B 包含5501 到 11000 号哈希槽.

节点 C 包含11001 到 16384号哈希槽.

# Redis 集群的主从复制模型

为了使几点失败或不可用的情况，集群使用了主从复制模型，每个节点都会有n-1个复制品。当节点及从节点都失败，集群不可用。

# redis 一致性

不能保证强一致性，意味这在实际集群中有可能会丢失数据。

第一个原因，集群是用了异步复制，写操作过程。客户端向节点b写入一条命令，节点b向客户端复制命令状态，主节点向写操作复制给其他从节点。

# 搭建并使用Redis集群

1  主从复制 模式搭建集群。

  在从节点 配置文件中配置主节点地址及ip
  slaveof 172.30.156.220 7010     
  
  bind 0.0.0.0 
  
  特点主从复制，   只能在主节点写数据， 在主节点和从节点读取数据
   
  
  

2 哨兵模式

  哨兵的作用是监控 redis系统的运行状况，他的功能如下
  
  监控主从数据库是否正常运行 
  
  master出现故障时，自动将slave转化为master
  
  多哨兵配置的时候，哨兵之间也会自动监控
  
  多个哨兵可以监控同一个redis
  
  哨兵模式与主从分离分开  redis配置redis
  
  sentinel  
  
  优点 主节点挂掉后自动选举主节点。
  
  
  3 redis-cluster搭建
  
 尽管可以使用哨兵主从集群实现可用性保证，但是这种实现方式每个节点的数据都是全量复制，数据存放量存在着局限性，受限于内存最小的节点，因此考虑采用数据分片的方式，来实现存储，这个就是redis-cluster。
 
` port 7005
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
daemonize yes
bind 0.0.0.0
`
实现集群 命令 ./redis-cli --cluster create host1:port1  host2:port2 host3:port3  host4:port4 host5:port5 host6:port6  --cluster-replicas 1
./redis-cli --cluster check host1:port1 


./redis-cli --cluster add-node  new_host:new_port existing_host:existing_port --cluster-master-id <arg>
  
./redis-cli --cluster add-node  new_host:new_port existing_host:existing_port --cluster-slave
  
  


# Redis 集群规范
1. Redis 集群的目标  

Redis 集群是 Redis 的一个分布式实现，主要是为了实现以下这些目标（按在设计中的重要性排序）：
*在1000个节点的时候仍能表现得很好并且可扩展性（scalability）是线性的。
*没有合并操作，这样在 Redis 的数据模型中最典型的大数据值中也能有很好的表现。
*写入安全（Write safety）：那些与大多数节点相连的客户端所做的写入操作，系统尝试全部都保存下来。不过公认的，还是会有小部分（small windows?）写入会丢失
*可用性（Availability）：在绝大多数的主节点（master node）是可达的，并且对于每一个不可达的主节点都至少有一个它的从节点（slave）可达的情况下，Redis 集群仍能进行分区（partitions）操作


