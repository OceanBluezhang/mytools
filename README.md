mytools
=======

qq交流群：178844602

ig+docker:一键构造mongodb集群；一键集群数据初始化；参数二次优化
http://t.cn/RwfruoO

fig+docker:一键构造分布式运行环境；参数二次优化
http://t.cn/RwfrDlM


最近摸索的一些新东东放上来2：
fig+docker:一键提供实时消息队列：kafka、实时运算：storm、应用程序发布：java、中文分词全文检索环境：solr
http://t.cn/RwfdAjS

fig+docker:一键提供hadoop伪分布式运行环境
http://t.cn/RwfdiQm




基于debian的images文件小；基于ubuntu的文件大。why
Dockerfile:
    RUN是在building image时会运行的指令, 在Dockerfile中可以写多条RUN指令.
    CMD和ENTRYPOINT则是在运行container 时会运行的指令, 都只能写一条, 如果写了多条, 则最后一条生效.
    CMD和ENTRYPOINT的区别是:
    CMD在运行时会被command覆盖, ENTRYPOINT不会被运行时的command覆盖, 但是也可以指定.
    CMD和ENTRYPOINT一般用于制作具备后台服务的image, 例如apache, database等. 在使用这种image启动container时, 自动启动服务.
    ENV 命令
        用于设置环境变量
        ENV <key> <value>
        设置了后，后续的RUN命令都可以使用
    ADD 命令
        从src复制文件到container的dest路径:
        ADD <src> <dest>
    VOLUME 命令
        VOLUME ["<mountpoint>"]
        如:
        VOLUME ["/data"]
        创建一个挂载点用于共享目录
    WORKDIR 命令
        WORKDIR /path/to/workdir
        配置RUN, CMD, ENTRYPOINT 命令设置当前工作路径
        可以设置多次，如果是相对路径，则相对前一个 WORKDIR 命令
    USER 命令
        比如指定 memcached 的运行用户，可以使用上面的 ENTRYPOINT 来实现:
        ENTRYPOINT ["memcached", "-u", "daemon"]
        更好的方式是：
        ENTRYPOINT ["memcached"]
        USER daemon


应用场景：wordpress,mysql 
    下载安排
    docker pull mysql:latest
    docker pull wordpress:latest
    启动环境
    docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=mysecretpassword -d mysql  //        "IPAddress": "172.17.0.2",
    docker run --name some-wordpress --link some-mysql:mysql -p 8081:80 -d wordpress    //"IPAddress": "172.17.0.3",
    测试
        #不允许远程访问
        boot2docker ssh -L 60001:localhost:3306 //可以在用户环境测试3306端口
        mysql -port 60001 -u mysql -pmysql
    boot2docker ssh -L 8081:localhost:8081
    http://localhost:8081
    自动进入安装界面
    http://127.0.0.1:8081/wp-admin/install.php?step=1
    安装完成之后进入登陆见面
    http://127.0.0.1:8081/wp-login.php    
  
应用场景：DB4Oracle  性能可以  oracel-11g <<<***todo download***>>>

    docker pull wnameless/oracle-xe-11g:latest
    docker run -d -p 49160:22 -p 49161:1521 wnameless/oracle-xe-11g 
    
    hostname: localhost port: 49161 sid: xe username: system password: oracle
    Password for SYS  oracle
    Login by SSH  ssh root@localhost -p 49160 password: admin
    #>su oracle
    oracle>sqlplus system/oracle
    sql>select * from dual
    boot2docker ssh -L 49166:localhost:49161   //49166是本机接收端口，49161是docker转出端口
    OracleSqlDeveloper 建立链接进行测试，成功。
    
    docker pull filemon/oracle_11g:latest
    docker run -d -p 1521:1521 --name oracle  filemon/oracle_11g  
    boot2docker ssh -L 1521:localhost:49159 
    OracleSqlDeveloper
    username-password:system/admin

    
应用场景：app-server tomcat 没有优化，没有集群

    docker pull tutum/tomcat
    启动服务
    docker run -d -p 8082:8080 -p 22 tutum/tomcat
    docker run -d -p 8080:8080 -e TOMCAT_PASS="mypass" tutum/tomcat  //制定口令
    
    看tomcat日志
    docker logs e7de45dad550
    tomcat-server,admin:uo8FsDhjjuUk
    http://127.0.0.1:8082/manager/status
    
    boot2docker ssh -L 80:localhost:8080 
    http://127.0.0.1/manager/status    admin/mypass

    
应用场景：web-server nginx  可制定显示内容和优化配置文件  性能 ok
    docker pull nginx:latest //https://registry.hub.docker.com/_/nginx/
    
    DockerFile:构建自己的nginx-web
        FROM nginx
        COPY static-html-directory /usr/share/nginx/html
        
        docker build -t some-content-nginx 
        docker run --name some-nginx -d some-content-nginx
       
    mkdir some/content
    echo 'hello,word'>some/content/index.html
    #一个本地主机的目录当做数据卷挂载在容器上,[host-dir]:[container-dir]:[rw|ro]
    docker run --net=host --name some-nginx -v /home/docker/some/content:/usr/share/nginx/html:ro -d nginx
    
    
    curl 172.17.0.6/index.html
    
    docker run -rm --volumes-from DATA -v $(pwd):/backup busybox tar cvf /backup/backup.tar /data
    创建一个新容器，挂载数据卷容器，同时挂载一个本地目录，然后把远程数据卷容器的数据卷通过备份命令备份到映射的本地目录里面。
        

    使用本地的配置文件运行容器    
    docker run --name some-nginx -v /some/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx
    docker cp some-nginx:/etc/nginx/nginx.conf some/nginx.conf
    
应用场景：nosql-mongodb 性能不错；没有集群
    
    docker pull mongo:latest
    docker run --name some-mongo -d mongo
    boot2docker ssh -L 27017:172.17.0.7:27017
    
    
应用场景：缓存服务器-memcached/redis ,可配置内存和链接数
    
    docker pull sylvainlasnier/memcached
    docker run --name memcached -d -p 50006:11211 sylvainlasnier/memcached
    docker run --rm -ti -e MAX_MEM=1024,MAX_CONN=10000 sylvainlasnier/memcached
    
    docker run -d -P  -ti -e MAX_MEM=1024,MAX_CONN=10000 sylvainlasnier/memcached
    
    $nc 127.0.0.1 50006
    stats
    
    docker pull redis
    docker run --name some-redis -p 6379 -d redis
    
    #自定义redis配置参数
        FROM redis
        redis.conf /data/
        CMD [ "redis-server", "/data/redis.conf" ]
    docker run --volumes-from datacontainer --name myredis redis
    
应用场景：云计算平台，cloudera   hadoop+hbase+hive+zookeeper+mahout
        https://github.com/cloudera/cdh-twitter-example
        https://github.com/wikimedia/puppet-cdh
        
        https://github.com/stackdio-formulas/cdh5-formula
        
        mahout:
           https://github.com/apache/mahout
           https://github.com/bsspirit/maven_mahout_template
           https://github.com/tdunning/MiA
           
        #初始化环境
        docker pull oddpoet/hbase-cdh5:latest
        docker run -d  -P  oddpoet/hbase-cdh5
        #影射端口访问
        boot2docker ssh -L 65010:127.0.0.1:49155
        http://localhost:65010/ 
        
        #hdfs://master:8020/
        docker run --name cdh -t -d -p 8020:8020 -p 50070:50070 -p 50010:50010 -p 50020:50020 -p 50075:50075 -p 8030:8030 -p 8031:8031 -p 8032:8032 -p 8033:8033 -p 8088:8088 -p 8040:8040 -p 8042:8042 -p 10020:10020 -p 19888:19888 chalimartines/cdh5-pseudo-distributed
        docker run --name cdh -t -d -P chalimartines/cdh5-pseudo-distributed

应用场景：全文检索 solr  未完成

        docker pull makuk66/docker-solr
        docker run -it -p 8983:8983 -t makuk66/docker-solr
        docker run -it -p 8983:8983 -p 7574:7574 makuk66/docker-solr \
            /bin/bash -c "/opt/solr/bin/solr -e cloud; echo hit return to quit; read"
             
            
        # run ZooKeeper, and define a name so we can link to it
        docker run -name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 jplock/zookeeper

        # run the first Solr node, with bootstrap parameters, and pass a link parameter to docker
        # so we can use the ZK_* environment variables in the container to locate the ZooKeeper container
        docker run -link zookeeper:ZK -i -p 8983:8983 -t makuk66/docker-solr \
          /bin/bash -c 'cd /opt/solr/example; java -Dbootstrap_confdir=./solr/collection1/conf -Dcollection.configName=myconf -DzkHost=$ZK_PORT_2181_TCP_ADDR:$ZK_PORT_2181_TCP_PORT -DnumShards=2 -jar start.jar'
        
        # in separate sessions, run two more zookeepers
        docker run -link zookeeper:ZK -i -p 8984:8983 -t makuk66/docker-solr \
        /bin/bash -c 'cd /opt/solr/example; java -DzkHost=$ZK_PORT_2181_TCP_ADDR:$ZK_PORT_2181_TCP_PORT -DnumShards=2 -jar start.jar'
        docker run -link zookeeper:ZK -i -p 8985:8983 -t makuk66/docker-solr \
        /bin/bash -c 'cd /opt/solr/example; java -DzkHost=$ZK_PORT_2181_TCP_ADDR:$ZK_PORT_2181_TCP_PORT -DnumShards=2 -jar start.jar'
        
 
应用场景：oracle java7 环境构建

install boot2docker
boot2docker up
boot2docker ssh
git clone https://github.com/supermy/mytools
cd mytools
docker build -t myjava7/debian myjava7
docker run -i -t myjava7/debian:latest
>java -version


----------------------------------------------------------------------    
集群todo

# 第一个-d表示让容器在后台运行
# 末尾的-D表示启动ssh的daemon模式，不然容器启动后立刻就变为停止状态了
docker run -i -t --name mydebian debian:latest
直接访问容器
docker run -i -t CONTAINER_ID /bin/bash


******************************************************************
todo 
    规则殷勤：drools 平台使用
    工作流：activiti
    云平台:
20150318
    增加rabbmitmq消息队列镜像包脚本；
    增加spring-boot-app and rabbmitmq运行脚本；

20150216
    docker run -d -P -m 1g redis
    docker run -d -P -e constraint:storage=ssd mysql

20150206
    mysolr带中文分词，基于solr4.10.2版本;solr自带运行环境。

20150205
    实时消息队列 mykafka
    /mystorm/
    即时通讯 myim
    
20150124
    docker run -v /usr/local/bin:/target jpetazzo/nsenter:latest
        nsenter（无需sshd、无需attach也可以登录容器）

    docker images|grep none|awk '{print $3}'|xargs docker rmi

    web+app(mytomcat,mynginx)
        tomcat and nginx 发布目录和日志绑定到宿主机

    hadoop+hbase+hive平台(version:1.21,2.10)
        kafuka应用
        mahout应用

     mydebian/myjava7重新定义，测试ok。
     

20150123
    删除无效镜像 docker images |grep none|awk '{print $3}'|xargs docker rmi
    增加mynginx镜像
        目录共享：通过主目录共享 $HOME/docker-share
    移入mytomcat镜像源码

    web+app(mytomcat,mynginx)
        fig-docker-nginx-tomcat-memcached-mysql 集群脚本完成,测试ok

20150120
    Fig 主要用来跟 Docker 一起来构建基于 Docker 的复杂应用，Fig 通过一个配置文件来管理多个Docker容器，非常适合组合使用多个容器进行开发的场景。
    安装配置fig:
    1.brew install fig
    2.$(boot2docker shellinit),或将脚本加入到.bash_profile中
    3.建立测试目录env-docker,测试http://www.fig.sh/index.html

20141106
    mongodb-cluster ok
    mydebian ok
    myjava7 ok
    mysolr todo
    
******************************************************************
#遇到的问题
/etc/hosts文件无法修改，这样你就不能自己做域名解析
VM的系统时间是UTC +0000的，而且貌似无法修改
Container的IP无法指定为静态IP，因此每次重启Container时，IP可能会变化

#绑定本地磁盘目录到容器目录
brew install sshfs 
cat tcuser > ~/.boot2docker/b2d-passwd 
#绑定本地目录到docker
sshfs docker@localhost:/mnt/sda1/dev ~/workspace/dev -p 2022 -o reconnect -o password_stdin < ~/.boot2docker/b2d-passwd 
#绑定docker目录到容器
docker run -i -t dev:base -v /mnt/sda1/dev:/src /bin/bash

#端口由容器内影射到本地机器
boot2docker ssh -L 8000:localhost:8000 
docker run -i -t -p 8000:8000

#Docker使用dnsmasq替代/etc/hosts解析

 
#基于Docker，构造测试和生产环境。
base debian nginx(淘宝)+tomcat+memcache 集群 
mongodb集群 
mysql集群 
oracle单机版 

全文检索：solr 
工作流：activity 
规则引擎：drools

dns容器ip地址配置； 

php框架
springmvc框架

web框架：jquery-mobile，extjs

docker build -t mydev/debian test

docker build -t mydb/mysql mysql

