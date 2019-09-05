# redis_c
redis集群，采用docker-compose建立，方便快捷
通过docker-compose 建立独立redis集群，三主三从。文件目录：

节点一：slave1 
Dockerfile
#基础镜像
FROM redis:3.0.6

#将自定义conf文件拷入
COPY redis.conf /usr/local/etc/redis/redis.conf

#修复时区
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo 'Asia/Shanghai' >/etc/timezone

#修改文件权限,使之可以通过config rewrite重写
RUN chmod 777 /usr/local/etc/redis/redis.conf

#Redis客户端连接端口
EXPOSE 6061
#集群总线端口:redis客户端连接的端口 + 10000
EXPOSE 16061

#使用自定义conf启动
CMD [ "redis-server", "/usr/local/etc/redis/redis.conf" ]

redis.conf(redis:3.0.6官方tar包的redis.conf，需要修改如下内容)
daemonize no 
pidfile /var/run/redis_6061.pid
port 6061
dbfilename dump_6061.rdb
cluster-enabled yes
cluster-config-file nodes-6061.conf
cluster-node-timeout 15000
剩余节点和Dockerfile 如节点一配置，修改配置中的6061为6062,6063,6064,6065,6066
配置完成后
docker-compose build
docker-compose up -d 
6个节点都已经启动
docker ps 

ps -ef|grep redis

然后将节点加入集群
由于未绑定任何地址，所以显示的是*:6061，但是用127.0.0.1:6061会报错无法连接
ifconfig 查看自己服务器的内网地址
然后运行
docker run --rm -it zvelo/redis-trib create --replicas 1 ip:6061 ip:6062 ip:6063 ip:6064 ip:6065 ip:6066
ip为本机内网地址
redis-cli -h 127.0.0.1 -c -p 6061
连接集群。-c是连接到集群

设定一个值，转存到6003节点了

在6001获取，会自动转到6003，集群搭建完毕。
redis 集群在Django中的连接，和单个redis数据库连接类似，但是要安装几个包
Django                        2.0.7
django-cluster-redis          1.0.5
django-redis                  4.10.0
redis-py-cluster              2.0.0
redis                         3.0.1
以上是我安装的版本，再修改setting.py
CACHES = {
  'default': {
    'BACKEND': 'django_redis.cache.RedisCache',
    'LOCATION': ['redis://127.0.0.1:7001', 'redis://127.0.0.1:7002', 'redis://127.0.0.1:7003',],  # 格式为 redis://IP:PORT/db_index，数>据库编号可为空，默认为0号
    'OPTIONS': {
      'REDIS_CLIENT_CLASS': 'rediscluster.RedisCluster',
      'CONNECTION_POOL_CLASS': 'rediscluster.connection.ClusterConnectionPool',
      'CONNECTION_POOL_KWARGS': {
        'skip_full_coverage_check': True # AWS ElasticCache has disabled CONFIG commands
      }
        }
  }
}
这样就OK了，在视图中使用或者session中使用与单独redis一样。















