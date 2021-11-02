# 一、Zookeeper简介

一、zk简介说明
zk是一个高效的分布式协调服务，他暴露了一些共用服务，比如命名、配置、管理、同步控制、群组服务等我们可以使用zk来实现比如打成共识，集群管理、leader选举等。
zk是一个高可用的分布式管理与协调框架，基于ZAB（原子消息广播协议）的实现。该框架能够很好地保证分布式环境中数据的一致性。也正是基于这样的特性。
顺序的一致性：从一个客户端发起事物请求。最终将会严格地按照其发起的顺序被应用到zk中去。
原子性：所书事物请求的处理结果在整个集群中国所有机器上的应用情况是一致的。也就是说。要么整个集群所有的机器都能成功应用到了某一事物。要么有没有应用，不一定会出现部分的机器应用到了该事物，而一部分没有应用的情况。
单一视图：无论客户端连接是哪一个zk服务器，其看到的服务器数据模型都是一致的。
可靠性：一旦服务器成功地应用了一个事物。并完成对客户端的响应.那么该事物所引起的服务端状态将会被一致保留下来。除非有另外一个事物对其更改。
实时性：通常所说的实时性就是指一旦事物被成功应用，那么客服端就能立刻从服务器上获取变更的数据、zk仅仅能保证在一段时间内。客服端最终一定能从服务器读取最新的数据状态。





# 二、配置文件讲解、客户端的使用

## 一、Zookeeper基础知识、体系结构、数据模型Zookeepr

```
   1.      zookeeper是一个类似hdfs的树形文件结构，zookeeper可以用来保证数据在(zk)集
            群之间的数据的事务性一致、
   2.      zookeeper有watch事件，是一次性触发的，当watch监视的数据发生变化时，通
           知设置了该watch的client，即watcher
  3.      zookeeper有三个角色：Learner，Follower，Observer
  4.      zookeeper应用场景：
                    统一命名服务（Name Service）
                    配置管理（Configuration Management）
                    集群管理（Group Membership）
                    共享锁（Locks）
                    队列管理
```

## 二、Zookeeper配置（搭建zookeeper服务器集群）

```
  1. 结构：一共三个节点 (zk服务器集群规模不小于3个节点),要求服务器之间系统时间保持一致。
  2.上传zk
          进行解压： tar zookeeper-3.4.5.tar.gz
          重命名： mv zookeeper-3.4.5 zookeeper
          修改环境变量： vi /etc/profile 
  3.ZOOKEEPER_HOME=/usr/local/zookeeper 
   export     PATH=.:$HADOOP_HOME/bin:$ZOOKEEPER_HOME/bin:$JAVA_HOME/...
   刷新： source /etc/profile
   到zookeeper下修改配置文件
            cd /usr/local/zookeeper/conf
            mv zoo_sample.cfg zoo.cfg
 修改conf: vi zoo.cfg 修改两处
     （1）dataDir=/usr/local/zookeeper/data
     （2）最后面添加
                server.0=wuliang:2888:3888
                server.1=hadoop1:2888:3888
                server.2=hadoop2:2888:3888
服务器标识配置：
            创建文件夹：mkdir data
            创建文件myid并填写内容为0：vi
            myid (内容为服务器标识 ： 0)
            进行复制zookeeper目录到hadoop01和hadoop02
还有/etc/profile文件
把hadoop01、hadoop02中的myid文件里的值修改为1和2
路径(vi /usr/local/zookeeper/data/myid)
启动zookeeper：
路径：/usr/local/zookeeper/bin
执行：zkServer.sh start
(注意这里3台机器都要进行启动)
状态：zkServer.sh
status(在三个节点上检验zk的mode,一个leader和俩个follower)
1.3 操作zookeeper (shell)
zkCli.sh 进入zookeeper客户端
根据提示命令进行操作：
        查找：ls    / ls /zookeeper
        创建并赋值：create   /wuliang hadoop
        获取：get /wuliang
        设值：set /wuliang master
        可以看到zookeeper集群的数据一致性
        创建节点有俩种类型：短暂（ephemeral）
        持久（persistent）
```

## 三、zoo.cfg详解：

```
tickTime： 基本事件单元，以毫秒为单位。这个时间是作为 Zookeeper
服务器之间或客户端与服务器之间维持心跳的时间间隔，
也就是每隔 tickTime时间就会发送一个心跳。
dataDir：存储内存中数据库快照的位置，顾名思义就是 Zookeeper
保存数据的目录，默认情况下，Zookeeper
将写数据的日志文件也保存在这个目录里。
clientPort： 这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper
会监听这个端口，接受客户端的访问请求。
initLimit： 这个配置项是用来配置 Zookeeper接受客户端初始化连接时最长能忍受多少个心跳时间间隔数，
当已经超过 10 个心跳的时间（也就是 tickTime）长度后
Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是10*2000=20 秒。
syncLimit： 这个配置项标识 Leader 与 Follower之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime
的时间长度，总的时间长度就是 5*2000=10 秒
server.A = B:C:D :
        A表示这个是第几号服务器,
        B 是这个服务器的 ip 地址；
        C 表示的是这个服务器与集群中的 Leader
        服务器交换信息的端口；
        D 表示的是万一集群中的 Leader
        服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader
```

## 四、JAVA 怎么操作zookeeper

```
zk的构造器的参数详解
      String connectString  连接服务器列表，已，分割
      int sessionTimeout     心跳检测时间周期 （毫秒）
      Watcher watcher        事件处理通知
      canBeReadOnly          标识当前会话是否支持只读
      long sessionId      sessionPasswd      提供连接zookeeper的sessionId 和密码，通过这两个确定唯一一台客户端，             目前可以提供重复会话

    /** zookeeper地址 */
    static final String CONNECT_ADDR = "172.20.58.192:30023,172.20.58.113:30023,172.20.58.85:30023";
    /** session超时时间 */
    static final int SESSION_OUTTIME = 2000;//ms 
    /** 信号量，阻塞程序执行，用于等待zookeeper连接成功，发送成功信号 */
    static final CountDownLatch connectedSemaphore = new CountDownLatch(1);

    public static void main(String[] args) throws Exception{

        ZooKeeper zk = new ZooKeeper(CONNECT_ADDR, SESSION_OUTTIME, new Watcher(){
            @Override
            public void process(WatchedEvent event) {
                //获取事件的状态
                KeeperState keeperState = event.getState();
                EventType eventType = event.getType();
                //如果是建立连
                if(KeeperState.SyncConnected == keeperState){
                    if(EventType.None == eventType){
                        //如果建立连接成功，则发送信号量，让后续阻塞程序向下执行
                        connectedSemaphore.countDown();
                        System.out.println("zk 建立连接");
                    }
                }
            }
        });
        //进行阻塞
        connectedSemaphore.await();

        System.out.println("..");
```

## 五、创建节点

### 第一个create的介绍

```
        zk.create("/testRoot", "testRoot".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        第一个参数： 路径的问题
        第二个参数： 数据
        第三个参数： 认证的方式
        第四个参数： 创建节点的持久化
        关闭连接 zk.close();
        总结
            1.不能创建相同的的节点
            2.如果在某个父节点下面创建某个子节点 要写全路径
            3.临时节点 一次会话
```

## 怎么获取、更新、判断上面的节点zk上面节点的数据

```
	//获取节点洗信息
            byte[] data = zk.getData("/testRoot", false, null);
            System.out.println(new String(data));
            
    //获取该路径下面的所有子节点（直接子节点）
            System.out.println(zk.getChildren("/testRoot", false)); 
            byte[] data = zk.getData("/testRoot", false, null);
            zk.getChildren("/testRoot", false)

  //修改节点的值
  
		    zk.setData("/testRoot", "modify data root".getBytes(), -1);
		    byte[] data = zk.getData("/testRoot", false, null);

 //判断节点是否存在
		    System.out.println(zk.exists("/testRoot/children", false));
```

### 第二个delete的方法

```
      	//删除节点
               -1   表示全部删除
				zk.delete("/testRoot/children", -1);  
```

## 总结：

### zookpeer分布式锁的呢

```
              第一可以采用zk的临时节点，先读取该临时节点上有没有数据，
               如果有没有则创建，如果有则更新。
               注意:
                     原生的zk不允许递归创建节点，也支持节点递归的删除
```

## 六、同步和异步

```
 异步方式：（在同步参数的基础上添加两个参数）
       参数五：注册一个异步回调函数，要实现AsyncCallBackStringCallBack接口，重写
       processResult（int rc,String path,Object ctx,String name）方法，当节点创建完毕
       后执行此方法。
       rc：为服务端响应吗 0 表示调用成功，-4 表示端口连接，-110表示指定节点存在，-112
       表示会话已经过期。
       path：接口调用时传入API的数据节点的路径参数
       ctx：为调用接口传入API的ctx值
       name：实际在服务器端创建节点的名称
       参数六，传递给回调的从参数，一般为上下文（context）信息
```









# 三、Zookeeper的命令

## 一、这么连接客户端

```
zkCli.sh的使用
ZooKeeper服务器简历客户端
    ./zkCli.sh -timeout 0 -r -server ip:port
   ./zkCli.sh -timeout 5000 -server 192.9.200.242:2181
  -r ：即使ZooKeeper服务器集群一般以上的服务器宕机，也给客户端体统读服务ls path:查看某个节点下的所有子节点信息
```

## Zookeeper的命令使用

```
ls / :列出根节点下所有的子节点信息
stat path :获取指定节点的状态信息
     czxid 创建该节点的事物ID
           ctime 创建该节点的时间
           mZxid 更新该节点的事物ID
          mtime 更新该节点的时间
         pZxid 操作当前节点的子节点列表的事物ID(这种操作包含增加子节点，删除子节点)
        cversion 当前节点的子节点版本号
        dataVersion 当前节点的数据版本号
        aclVersion 当前节点的acl权限版本号
        ephemeralowner 当前节点的如果是临时节点，该属性是临时节点的事物ID
        dataLength 当前节点的d的数据长度
        umchildren 当前节点的子节点个数
```

## 查看节点数据的信息

get path 获取当前节点的数据内容
ls2 path :是ls 和 stat两个命令的结合

## 三、创建节点

```
 create [-s] [-e] path data acl
    -s 表示是顺序节点
    -e 标识是临时节点
     path 节点路径
    data 节点数据
     acl 节点权限
     
注：临时节点在客户端结束与服务器的会话后，自动消失
 quit  :退出客户端
```

## 四、赋值

```
set path data [version] :修改当前节点的数据内容  如果指定版本，需要和当     前节点的数据版本一致
```

## 五、删除

```
  delete path [version] 删除指定路径的节点 如果有子节点要先删除子节点
  rmr path 删除当前路径节点及其所有子节点
 setquota -n|-b val path 设置节点配额（比如限制节点数据长度，限制节点        中子节点个数）
 -n 是限制子节点个数 -b是限制节点数据长度
 超出配额后，ZooKeeper不会报错，而是在日志信息中记录
 tail zookeeper.out
 listquota path 查看路径节点的配额信息
 delquota [-n|-b] path 删除节点路径的配额信息 
 connect host:port 和 clost
 在当前连接中连接其他的ZooKeeper服务器和关闭服务器
 history 和 redo cmdno :查看客户端这次会话所执行的所有命令 和 执行指定      历史命令          
```







# 四、Zookeeper Watcher核心机制讲解、实现多个watcher集群操作

## 一、Watcher 、ZK状态、事件类型

```
  zookeeper有watch事件，是一次性触发的，当watch监视的数据发生变化时，通知设置了该watch的client的watch
  同样，其watch是监听数据发生了某些变法，那就一定会有对应的时间类型和状态类型
  事件类型：（znode节点想过的）
      EventType.NodeCreated                  创建
      EventType.NodeDataChanged         节点的数据发生变更
      EventType.NodeChildrenChanged    子节点发生变更
      EventType.NodeDeteted                    
  
  状态类型 （是跟客户端实例相关的）
      KeeperState.Disconnected  连接不上
      KeeperState.SyncConnected  连接上了
      KeeperState.AuthFailed          认证失败
      KeeperState.Expired               过期
```

## 二、 代码的实现

```
  public class ZooKeeperWatcher implements Watcher {
        /** 定义原子变量 */
        AtomicInteger seq = new AtomicInteger();
        /** 定义session失效时间 */
        private static final int SESSION_TIMEOUT = 10000;
        /** zookeeper服务器地址 */
        private static final String CONNECTION_ADDR = "192.168.80.88:2181";
        /** zk父路径设置 */
        private static final String PARENT_PATH = "/testWatch";
        /** zk子路径设置 */
        private static final String CHILDREN_PATH = "/testWatch/children";
        /** 进入标识 */
        private static final String LOG_PREFIX_OF_MAIN = "【Main】";
        /** zk变量 */
        private ZooKeeper zk = null;
        /** 信号量设置，用于等待zookeeper连接建立之后 通知阻塞程序继续向下执行 */
        private CountDownLatch connectedSemaphore = new CountDownLatch(1);

    /**
     * 创建ZK连接
     * @param connectAddr ZK服务器地址列表
     * @param sessionTimeout Session超时时间
     */
    public void createConnection(String connectAddr, int sessionTimeout) {
        this.releaseConnection();
        try {
            zk = new ZooKeeper(connectAddr, sessionTimeout, this);
            System.out.println(LOG_PREFIX_OF_MAIN + "开始连接ZK服务器");
            connectedSemaphore.await();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 关闭ZK连接
     */
    public void releaseConnection() {
        if (this.zk != null) {
            try {
                this.zk.close();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 创建节点
     * @param path 节点路径
     * @param data 数据内容
     * @return 
     */
    public boolean createPath(String path, String data) {
        try {
            //设置监控(由于zookeeper的监控都是一次性的所以 每次必须设置监控)
            this.zk.exists(path, true);
            System.out.println(LOG_PREFIX_OF_MAIN + "节点创建成功, Path: " + 
                               this.zk.create(	/**路径*/ 
                                                path, 
                                                /**数据*/
                                                data.getBytes(), 
                                                /**所有可见*/
                                                Ids.OPEN_ACL_UNSAFE, 
                                                /**永久存储*/
                                                CreateMode.PERSISTENT ) + 	
                               ", content: " + data);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }

    /**
     * 读取指定节点数据内容
     * @param path 节点路径
     * @return
     */
    public String readData(String path, boolean needWatch) {
        try {
            return new String(this.zk.getData(path, needWatch, null));
        } catch (Exception e) {
            e.printStackTrace();
            return "";
        }
    }

    /**
     * 更新指定节点数据内容
     * @param path 节点路径
     * @param data 数据内容
     * @return
     */
    public boolean writeData(String path, String data) {
        try {
            System.out.println(LOG_PREFIX_OF_MAIN + "更新数据成功，path：" + path + ", stat: " +
                                this.zk.setData(path, data.getBytes(), -1));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return false;
    }

    /**
     * 删除指定节点
     * 
     * @param path
     *            节点path
     */
    public void deleteNode(String path) {
        try {
            this.zk.delete(path, -1);
            System.out.println(LOG_PREFIX_OF_MAIN + "删除节点成功，path：" + path);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 判断指定节点是否存在
     * @param path 节点路径
     */
    public Stat exists(String path, boolean needWatch) {
        try {
            return this.zk.exists(path, needWatch);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 获取子节点
     * @param path 节点路径
     */
    private List<String> getChildren(String path, boolean needWatch) {
        try {
            return this.zk.getChildren(path, needWatch);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 删除所有节点
     */
    public void deleteAllTestPath() {
        if(this.exists(CHILDREN_PATH, false) != null){
            this.deleteNode(CHILDREN_PATH);
        }
        if(this.exists(PARENT_PATH, false) != null){
            this.deleteNode(PARENT_PATH);
        }		
    }

    /**
     * 收到来自Server的Watcher通知后的处理。
     */
    @Override
    public void process(WatchedEvent event) {

        System.out.println("进入 process 。。。。。event = " + event);

        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        if (event == null) {
            return;
        }

        // 连接状态
        KeeperState keeperState = event.getState();
        // 事件类型
        EventType eventType = event.getType();
        // 受影响的path
        String path = event.getPath();

        String logPrefix = "【Watcher-" + this.seq.incrementAndGet() + "】";

        System.out.println(logPrefix + "收到Watcher通知");
        System.out.println(logPrefix + "连接状态:\t" + keeperState.toString());
        System.out.println(logPrefix + "事件类型:\t" + eventType.toString());

        if (KeeperState.SyncConnected == keeperState) {
            // 成功连接上ZK服务器
            if (EventType.None == eventType) {
                System.out.println(logPrefix + "成功连接上ZK服务器");
                connectedSemaphore.countDown();
            } 
            //创建节点
            else if (EventType.NodeCreated == eventType) {
                System.out.println(logPrefix + "节点创建");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                this.exists(path, true);
            } 
            //更新节点
            else if (EventType.NodeDataChanged == eventType) {
                System.out.println(logPrefix + "节点数据更新");
                System.out.println("我看看走不走这里........");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(logPrefix + "数据内容: " + this.readData(PARENT_PATH, true));
            } 
            //更新子节点
            else if (EventType.NodeChildrenChanged == eventType) {
                System.out.println(logPrefix + "子节点变更");
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(logPrefix + "子节点列表：" + this.getChildren(PARENT_PATH, true));
            } 
            //删除节点
            else if (EventType.NodeDeleted == eventType) {
                System.out.println(logPrefix + "节点 " + path + " 被删除");
            }
            else ;
        } 
        else if (KeeperState.Disconnected == keeperState) {
            System.out.println(logPrefix + "与ZK服务器断开连接");
        } 
        else if (KeeperState.AuthFailed == keeperState) {
            System.out.println(logPrefix + "权限检查失败");
        } 
        else if (KeeperState.Expired == keeperState) {
            System.out.println(logPrefix + "会话失效");
        }
        else ;

        System.out.println("--------------------------------------------");

    }

    /**
     * <B>方法名称：</B>测试zookeeper监控<BR>
     * <B>概要说明：</B>主要测试watch功能<BR>
     * @param args
     * @throws Exception
     */
    public static void main(String[] args) throws Exception {

        //建立watcher
        ZooKeeperWatcher zkWatch = new ZooKeeperWatcher();
        //创建连接
        zkWatch.createConnection(CONNECTION_ADDR, SESSION_TIMEOUT);
        //System.out.println(zkWatch.zk.toString());

        Thread.sleep(1000);

        // 清理节点
        zkWatch.deleteAllTestPath();

        if (zkWatch.createPath(PARENT_PATH, System.currentTimeMillis() + "")) {

            Thread.sleep(1000);


            // 读取数据
            System.out.println("---------------------- read parent ----------------------------");
            //zkWatch.readData(PARENT_PATH, true);

            // 读取子节点
            System.out.println("---------------------- read children path ----------------------------");
            zkWatch.getChildren(PARENT_PATH, true);

            // 更新数据
            zkWatch.writeData(PARENT_PATH, System.currentTimeMillis() + "");

            Thread.sleep(1000);

            // 创建子节点
            zkWatch.createPath(CHILDREN_PATH, System.currentTimeMillis() + "");

            Thread.sleep(1000);

            zkWatch.writeData(CHILDREN_PATH, System.currentTimeMillis() + "");
        }

        Thread.sleep(50000);
        // 清理节点
        zkWatch.deleteAllTestPath();
        Thread.sleep(1000);
        zkWatch.releaseConnection();
    }

}
```

## 三、总结

```
zookeeper 是如何监听到节点的变法。他有watch的事件，通过watch观察到节点的变法，然后触发process事件
然后检查数据类型，在检测状态类的节点变法。
具体怎么做：
    第一步：注册watch事件，使用zookeeper里面拥有该事件，什么时候触发
    第二步：需要想节点的注册是否启动事件 意思就是是否触发process方法
    在这里需要注意：
           监听只能监听到已经子节点的变法，如果想法在子节点发生改变的时候，那么父节点需要启动事件
           每次监听都要开启是否启用事件。上面有详细的代码过程。
```









# 五、Zookeeper的ACL(AUTH)

## 一、ACL（Access contol List）

```
什么是ACL，Zookeeper作为一个分布式协调框架，其内部存储的都是一些关乎分布式系统运行时状态的元数据
，尤其是设计到了一些分布式锁，Master选举和协调应用场景，有效的保障Zookeeper中的数据安全，Zookeeper
ZK提供了三种模式，权限模式，授权对象，权限
权限模式：Scheme 开发人员最多使用的如下四种权限模式
     IP: ip模式通过ip地址，来进行权限控制
     Digest： digest是最常用的权限控制模式，zk会对形成的权限标识先后进行两次编码处理，分别是SHA-1加密        算法、BASE64编码。
     World：World是一值最开放的权限控制模式、这种模式可以看做为特殊的Digest，他仅仅是一个标识而已
     Super：超级用户模式，在超级用户模式下可以ZK进行操作
权限： 权限就是指那些通过权限检测后可以允许执行的操作，在ZK中，对数据的操作权限分为以下五大类
    CREATE、Delete、READ、WRITE、ADMIN
```

## 二、代码的实现

```
    /**
     * Zookeeper 节点授权
     * @author（alienware）
     * @since 2015-6-14
     */
    public class ZookeeperAuth implements Watcher {

        /** 连接地址 */
        final static String CONNECT_ADDR = "192.168.80.88:2181";
        /** 测试路径 */
        final static String PATH = "/testAuth";
        final static String PATH_DEL = "/testAuth/delNode";
        /** 认证类型 */
        final static String authentication_type = "digest";
        /** 认证正确方法 */
        final static String correctAuthentication = "123456";
        /** 认证错误方法 */
        final static String badAuthentication = "654321";

        static ZooKeeper zk = null;
        /** 计时器 */
        AtomicInteger seq = new AtomicInteger();
        /** 标识 */
        private static final String LOG_PREFIX_OF_MAIN = "【Main】";

        private CountDownLatch connectedSemaphore = new CountDownLatch(1);

        @Override
        public void process(WatchedEvent event) {
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (event==null) {
                return;
            }
            // 连接状态
            KeeperState keeperState = event.getState();
            // 事件类型
            EventType eventType = event.getType();
            // 受影响的path
            String path = event.getPath();

            String logPrefix = "【Watcher-" + this.seq.incrementAndGet() + "】";

            System.out.println(logPrefix + "收到Watcher通知");
            System.out.println(logPrefix + "连接状态:\t" + keeperState.toString());
            System.out.println(logPrefix + "事件类型:\t" + eventType.toString());
            if (KeeperState.SyncConnected == keeperState) {
                // 成功连接上ZK服务器
                if (EventType.None == eventType) {
                    System.out.println(logPrefix + "成功连接上ZK服务器");
                    connectedSemaphore.countDown();
                } 
            } else if (KeeperState.Disconnected == keeperState) {
                System.out.println(logPrefix + "与ZK服务器断开连接");
            } else if (KeeperState.AuthFailed == keeperState) {
                System.out.println(logPrefix + "权限检查失败");
            } else if (KeeperState.Expired == keeperState) {
                System.out.println(logPrefix + "会话失效");
            }
            System.out.println("--------------------------------------------");
        }
        /**
         * 创建ZK连接
         * 
         * @param connectString
         *            ZK服务器地址列表
         * @param sessionTimeout
         *            Session超时时间
         */
        public void createConnection(String connectString, int sessionTimeout) {
            this.releaseConnection();
            try {
                zk = new ZooKeeper(connectString, sessionTimeout, this);
                //添加节点授权
                zk.addAuthInfo(authentication_type,correctAuthentication.getBytes());
                System.out.println(LOG_PREFIX_OF_MAIN + "开始连接ZK服务器");
                //倒数等待
                connectedSemaphore.await();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        /**
         * 关闭ZK连接
         */
        public void releaseConnection() {
            if (this.zk!=null) {
                try {
                    this.zk.close();
                } catch (InterruptedException e) {
                }
            }
        }

        /**
         * 
         * <B>方法名称：</B>测试函数<BR>
         * <B>概要说明：</B>测试认证<BR>
         * @param args
         * @throws Exception
         */
        public static void main(String[] args) throws Exception {

            ZookeeperAuth testAuth = new ZookeeperAuth();
            testAuth.createConnection(CONNECT_ADDR,2000);
            List<ACL> acls = new ArrayList<ACL>(1);
            for (ACL ids_acl : Ids.CREATOR_ALL_ACL) {
                acls.add(ids_acl);
            }

            try {
                zk.create(PATH, "init content".getBytes(), acls, CreateMode.PERSISTENT);
                System.out.println("使用授权key：" + correctAuthentication + "创建节点："+ PATH + ", 初始内容是: init content");
            } catch (Exception e) {
                e.printStackTrace();
            }
            try {
                zk.create(PATH_DEL, "will be deleted! ".getBytes(), acls, CreateMode.PERSISTENT);
                System.out.println("使用授权key：" + correctAuthentication + "创建节点："+ PATH_DEL + ", 初始内容是: init content");
            } catch (Exception e) {
                e.printStackTrace();
            }

            // 获取数据
            getDataByNoAuthentication();
            getDataByBadAuthentication();
            getDataByCorrectAuthentication();

            // 更新数据
            updateDataByNoAuthentication();
            updateDataByBadAuthentication();
            updateDataByCorrectAuthentication();

            // 删除数据
            deleteNodeByBadAuthentication();
            deleteNodeByNoAuthentication();
            deleteNodeByCorrectAuthentication();
            //
            Thread.sleep(1000);

            deleteParent();
            //释放连接
            testAuth.releaseConnection();
        }
        /** 获取数据：采用错误的密码 */
        static void getDataByBadAuthentication() {
            String prefix = "[使用错误的授权信息]";
            try {
                ZooKeeper badzk = new ZooKeeper(CONNECT_ADDR, 2000, null);
                //授权
                badzk.addAuthInfo(authentication_type,badAuthentication.getBytes());
                Thread.sleep(2000);
                System.out.println(prefix + "获取数据：" + PATH);
                System.out.println(prefix + "成功获取数据：" + badzk.getData(PATH, false, null));
            } catch (Exception e) {
                System.err.println(prefix + "获取数据失败，原因：" + e.getMessage());
            }
        }

        /** 获取数据：不采用密码 */
        static void getDataByNoAuthentication() {
            String prefix = "[不使用任何授权信息]";
            try {
                System.out.println(prefix + "获取数据：" + PATH);
                ZooKeeper nozk = new ZooKeeper(CONNECT_ADDR, 2000, null);
                Thread.sleep(2000);
                System.out.println(prefix + "成功获取数据：" + nozk.getData(PATH, false, null));
            } catch (Exception e) {
                System.err.println(prefix + "获取数据失败，原因：" + e.getMessage());
            }
        }

        /** 采用正确的密码 */
        static void getDataByCorrectAuthentication() {
            String prefix = "[使用正确的授权信息]";
            try {
                System.out.println(prefix + "获取数据：" + PATH);

                System.out.println(prefix + "成功获取数据：" + zk.getData(PATH, false, null));
            } catch (Exception e) {
                System.out.println(prefix + "获取数据失败，原因：" + e.getMessage());
            }
        }

        /**
         * 更新数据：不采用密码
         */
        static void updateDataByNoAuthentication() {

            String prefix = "[不使用任何授权信息]";

            System.out.println(prefix + "更新数据： " + PATH);
            try {
                ZooKeeper nozk = new ZooKeeper(CONNECT_ADDR, 2000, null);
                Thread.sleep(2000);
                Stat stat = nozk.exists(PATH, false);
                if (stat!=null) {
                    nozk.setData(PATH, prefix.getBytes(), -1);
                    System.out.println(prefix + "更新成功");
                }
            } catch (Exception e) {
                System.err.println(prefix + "更新失败，原因是：" + e.getMessage());
            }
        }

        /**
         * 更新数据：采用错误的密码
         */
        static void updateDataByBadAuthentication() {

            String prefix = "[使用错误的授权信息]";

            System.out.println(prefix + "更新数据：" + PATH);
            try {
                ZooKeeper badzk = new ZooKeeper(CONNECT_ADDR, 2000, null);
                //授权
                badzk.addAuthInfo(authentication_type,badAuthentication.getBytes());
                Thread.sleep(2000);
                Stat stat = badzk.exists(PATH, false);
                if (stat!=null) {
                    badzk.setData(PATH, prefix.getBytes(), -1);
                    System.out.println(prefix + "更新成功");
                }
            } catch (Exception e) {
                System.err.println(prefix + "更新失败，原因是：" + e.getMessage());
            }
        }

        /**
         * 更新数据：采用正确的密码
         */
        static void updateDataByCorrectAuthentication() {

            String prefix = "[使用正确的授权信息]";

            System.out.println(prefix + "更新数据：" + PATH);
            try {
                Stat stat = zk.exists(PATH, false);
                if (stat!=null) {
                    zk.setData(PATH, prefix.getBytes(), -1);
                    System.out.println(prefix + "更新成功");
                }
            } catch (Exception e) {
                System.err.println(prefix + "更新失败，原因是：" + e.getMessage());
            }
        }

        /**
         * 不使用密码 删除节点
         */
        static void deleteNodeByNoAuthentication() throws Exception {

            String prefix = "[不使用任何授权信息]";

            try {
                System.out.println(prefix + "删除节点：" + PATH_DEL);
                ZooKeeper nozk = new ZooKeeper(CONNECT_ADDR, 2000, null);
                Thread.sleep(2000);
                Stat stat = nozk.exists(PATH_DEL, false);
                if (stat!=null) {
                    nozk.delete(PATH_DEL,-1);
                    System.out.println(prefix + "删除成功");
                }
            } catch (Exception e) {
                System.err.println(prefix + "删除失败，原因是：" + e.getMessage());
            }
        }

        /**
         * 采用错误的密码删除节点
         */
        static void deleteNodeByBadAuthentication() throws Exception {

            String prefix = "[使用错误的授权信息]";

            try {
                System.out.println(prefix + "删除节点：" + PATH_DEL);
                ZooKeeper badzk = new ZooKeeper(CONNECT_ADDR, 2000, null);
                //授权
                badzk.addAuthInfo(authentication_type,badAuthentication.getBytes());
                Thread.sleep(2000);
                Stat stat = badzk.exists(PATH_DEL, false);
                if (stat!=null) {
                    badzk.delete(PATH_DEL, -1);
                    System.out.println(prefix + "删除成功");
                }
            } catch (Exception e) {
                System.err.println(prefix + "删除失败，原因是：" + e.getMessage());
            }
        }

        /**
         * 使用正确的密码删除节点
         */
        static void deleteNodeByCorrectAuthentication() throws Exception {

            String prefix = "[使用正确的授权信息]";

            try {
                System.out.println(prefix + "删除节点：" + PATH_DEL);
                Stat stat = zk.exists(PATH_DEL, false);
                if (stat!=null) {
                    zk.delete(PATH_DEL, -1);
                    System.out.println(prefix + "删除成功");
                }
            } catch (Exception e) {
                System.out.println(prefix + "删除失败，原因是：" + e.getMessage());
            }
        }

        /**
         * 使用正确的密码删除节点
         */
        static void deleteParent() throws Exception {
            try {
                Stat stat = zk.exists(PATH_DEL, false);
                if (stat == null) {
                    zk.delete(PATH, -1);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

    }
```











# 六、ZKClientAPI讲解、CuratorAPI讲解、分布式锁讲解、实现多Watcher

## 一、ZKClient的基本使用

```
        public class ZkClientBase {
        /** zookeeper地址 */
        static final String CONNECT_ADDR = "192.168.1.171:2181,192.168.1.172:2181,192.168.1.173:2181";
        /** session超时时间 */
        static final int SESSION_OUTTIME = 5000;//ms 
        public static void main(String[] args) throws Exception {
            ZkClient zkc = new ZkClient(new ZkConnection(CONNECT_ADDR), 5000);
            //1. create and delete方法 
            zkc.createEphemeral("/temp");
            zkc.createPersistent("/super/c1", true);
            Thread.sleep(10000);
            zkc.delete("/temp");
            zkc.deleteRecursive("/super");		
            //2. 设置path和data 并且读取子节点和每个节点的内容
            zkc.createPersistent("/super", "1234");
            zkc.createPersistent("/super/c1", "c1内容");
            zkc.createPersistent("/super/c2", "c2内容");
            List<String> list = zkc.getChildren("/super");
            for(String p : list){
                System.out.println(p);
                String rp = "/super/" + p;
                String data = zkc.readData(rp);
                System.out.println("节点为：" + rp + "，内容为: " + data);
            }
            3. 更新和判断节点是否存在
            zkc.writeData("/super/c1", "新内容");
            System.out.println(zkc.readData("/super/c1"));
            System.out.println(zkc.exists("/super/c1"));

            4.递归删除/super内容
            zkc.deleteRecursive("/super");
        }
    }
```

## 二、zkClient使用 subscribeChildChanges

```
 subscribeChildChanges 方法
 参数一：path 路径
 参数二、时间了IZKChildListener接口的类（如：实例化IZkChildListener类）
 只需要重写其handleChildChange（String parentPath，List<String> currentChilds）
 其中parentPath为最新的子节点列表（相对路径）
 IZkChildListener事件说明针对于下面三个时间触发
 新增子节点、减少子节点、删除节点
```

### 











# 七、Curator框架的使用

## 一、Curator框架使用

```
Curator框架，非常强大，目前已经是Apache的顶级项目，里面提供了更多丰富的操作，session超时重连，主从选举，分布式计数器，分布式锁等适合各种复杂的zookeeper
```

## 二、依赖的引入

```
<!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.0.1</version>
</dependency>
```

## 三、Curator框架中使用链式编程的风格，易读性更强，试用工程方法创建连接对象

```
  1.使用CuratorFameworkFactory的两个静态工厂方法来实现
       参数一：connectString，连接串
       参数二: retryPolicy，重试连接策略，
       参数三 sessionTimeoutMs会话超时，默认60ms
       参数四 connectionTimeout 连接超时时间，默认为15000ms
  2.创建节点create方法，可选链式项
       creatingParentsIfNeeded：是否需要父节点
       withMode： 需要的模式
       forPath：     路径  key value
       withACL：    需要认证
   3.删除节点delete方法，可选择链式项
      deletingChildrenIfNeeded：递归的删除
      graranted：    安全的操作   
      withVersion：   删除版本
      forPath：          
    4.读取和修改数据
        gatData：  读取数据
        setData：   设置数据
    6.异步绑定回调方法，比如节点绑定一个回调函数，该回调函数可以输出服务器的状态码，以及服务的时间的类型，还可以加入一个线程池进行优化操作。
    7.读取子节点方法getChildren  
    8.判断节点是否存在方法checkExists   
```

## 四、代码实现

```
     public class CuratorBase {
    /** zookeeper地址 */
    static final String CONNECT_ADDR = "192.168.1.171:2181,192.168.1.172:2181,192.168.1.173:2181";
    /** session超时时间 */
    static final int SESSION_OUTTIME = 5000;//ms 
    public static void main(String[] args) throws Exception {
        //1 重试策略：初试时间为1s 重试10次
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 10);
        //2 通过工厂创建连接
        CuratorFramework cf = CuratorFrameworkFactory.builder()
                    .connectString(CONNECT_ADDR)
                    .sessionTimeoutMs(SESSION_OUTTIME)
                    .retryPolicy(retryPolicy)
//					.namespace("super")
                    .build();
        //3 开启连接
        cf.start();
//		System.out.println(States.CONNECTED);
//		System.out.println(cf.getState());
        // 新加、删除
        /**
        //4 建立节点 指定节点类型（不加withMode默认为持久类型节点）、路径、数据内容
        cf.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT).forPath("/super/c1","c1内容".getBytes());
        //5 删除节点
        cf.delete().guaranteed().deletingChildrenIfNeeded().forPath("/super");
        */
        // 读取、修改
        /**
        //创建节点
//		cf.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT).forPath("/super/c1","c1内容".getBytes());
//		cf.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT).forPath("/super/c2","c2内容".getBytes());
        //读取节点
//		String ret1 = new String(cf.getData().forPath("/super/c2"));
//		System.out.println(ret1);
        //修改节点
//		cf.setData().forPath("/super/c2", "修改c2内容".getBytes());
//		String ret2 = new String(cf.getData().forPath("/super/c2"));
//		System.out.println(ret2);	
        */
        // 绑定回调函数
        /**
        ExecutorService pool = Executors.newCachedThreadPool();
        cf.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT)
        .inBackground(new BackgroundCallback() {
            @Override
            public void processResult(CuratorFramework cf, CuratorEvent ce) throws Exception {
                System.out.println("code:" + ce.getResultCode());
                System.out.println("type:" + ce.getType());
                System.out.println("线程为:" + Thread.currentThread().getName());
            }
        }, pool)
        .forPath("/super/c3","c3内容".getBytes());
        Thread.sleep(Integer.MAX_VALUE);
        */
        // 读取子节点getChildren方法 和 判断节点是否存在checkExists方法
        /**
        List<String> list = cf.getChildren().forPath("/super");
        for(String p : list){
            System.out.println(p);
        }

        Stat stat = cf.checkExists().forPath("/super/c3");
        System.out.println(stat);
        Thread.sleep(2000);
        cf.delete().guaranteed().deletingChildrenIfNeeded().forPath("/super");
        */
        //cf.delete().guaranteed().deletingChildrenIfNeeded().forPath("/super");
    }
}      
```

### 总结： curator 对节点的简单操作

#### 一、新建

```
cf.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT) .forPath("/super/c1","c1内容".getBytes());
```

#### 二、删除节点

```
cf.delete().guaranteed().deletingChildrenIfNeeded().forPath("/super");
```

### 三、获取子节点

```
cf.getData().forPath("/super/c2")
```

### 四、修改子节点

```
cf.setData().forPath("/super/c2", "修改c2内容".getBytes());
```

### 重点

```
  ExecutorService pool = Executors.newCachedThreadPool();
        cf.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT)
        .inBackground(new BackgroundCallback() {
            @Override
            public void processResult(CuratorFramework cf, CuratorEvent ce) throws Exception {
                System.out.println("code:" + ce.getResultCode());
                System.out.println("type:" + ce.getType());
                System.out.println("线程为:" + Thread.currentThread().getName());
            }
        }, pool)
         为什么采用线程池去承载，是因为在高并发的情况下，我们不能为每一个线程开辟一段空间，我们采用
 线程池去调度。当线程池里面的线程有空闲的时间，我们会调度线程池里面的线程去执行里面的代码。
```

## 五、 Curator 的监听 之一

```
  第一步导入jar包
    <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.0.1</version>
</dependency>
   使用NodeCache的方式去客服端实例中注册一个监听缓存，然后实现对应的监听方法即可，这里我们主要有两种监听方式。
   NodeCacheListener： 监听节点的新增、修改操作
   PathChildrenCacheListener：监听子节点的新增、修改、删除操作
```

#### 加缓存，不是重复注册

```
   public class CuratorWatcher1 {	
/** zookeeper地址 */
static final String CONNECT_ADDR = "192.168.1.171:2181,192.168.1.172:2181,192.168.1.173:2181";
/** session超时时间 */
static final int SESSION_OUTTIME = 5000;//ms 	
public static void main(String[] args) throws Exception {		
	//1 重试策略：初试时间为1s 重试10次
	RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 10);
	//2 通过工厂创建连接
	CuratorFramework cf = CuratorFrameworkFactory.builder()
				.connectString(CONNECT_ADDR)
				.sessionTimeoutMs(SESSION_OUTTIME)
				.retryPolicy(retryPolicy)
				.build();
	
	//3 建立连接
	cf.start();		
	//4 建立一个cache缓存
	final NodeCache cache = new NodeCache(cf, "/super", false);
	cache.start(true);
	cache.getListenable().addListener(new NodeCacheListener() {
		/**
		 * <B>方法名称：</B>nodeChanged<BR>
		 * <B>概要说明：</B>触发事件为创建节点和更新节点，在删除节点的时候并不触发此操作。<BR>
		 * @see org.apache.curator.framework.recipes.cache.NodeCacheListener#nodeChanged()
		 */
		@Override
		public void nodeChanged() throws Exception {
			System.out.println("路径为：" + cache.getCurrentData().getPath());
			System.out.println("数据为：" + new String(cache.getCurrentData().getData()));
			System.out.println("状态为：" + cache.getCurrentData().getStat());
			System.out.println("---------------------------------------");
		}
	});
	Thread.sleep(1000);
	cf.create().forPath("/super", "123".getBytes());		
	Thread.sleep(1000);
	cf.setData().forPath("/super", "456".getBytes())；	
	Thread.sleep(1000);
	cf.delete().forPath("/super");		
	Thread.sleep(Integer.MAX_VALUE);

	}
}
```

## 六、 Curator 的监听 之二

```
public class CuratorWatcher {

    /** zookeeper地址 */
    static final String CONNECT_ADDR = "192.168.1.171:2181,192.168.1.172:2181,192.168.1.173:2181";
    /** session超时时间 */
    static final int SESSION_OUTTIME = 5000;//ms 

    public static void main(String[] args) throws Exception {

        //1 重试策略：初试时间为1s 重试10次
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 10);
        //2 通过工厂创建连接
        CuratorFramework cf = CuratorFrameworkFactory.builder()
                    .connectString(CONNECT_ADDR)
                    .sessionTimeoutMs(SESSION_OUTTIME)
                    .retryPolicy(retryPolicy)
                    .build();

        //3 建立连接
        cf.start();

        //4 建立一个PathChildrenCache缓存,第三个参数为是否接受节点数据内容 如果为false则不接受
        PathChildrenCache cache = new PathChildrenCache(cf, "/super", true);
        //5 在初始化的时候就进行缓存监听
        cache.start(StartMode.POST_INITIALIZED_EVENT);
        cache.getListenable().addListener(new PathChildrenCacheListener() {
            /**
             * <B>方法名称：</B>监听子节点变更<BR>
             * <B>概要说明：</B>新建、修改、删除<BR>
             * @see org.apache.curator.framework.recipes.cache.PathChildrenCacheListener#childEvent(org.apache.curator.framework.CuratorFramework, org.apache.curator.framework.recipes.cache.PathChildrenCacheEvent)
             */
            @Override
            public void childEvent(CuratorFramework cf, PathChildrenCacheEvent event) throws Exception {
                switch (event.getType()) {
                case CHILD_ADDED:
                    System.out.println("CHILD_ADDED :" + event.getData().getPath());
                    break;
                case CHILD_UPDATED:
                    System.out.println("CHILD_UPDATED :" + event.getData().getPath());
                    break;
                case CHILD_REMOVED:
                    System.out.println("CHILD_REMOVED :" + event.getData().getPath());
                    break;
                default:
                    break;
                }
            }
        });

        //创建本身节点不发生变化
        cf.create().forPath("/super", "init".getBytes());

        //添加子节点
        Thread.sleep(1000);
        cf.create().forPath("/super/c1", "c1内容".getBytes());
        Thread.sleep(1000);
        cf.create().forPath("/super/c2", "c2内容".getBytes());

        //修改子节点
        Thread.sleep(1000);
        cf.setData().forPath("/super/c1", "c1更新内容".getBytes());

        //删除子节点
        Thread.sleep(1000);
        cf.delete().forPath("/super/c2");		

        //删除本身节点
        Thread.sleep(1000);
        cf.delete().deletingChildrenIfNeeded().forPath("/super");

        Thread.sleep(Integer.MAX_VALUE);
    }
}
```

## 七、分布式锁

### 在java高并发和多线程中是怎么解决的呢？

```
      在分布式场景中，我们为了保证数据的一致性，经常在程序运行的某一点需要同步操作
    （java 可提供synchronized 或者 Reentrantlock实现）
  
     public class Lock1 {

static ReentrantLock reentrantLock = new ReentrantLock();
static int count = 10;
public static void genarNo(){
	try {
		reentrantLock.lock();
		count--;
		//System.out.println(count);
	} finally {
		reentrantLock.unlock();
	}
}

public static void main(String[] args) throws Exception{
	
	final CountDownLatch countdown = new CountDownLatch(1);
	for(int i = 0; i < 10; i++){
		new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					countdown.await();
					genarNo();
					SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss|SSS");
					System.out.println(sdf.format(new Date()));
					//System.out.println(System.currentTimeMillis());
				} catch (Exception e) {
					e.printStackTrace();
				} finally {
				}
			}
		},"t" + i).start();
	}
	Thread.sleep(50);
	countdown.countDown();

	
	}
}
```

### Zookpeer分布式锁的实现

```
  public class Lock2 {

        /** zookeeper地址 */
        static final String CONNECT_ADDR = "192.168.1.171:2181,192.168.1.172:2181,192.168.1.173:2181";
        /** session超时时间 */
        static final int SESSION_OUTTIME = 5000;//ms 

        static int count = 10;
        public static void genarNo(){
            try {
                count--;
                System.out.println(count);
            } finally {

            }
        }

        public static void main(String[] args) throws Exception {

            //1 重试策略：初试时间为1s 重试10次
            RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 10);
            //2 通过工厂创建连接
            CuratorFramework cf = CuratorFrameworkFactory.builder()
                        .connectString(CONNECT_ADDR)
                        .sessionTimeoutMs(SESSION_OUTTIME)
                        .retryPolicy(retryPolicy)
    //					.namespace("super")
                        .build();
            //3 开启连接
            cf.start();

            //4 分布式锁
            final InterProcessMutex lock = new InterProcessMutex(cf, "/super");
            //final ReentrantLock reentrantLock = new ReentrantLock();
            final CountDownLatch countdown = new CountDownLatch(1);

            for(int i = 0; i < 10; i++){
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            countdown.await();
                            //加锁
                            lock.acquire();
                            //reentrantLock.lock();
                            //-------------业务处理开始
                            //genarNo();
                            SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss|SSS");
                            System.out.println(sdf.format(new Date()));
                            //System.out.println(System.currentTimeMillis());
                            //-------------业务处理结束
                        } catch (Exception e) {
                            e.printStackTrace();
                        } finally {
                            try {
                                //释放
                                lock.release();
                                //reentrantLock.unlock();
                            } catch (Exception e) {
                                e.printStackTrace();
                            }
                        }
                    }
                },"t" + i).start();
            }
            Thread.sleep(100);
            countdown.countDown();

        }
    }
```

## 八、分布式计数器

```
   分布式计数器，在高并发中使用AtomicInteger这种经典的方式、如果对于一个jvm
   的场景当然没有问题，但是我们现在是分布式、就需要利用Curator框架的Distributed
   AtomicInteger了
       public class CuratorAtomicInteger {
    /** zookeeper地址 */
    static final String CONNECT_ADDR = "192.168.1.171:2181,192.168.1.172:2181,192.168.1.173:2181";
    /** session超时时间 */
    static final int SESSION_OUTTIME = 5000;//ms 
    public static void main(String[] args) throws Exception {
        //1 重试策略：初试时间为1s 重试10次
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 10);
        //2 通过工厂创建连接
        CuratorFramework cf = CuratorFrameworkFactory.builder()
                    .connectString(CONNECT_ADDR)
                    .sessionTimeoutMs(SESSION_OUTTIME)
                    .retryPolicy(retryPolicy)
                    .build();
        //3 开启连接
        cf.start();
        //cf.delete().forPath("/super");
        //4 使用DistributedAtomicInteger
        DistributedAtomicInteger atomicIntger = 
                new DistributedAtomicInteger(cf, "/super", new RetryNTimes(3, 1000));
        AtomicValue<Integer> value = atomicIntger.add(1);
        System.out.println(value.succeeded());
        System.out.println(value.postValue());	//最新值
        System.out.println(value.preValue());	//原始值
    }
}
```

## 九、barrier

```
public class CuratorBarrier1 {

        /** zookeeper地址 */
        static final String CONNECT_ADDR = "192.168.1.171:2181,192.168.1.172:2181,192.168.1.173:2181";
        /** session超时时间 */
        static final int SESSION_OUTTIME = 5000;//ms 

        public static void main(String[] args) throws Exception {



            for(int i = 0; i < 5; i++){
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 10);
                            CuratorFramework cf = CuratorFrameworkFactory.builder()
                                        .connectString(CONNECT_ADDR)
                                        .retryPolicy(retryPolicy)
                                        .build();
                            cf.start();

                            DistributedDoubleBarrier barrier = new DistributedDoubleBarrier(cf, "/super", 5);
                            Thread.sleep(1000 * (new Random()).nextInt(3)); 
                            System.out.println(Thread.currentThread().getName() + "已经准备");
                            barrier.enter();
                            System.out.println("同时开始运行...");
                            Thread.sleep(1000 * (new Random()).nextInt(3));
                            System.out.println(Thread.currentThread().getName() + "运行完毕");
                            barrier.leave();
                            System.out.println("同时退出运行...");


                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                },"t" + i).start();
            }



        }
    }






    public class CuratorBarrier2 {

    /** zookeeper地址 */
    static final String CONNECT_ADDR = "192.168.1.171:2181,192.168.1.172:2181,192.168.1.173:2181";
    /** session超时时间 */
    static final int SESSION_OUTTIME = 5000;//ms 

    static DistributedBarrier barrier = null;

    public static void main(String[] args) throws Exception {



        for(int i = 0; i < 5; i++){
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 10);
                        CuratorFramework cf = CuratorFrameworkFactory.builder()
                                    .connectString(CONNECT_ADDR)
                                    .sessionTimeoutMs(SESSION_OUTTIME)
                                    .retryPolicy(retryPolicy)
                                    .build();
                        cf.start();
                        barrier = new DistributedBarrier(cf, "/super");
                        System.out.println(Thread.currentThread().getName() + "设置barrier!");
                        barrier.setBarrier();	//设置
                        barrier.waitOnBarrier();	//等待
                        System.out.println("---------开始执行程序----------");
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            },"t" + i).start();
        }

        Thread.sleep(5000);
        barrier.removeBarrier();	//释放


    }
}
```

## 十、集群管理

```
       public class CuratorWatcher {

        /** 父节点path */
        static final String PARENT_PATH = "/super";

        /** zookeeper服务器地址 */
        public static final String CONNECT_ADDR = "192.168.1.171:2181,192.168.1.172:2181,192.168.1.173:2181";	/** 定义session失效时间 */

        public static final int SESSION_TIMEOUT = 30000;

        public CuratorWatcher() throws Exception{
            //1 重试策略：初试时间为1s 重试10次
            RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 10);
            //2 通过工厂创建连接
            CuratorFramework cf = CuratorFrameworkFactory.builder()
                    .connectString(CONNECT_ADDR)
                    .sessionTimeoutMs(SESSION_TIMEOUT)
                    .retryPolicy(retryPolicy)
                    .build();
            //3 建立连接
            cf.start();

            //4 创建跟节点
            if(cf.checkExists().forPath(PARENT_PATH) == null){
                cf.create().withMode(CreateMode.PERSISTENT).forPath(PARENT_PATH,"super init".getBytes());
            }

            //4 建立一个PathChildrenCache缓存,第三个参数为是否接受节点数据内容 如果为false则不接受
            PathChildrenCache cache = new PathChildrenCache(cf, PARENT_PATH, true);
            //5 在初始化的时候就进行缓存监听
            cache.start(StartMode.POST_INITIALIZED_EVENT);
            cache.getListenable().addListener(new PathChildrenCacheListener() {
                /**
                 * <B>方法名称：</B>监听子节点变更<BR>
                 * <B>概要说明：</B>新建、修改、删除<BR>
                 * @see org.apache.curator.framework.recipes.cache.PathChildrenCacheListener#childEvent(org.apache.curator.framework.CuratorFramework, org.apache.curator.framework.recipes.cache.PathChildrenCacheEvent)
                 */
                @Override
                public void childEvent(CuratorFramework cf, PathChildrenCacheEvent event) throws Exception {
                    switch (event.getType()) {
                    case CHILD_ADDED:
                        System.out.println("CHILD_ADDED :" + event.getData().getPath());
                        System.out.println("CHILD_ADDED :" + new String(event.getData().getData()));
                        break;
                    case CHILD_UPDATED:
                        System.out.println("CHILD_UPDATED :" + event.getData().getPath());
                        System.out.println("CHILD_UPDATED :" + new String(event.getData().getData()));
                        break;
                    case CHILD_REMOVED:
                        System.out.println("CHILD_REMOVED :" + event.getData().getPath());
                        System.out.println("CHILD_REMOVED :" + new String(event.getData().getData()));
                        break;
                    default:
                        break;
                    }
                }
            });
        }

    }
    
    
            public class Client1 {

                    public static void main(String[] args) throws Exception{

                        CuratorWatcher watcher = new CuratorWatcher();
                        Thread.sleep(100000000);
                    }
       }
       
       
        public class Client2 {

            public static void main(String[] args) throws Exception{

                CuratorWatcher watcher = new CuratorWatcher();
                Thread.sleep(100000000);
            }
        }
```