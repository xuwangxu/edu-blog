# redis
## 1.redis简介
REmote DIctionary Server(Redis）是一个由Slvatore Sanfilippo 写的key-value内存中的数据结构存储系统，并且是开源（BSD许可）的，它可以用作数据库、缓存和消息中间件。

## 2.redis组成
- 它支持多种类型的数据结构，如字符串（string），散列（hashes），列表（lists）,集合（sets），有序集合（sorted sets）与范围查询，bitmaps,hyperloglogs和地理空间（geospatial）索引半径查询；
- redis内置了复制（replication）,LUA脚本（LUA scriptiong）,LRU驱动事件（LRU eviction），事务（transactions）和不同级别的磁盘持久化（persistence）;
-  并通过Redis哨兵（Sentinel）和自动分区（Cluster）提供高可用性（high availablility）.

## 3.redis优点
- 性能很高：redis支持超过100k每秒的读取频率；
- 丰富数据类型：redis支持二进制的String，Lisst,Hashes,Sets及Ordered Sets等数据类型操作；
- 原子：Redis的所有操作都是原则性的，同时Redis还支持对几个操作全并后的原则性执行；
- 丰富的特性：redis还支持publiss/subscribe，通知，key过期等等特性，redis支持异机主从复制

## 4.redis应用场景
- redis最佳使用场景的全部是数据in-memory
- redis更多场景是作为Memecached的替代品来使用
- 当需要除key/value之外的更多数据类型支持时，使用redis更合适
- 当存储的数据不能被提出时，使用redis更合适


## 5.redis小结
### 5.1redis数据库小结
- 提高了DB的可拓展性，只需要将新加的数据放到新加的服务器上就可以了
- 提高了DB的可用性，只影响到需要访问的shard服务器上的数据的用户
- 提高了DB的可维护性，对系统的升级和配置可以按shard一个个来搞，对服务产生的影响较小
- 小的数据库存的查询压力小，查询更快，性能更好

### 5.2redis应用小结
- 要进行Master-slave的配置，出现服务故障时可以支持切换
- 在master侧禁用数据持久化，只需在slave上配置数据持久化
- 物理内存+虚拟内存不足，这个dump一致死着，时间久了机器挂掉，这个情况下就是灾难
- 当Redis物理内存使用超过内存总容量的3/5时就会开始比较危险了，就开始做swap，内存碎片
- 当达到最大内存时，回清空带有过期时间的key，即使key未过期时间
- redis与DB同步写的问题，先写DB，后写redis，因为内存基本上没问题

## 6.redis部署搭建
在redis的官方网站(http://www.redis.io)下载最新的稳定版本redis
	操作命令如下：
### 6.1、获取redis安装包，并解压
    wget http://download.redis.io/releases/redis-3.0.7.tar.gz
    tar xzf redis-3.0.7.tar.gz
### 6.2、进入目录，并查看说明文件README
    cd redis-3.0.7
    less README
### 6.3、配置redis安装包
    make MANIFESTO=jemalloc && \
    make PREFIX=/application/redis-3.0.7 install
### 6.4、创建软连接
    ln -s /application/redis-3.0.7/ /application/redis
###	6.5、配置环境语言
    LANG=en_US.UTF-8
### 6.6、查看redis命令的目录
    tree /application/redis/bin/
        执行结果如下：
    /application/redis/bin/
    ├── redis-benchmark
    ├── redis-check-aof
    ├── redis-check-dump
    ├── redis-cli
    ├── redis-sentinel -> redis-server
    └── redis-server
	
	命令执行完成后，会在/applicaion/redis/bin/目录下生成5个可执行文件，分别是：
    redis-benchmark  redis-check-aof  redis-check-dump  redis-cli  redis-sentinel redis-server
    它们的作用如下：
    命令	说明
    redis-server	Redis服务器的daemon启动程序
    redis-cli	Redis命令行操作工具，当然也可以用telnet跟其纯文本协议来操作
    redis-benchmark	Redis性能测试工具，测试Redis在你的系统及你的配置下的读写
    redis-check-aof	更新日志检查
    redis-check-dump	用语本地数据库的检查

## 7.配置并启动redis服务
a. 配置环境变量，命令如下

    echo ' PATH=/application/redis/bin/:$PATH' >> /etc/profile
    source /etc/profile
查看执行结果如下：

    tail -1 /etc/profile
    提示：PATH=/application/redis/bin/:$PATH
查看是否被导入到全局路径下

    which redis-server
执行结果如下

    /application/redis/bin/redis-server
b. 查看命令帮助

    [root@mysql01 ~]# redis-server --help
    Usage: ./redis-server [/path/to/redis.conf] [options]
           ./redis-server - (read config from stdin)
           ./redis-server -v or --version
           ./redis-server -h or --help
           ./redis-server --test-memory <megabytes>
    Examples:
           ./redis-server (run the server with default conf)
           ./redis-server /etc/redis/6379.conf
           ./redis-server --port 7777
           ./redis-server --port 7777 --slaveof 127.0.0.1 8888
           ./redis-server /etc/myredis.conf --loglevel verbose

    Sentinel mode:
           ./redis-server /etc/sentinel.conf --sentinel
c. 启动redis服务
	操作命令：
	
    cp ~/redis-3.0.7/redis.conf /application/redis /#安装包内含有redis的默认配置文件
    a) echo vm.overcommit_memory=1 >> /etc/sysctl.conf
    b) sysctl vm.overcommit_memory=1 或执行echo vm.overcommit_memory=1 >>/proc/sys/vm/overcommit_memory

    redis-server  /application/redis/conf/redis.conf &
    提示：启动后会出现夯住的命令行，但是其实只要回车就好，因为此处回车后台执行
    提示:查看端口是否存在确定redis是否启动成功
    [root@mysql01 ~]# netstat -lntup|grep -w 6379
    tcp        0      0 0.0.0.0:6379                0.0.0.0:*                       LISTEN      5896/redis-server * 
    tcp        0      0 :::6379                     :::*                            LISTEN      5896/redis-server *
d. 关闭redis

    redis-cli  shutdown
    或者
    killall redis-server

## 8.redis主配置文件详解
    [root@dbserver ~]# grep -vE "#|^$" /application/redis/redis.conf
    1.redis默然不是以守护进程的方式运行，可以通过该配置修改，使用yes启用守护进程
    daemonize yes
    2.当redis以守护进程方式运行时，redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
    pidfile /var/run/redis.pid 
    3.指定redis监听端口，默认端口为6379，为了安全一般修改端口
    port 6397
    4.在高并发环境下需要一个backlog值来避免慢客户端连接问题。注意Linux内核默默地将这个值减少到/proc/sys/net/core/somaxconn的值，所以需要确认增大aomaxconn和tcp_max_syx_backlog两个值来表达想要的效果。
    tcp-backlog 511
    5.绑定主机地址
    bind 127.0.0.1
    6.当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭功能
    timeout 0
    7.tcp连接保活策略，可以通过tcp-keepalive配置项来进行设置，单位为妙，假如设置为60秒，则server端会每60秒向连接空闲的客户端发起一次ACK请求，以检查客户端是否已经挂掉，对于无响应的客户端则会关闭其连接，所以关闭一个连接最长需要120秒的时间，如果设置为0.则不进行保活检测。一般合理值为60秒
    tcp-keepalive 0
    8.指定日志级别，redis总共支持四个级别：debug/verbose/notice/warning，默认为verbose.debug记录很多信息，用于开发与测试；verbose：很多精简的有用信息，不像debug会记录那么多；notice普通的verbose，常用于生产环境；warning只有非常重要或者严重的信息会记录到日志
    loglevel notice
    9.指定日志文件名，也可以使用“stdout”来强制让redis把日志信息写到标准输出上，默认为标准输出，如果配置redis为守护进程的方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发给/dev/null
    logfile ""
    10.设置数据库的数量，默认数据为0，可以使用select<dbid>命令在连接上指定数据库ID
    databases 16
    11.指定在多长时间内，有多少次更新操作，就讲数据同步到数据文件，可以多个条件配合
    save <seconds> <changes>
    redis默认配置文件中提供了三个条件：
    save 900 1
    save 300 10
    save 60 10000
    分别表示900秒（15分钟）内1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
    12.如果开启了RDB快照功能，那么在redis持久化数据到磁盘时如果出现失败，默认情况下，redis会停止接受所有的写请求，这样做的好处在于可以让用户很明确的知道内存中的数据和磁盘上的数据已经存在不一致了，如果redis不顾这种不一致，一意孤行的继续接受请求，就可能会引起一些灾难性的的后果，如果下一次RDB持久化成功，redis会自动回复接受请求，当然，如果不在乎这种数据不一致或者有其他的手段发现和控制这种不一致的话，你完全可以关闭这个功能，以便在快照写入失败时，也能确保redis继续接受新的请求。
    stop-writes-on-bgsave-error yes
    13.当导出到.rdb数据库时是否用LZF压缩字符串对象，默认设置为“yes”如果想节省CPU的话，可以把这个设置为“no"，但是如果有压缩的key却没有压缩的话，那数据文件会变得更大。
    rdbcompression yes
    14.因为版本5的RDB有一个CRC64算法的效验和放在了文件的最后。这将使文件格式更加可靠但在生产和加载RDB文件时，这有一个性能消耗（大约10%）,所以可以关掉它来获取最好的新能，生成的关闭效验的RDB文件有一0的效验和，它将告诉加载代码跳过检查
    rdbchecksum yes
    15.数据库的文件名及存放路径
    dbfilename dump.rdb
    16.工作目录，本地数据库会写到这个目录下，文件名是上面的“dbfilename”的值,累加文件也放这里，注意你这里指定的必须是目录，不是文件名。
    dir ./
    17.当一个slave失去和master的连接，或者同步正在进行中，slave的行为有两种可能：1）如果slave-server-stale-data设置为“yes”(默认值)，slave会继续响应客户端请求，可能是正常数据，也可能是还没获得值的空数据。
    2）如果slave-server-stale-data设置为“no”,slave会回复“正在从master同步（SYNC with master in progress）”来处理各种请求，除了INFO和SLAVEOF命令。
    slave-serve-stale-data yes
    18.可以配置slave实例是否接受写操作。可写的slave实例可能对存储临时数据比较有用（因为写入slave#的数据在同master同步之后将很容易被删除），但是如果客户端由于配置错误在写入时也可能产生一些问题，从Redis2.6默认所有的slave为只读，注意：只读slave不是为了暴露给互联网上不可信的客户端而设计的，它只是一个防止实例误用的保护层。一个只读的slave支持所有的管理命令比如config,debug等。为了限制你可以用“rename-command”来，隐藏所有的管理和危险命令来增强只读slave的安全性
    slave-read-only yes
    19.复制集同步策略：磁盘或者socket，新slave连接或者老slave重新连接时候不能只接收不同，得做一个全同步。需要一个新的RDB文件dump出来，然后从master传到slave。可能有两种情况：1）基于硬盘（disk-backed）:master创建一个新的进程dump RDB，完事儿之后由父进程（即主进程）增量传给slave；2）基于socket（diskless）:master创建一个新进程直接dump RDB到slave的socket，不经过主进程，不经过硬盘。基于硬盘的话，RDB文件创建后，一旦创建完毕，可以同时服务更多的slave。基于socket的话，新slave来了后，得排队（如果超出了repl-diskless-sync-delay还没来)，完事儿一个再进行下一个。当用diskless的时候，master等待一个repl-diskless-sync-dely的秒数，如果没slave来的话，就直接传，后来的得排队等了，否则就可以一起传，disk较慢，并且网络较快的时候，可以用diskless(默认用disk-based)
    repl-diskless-sync no
    20.设置成0的话，传输开始ASAP
    repl-diskless-sync-delay 5
    21.是否在slave套接字发送SYNC之后禁用TCP_NODELAY？如果选择yes redis将使用更少的TCP包和带宽来向slave发送数据，但是这将使数据传输到slave上有延迟，Linux内核的默认会达到40毫秒，如果选中了no数据传输到slave的延迟将会减少但要使用更多的带宽默认我们会为低延迟做优化，但高流量情况或主从之间的跳数过多时，把这个选项设置为yes是个不错的选择。
    repl-disable-tcp-nodelay no
    22.slave的优先级是一个整数展示在redis的info输出中，如果master不再正常工作了，sentinel将用它来选择一个slave提升=升为master。优先级数字小的slave会优先考虑提升为master，所以例如有三个slave优先级分别为10,100,25，sentinel将挑选优先级最小数字为10的slave，0作为一个特殊的优先级，标示这个slave不能作为master，所以一个优先级为0的slave永远不会被sentinel挑选提升为master，默认优先级为100
    slave-priority 100
    23.要求客户端在处理任何命令时都要验证身份和密码，这个功能在有你不信任的其它客户端能够访问redis服务器的环境里非常有用，为了向后兼容的话这段应该注释掉。而且大多数人不需要身份证（例如：它们运行在自己的服务器上）警告：因为redis太快了，所以外面的人可以尝试每秒150k的密码来试图破解密码。这意味着你需要一个高强度的密码，否则容易破解太容易了
    requirepass 898redis
    24.默认情况下，redis是异步的把数据导出到磁盘上。这种模式在很多应用里已经足够好，但redis进程出问题或断电时可能造成一段时间的写操作丢失（这取决于配置的save指令)。AOF是一种提供了更可靠的替代持久化模式，例如使用默认的数据写入文件策略，在遇到像服务器断电或单写情况下redis自身进程出现问题但操作系统仍正常运行等突发事件时，redis能丢失1秒的写操作，AOF和RDB持久化能同时启动并且不会有问题。如果AOF开启，那么在启动时redis将加载AOF文件，它更能保证数据的可靠性
    appendonly no
    25.追加加文件名字，默认appendonly.aof
    appendfilename "appendonly.aof"
    26.fsync()系统调用告诉操作系统把数据写到磁盘上，而不是等更多的数据进入输出缓冲区。有些操作系统会真的把数据马上刷到磁盘上；有些则会快速去尝试这么做。
    redis支持三种不同的模式：
    no：不要立刻刷，只有操作系统需要刷的时候再刷。比较快；
    always：每次写操作都立刻写入到AOF文件。慢，但是最安全
    everysec：每秒写一次。折中方案。
    默认的“everysec”通常来说能在速度和数据安全性之间取得比较好的平衡。根据你的理解来决定，如果你能放宽该配置为“no”来获取更好的性能（但如果你能忍受一些数据丢失，可以考虑使用默认的快照持久化模式），或者相反，用“always”会比较慢但比everysec要更安全。
    appendfsync everysec
    27.如果AOF的同步策略设置成“always”或者“everysec”,并且后台的存储进程（后台存储进程或写入AOF日志）会产生很多磁盘I/0开销。某些Linux的配置下会使redis因为fsync()系统调用而阻塞很久。注意，目前对这个情况还没有完美修正，甚至不同线程的fsync（）会阻塞我们同步的write（2）调用。为了缓解这个问题，可以用下面这个选项。它可以在BGSAVE或BGREWRITEAOF处理时组织fsync（）.这就意味着如果有子进程在进行保存操作，那么redis就处于“不可同步”的状态。这实际上是说，在最差的情况下可能会丢掉30秒钟的日志数据（默认Linux设定）如果把这个设置成yes带来了延迟问题，就保持no，这是保存持久数据的最安全的方式。
    no-appendfsync-on-rewrite no
    28.自动重写AOF文件，如果AOF日志文件增大到指定百分比，redis能够通过BGREWRITEAOF自动重写AOF日志文件。工作原理：redis记住上次重写AOF文件的大小（如果重启还没有写操作，就直接启动时的AOF大小）这个基准大小和当前大小做比较。如果当前大小超过指定比例，就会触发重写操作，还需要指定被重写日志的最小尺寸，这样避免了打到指定百分比但尺寸仍然很小的情况还要重写。指定百分比为0会禁用AOF自动重写特征。
    auto-aof-rewrite-percentage 100
    auto-aof-rewrite-min-size 64mb
    29.AOF文件可能在尾部是不完整的（上次system关闭有问题，尤其是mount ext4文件系统时没有加上data=ordered选项。只会发生在OS死时，redis自己死不会不完整）。那redis重启时load进内存的时候就有问题了。发生的时候，可以选择redis启动报错，或者load尽量多正常的数据。如果aof-load-truncated是yes，会自动发布一个log给客户端然后load（默认）如果是no,用户必须手动redis-check-aof修复AOF文件才可以。
    aof-load-truncated yes
    30.如果达到最大时间限制（毫秒)，redis会记个log，然后返回error，当一个脚本超过了最大时限，只有script kill 和shutdown nosave可以用，第一个可以杀没有调用了write，只能用第二个命令杀，设置成0或者负值，时限就无限。
    lua-time-limit 5000
    31.redis慢查询日志可以记录超过指定时间的查询。运行时间不包括各种I/O时间，例如：连接客户端，发送响应数据等，而只计算命令执行的实际时间（这只是线程阻塞而无法同时为其它请求服务的命令执行阶段）可以为慢查询日志配置；两个参数：一个指明redis的超时时间（单位为微妙）来记录超过时间的命令，另一个是慢查询日志长度。当一个新的命令被写进日志的时候，最老的那个记录从队列中移除，下面的时间单位是微妙，所以1000000就是1秒。注意，负数时间会禁用慢查询日志，而0则会强制记录所有命令。
    slowlog-log-slower-than 10000
    32.这个长度没有限制，只是要主要会消耗内存。可以通过slowlog reset来回收内存
    slowlog-max-len 128
    33.默认情况下禁用延迟监控，因为它基本上是不需要的，单位为毫秒
    latency-monitor-threshold 0
    34.redis能通知pub/Sub客户端关于键空间发生的事件例如：如果键空间事件通知被开启，并且客户端对0号数据库的键foo执行del命令时，将通过Pub/Sub发布两条消息：
    PUBLISH_keyspace@0_:foo del
    PUBLISH_keyevent@0_:del foo
    可以在下表中选择redis要通知的事件类型。事件类型由单个字符来标示：
    K 键空间通知，以_keyspace@<db>_为前缀
    E 键事件通知，以_keyevent@<db>_为前缀
    g DEL,EXPIRE,RENAME等类型无关的通用命令的通知。。。
    $ String命令
    l List命令
    s Set命令
    h Hash命令
    z 有序集合命令
    x 过期事件（每次key过期时生成）
    e 驱逐事件（当key在内存满了被清楚时生成）
    A g$lshzce的别名，因此“AKE”意味着所有的事件
    notify-keyspace-events带一个由0到多个字符组成的字符串参数。空字符串意思是通知被禁用。例子：启用list和通用事件通知：notify-keyspace-events Elg 例子2：为了获取过期key的通知订阅名字为_keyevent@_:expired的频道，用以下配置notify-keyspace-events Ex 默认所用的通知被禁用，因为用户通常不需要该特性，并且该特性会有性能耗损。注意所用的通知被禁用，因为用户通常不需要该特性，并且该特性会有性能耗损。注意如果你不指定至少k或E之一，不会发送任何事件。
    notify-keyspace-events ""
    35.当有大量数据时，适合用哈希编码（这会需要更多的内存），元素数量上限不能超过给定限制。redis hash是value内部为一个hashmap，如果该map的成员数比较少，则会采用类似一维线性的紧凑格式来存储该map，即省去了大量指针的内存开销，如果2个条件任意一个条件超过设置值都会转换成真正的hashmap，当value这个map内部不超过多少个成员时会采用线性紧凑格式存储，默认是64，即value内部有64个一下的成员就是使用线性紧凑存储，超过该值自动转成正在的hashmap
    hash-max-ziplist-entries 512
    36.当value这个map内部的每个成员值不超过多少字节就会采用线性紧凑存储来节省空间。
    hash-max-ziplist-value 64
    37.与hash-max-zipmap-entries哈希相类似，数据元素较少的情况下，可以用另一种方式来编码从而节省大量空间。list数据类型多少节点一下会采用去指针的紧凑存储格式
    list-max-ziplist-entries 512
    38.list数据类型节点值小小于多少字节会采用紧凑格式
    list-max-ziplist-value 64
    39.还有这样一种特殊编码的情况：数据全是64位无符号整形数字构成的字符串。下面这个配置就是用来限制这种情况下使用这种编码的最大上限的。
    set-max-intset-entries 512
    40.与第一、第二种情况类似，有序序列可以用一种特别的编码方式来处理，可节省大量空间。这种编码只适合长度和元素都符合下面限制的有序序列：
    zset-max-ziplist-entries 128
    zset-max-ziplist-value 64
    41.关于Hyperloglogs稀疏表示限制设置，如果其值大于16000，则仍然采用稠密表示，因为这时稠密表示更能有效使用内存，建议值为3000
    hll-sparse-max-bytes 3000
    42. 哈希刷新，每100个CPU毫秒会拿出1个毫秒来刷新Redis的主哈希表(顶级键值映射表)。redis所用的哈希表实现(见dict.c)采用延迟哈希刷新机制：你对一个哈希表操作越多，哈希刷新操作就越频繁；反之，如果服务器非常不活跃那么也就是用点内存保存哈希表而已。默认是每秒钟进行10次哈希表刷新，用来刷新字典，然后尽快释放内存。建议：如果对延迟比较在意的话就用 "activerehashing no"，每个请求延迟2毫秒不太好嘛。如果你不太在意延迟而希望尽快释放内存的话就设置 
    activerehashing yes
    43. 客户端的输出缓冲区的限制，可用于强制断开那些因为某种原因从服务器读取数据的速度不够快的客户端，（一个常见的原因是一个发布/订阅客户端消费消息的速度无法赶上生产它们的速度）可以对三种不同的客户端设置不同的限制： normal -> 正常客户端slave -> slave和 MONITOR 客户端pubsub -> 至少订阅了一个pubsub channel或pattern的客户端，下面是每个client-output-buffer-limit语法:
    
client-output-buffer-limit <class><hard limit> <soft limit> <soft seconds>一旦达到硬限制客户端会立即被断开，或者达到软限制并持续达到指定的秒数（连续的）。例如，如果硬限制为32兆字节和软限制为16兆字节/10秒，客户端将会立即断开如果输出缓冲区的大小达到32兆字节，或客户端达到16兆字节并连续超过了限制10秒，就将断开连接。默认normal客户端不做限制，因为他们在不主动请求时不接收数据（以推的方式），只有异步客户端，可能会出现请求数据的速度比它可以读取的速度快的场景。pubsub和slave客户端会有一个默认值，因为订阅者和slaves以推的方式来接收数据，把硬限制和软限制都设置为0来禁用该功能
    client-output-buffer-limit normal 0 0 0
    client-output-buffer-limit slave 256mb 64mb 60
    client-output-buffer-limit pubsub 32mb 8mb 60
    44.redis调用内部函数来执行许多后台任务，如关闭客户端超时的连接，清除未被请求过得过期key等等。不是所有的任务都以相同的频率执行，但redis依照指定的“hz”值来执行检查任务。默认情况下，“hz”的被设定为10.提高该值将在redis空闲时使用更多的CPU时，但同时当有多个key同时到期会使redis的反应更灵敏，以及超时可以更精确的处理。范围是1到500之间，但是值超过100通常不是一个好主意。大多数用户应该使用10这个默认值，只有在非常低的延迟要求时有必要提高到100.
    hz 10
    45.当一个子进程重写AOF文件时，如果启用下面的选项，则文件没生成32M数据会被同步。为了增量式的写入硬盘并且避免大的延迟高峰这个指令是非常有用的
    aof-rewrite-incremental-fsync yes
