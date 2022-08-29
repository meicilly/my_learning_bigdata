### Zookeeper系统模型
#### Zookeeper数据模型Znode
- 持久性节点(Persistent):也就是一直存在服务器，直到删除操作主动清除。
- 持久顺序节点(Sequential):就是有顺序的持久节点。
- 临时节点(Ephemeral):就是会自动清理掉的节点，生命周期和客户端会话绑在一起，客户端会话结束，节点就会被删除。临时节点不会有子节点。
- 临时顺序节点。
#### 事务ID
- 是指能够改变Zookeeper服务器状态的操作，我们也称之为事务操作或更新操作，一般包括节点创建与删除、数据节点内容更新操作。对于每一个事务请求，Zookeeper都会为其分配一个全局的事务ID，用ZXID表示，一般是64位的数字。每一个ZXID对应一次更新操作。ZXID可以间接地识别Zookeeper处理这些更新操作请求的全局顺序。
#### ZNode的状态信息
###### ZNode节点内容包括两部分:节点的数据的内容和节点状态信息。
- cZxid就是Create zXID,表示节点被创建时的事务ID。
- ctime就是Create Time,表示节点的创建时间。
- mZxid就是Modified zXID,表示节点最后一次被修改的事务ID。
- mtime就是Modified Time,表示最后一次被修改的时间。
- pZxid表示该节点的子节点列表最后一次被修改的事务ID。只有子节点列表变更才会更新pZxid,子节点内容变更不会更新。
- cversion表示子节点的版本号。
- dataVersion表示内容版本。
- aclVersion表示acl版本。
- ephemeralOwner表示创建该临时节点时的会话sessionID,如果是持久性节点那么值为0。
- dataLength表示数据长度。
- numChildren表示直系子节点数。
#### Watcher-数据变更通知
###### 在Zookeeper中，引入了Watcher机制来实现分布式的通知功能。Zookeeper允许客户端向服务端注册一个Watcher监听，当服务端的一些指定事件触发了这个Watcher,那么就会向指定客户端发送一个事件通知来实现分布式的通知功能。Zookeeper的Watcher机制主要包括客户端线程、客户端WatcherManager、Zookeeper服务器三部分。![img.png](img/Watcher的流程图.png)
#### ACL-保障数据的安全
###### Zookeeper的内部存储了分布式系统运行时状态的元数据，这些元数据会直接影响基于Zookeeper进行构造的分布式系统运行状态。
###### 三方面类理解ACL机制:权限模式(Scheme)、授权对象(ID)、权限(Permission),通常使用"scheme:id:permission"来标识一个有效的ACL信息。
- IP模式:就是通过IP地址粒度来进行权限控制。IP模式可以支持按照网段方式进行配置。
- Digest模式:最常用的权限控制模式，要更符合我们对权限控制的认识
- World模式:是一种开放的模式，用户可以不在进行任何权限校验的情况下操作Zookeeper上的数据。
- Super模式:就是一种特殊的Digest模式。在Super模式下，超级用户可以对任意Zookeeper上的数据节点进行任何操作。
#### Zookeeper的命令行操作
```shell
#1.创建顺序节点
create -s /zk-demo 123
#2.创建临时节点
create -e /zk-e 123
#3.创建永久节点
create /zk-permanent 123
#4.更新节点
set /zk-demo 222
```
#### Zookeeper的api的使用
###### 建立会话的操作
```java
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;

import java.io.IOException;
import java.util.concurrent.CountDownLatch;

public class CreateSession implements Watcher {
    private static CountDownLatch countDownLatch = new CountDownLatch(1);
    public static void main(String[] args) throws IOException, InterruptedException {
        /*
            客户端可以通过创建⼀个zk实例来连接zk服务器
            new Zookeeper(connectString,sesssionTimeOut,Wather)
            connectString: 连接地址：IP：端⼝
            sesssionTimeOut：会话超时时间：单位毫秒
            Wather：监听器(当特定事件触发监听时，zk会通过watcher通知到客户端)
        */
        ZooKeeper zooKeeper = new ZooKeeper("192.168.233.17:2181", 5000, new CreateSession());
        // 获取连接zookeeper状态
        System.out.println(zooKeeper.getState());
        // 计数工具类：CountDownLatch:不让main方法结束，让线程等待
        countDownLatch.await();
        System.out.println("客户端会话建立了");
    }

    /*
    * 回调方法：处理来自服务器端的watcher通知
    * */
    @Override
    public void process(WatchedEvent watchedEvent) {
        // SyncConnected 表示会话创建成功
        if(watchedEvent.getState()== Event.KeeperState.SyncConnected){
            // 解除主程序在CountDownLatch上的等待阻塞
            System.out.println("countDownLatch方法执行了");
            countDownLatch.countDown();
        }
    }
}
```

###### 创建节点的操作
```java
import org.apache.zookeeper.*;
import java.io.IOException;
import java.util.concurrent.CountDownLatch;
public class CreateNode implements Watcher{
    private static CountDownLatch countDownLatch = new CountDownLatch(1);
    private static ZooKeeper zooKeeper;
    public static void main(String[] args) throws IOException, InterruptedException {
        /*
            客户端可以通过创建⼀个zk实例来连接zk服务器
            new Zookeeper(connectString,sesssionTimeOut,Wather)
            connectString: 连接地址：IP：端⼝
            sesssionTimeOut：会话超时时间：单位毫秒
            Wather：监听器(当特定事件触发监听时，zk会通过watcher通知到客户端)
        */
        zooKeeper = new ZooKeeper("192.168.233.17:2181,192.168.233.16:2181,192.168.233.18:2181", 5000, new CreateNode());
        // 获取连接zookeeper状态
        System.out.println(zooKeeper.getState());
        // 计数工具类：CountDownLatch:不让main方法结束，让线程等待
        //countDownLatch.await();
        Thread.sleep(Integer.MAX_VALUE);
    }

    /*
     * 回调方法：处理来自服务器端的watcher通知
     * */
    @Override
    public void process(WatchedEvent watchedEvent) {
        // SyncConnected 表示会话创建成功
        if(watchedEvent.getState()== Event.KeeperState.SyncConnected){
            // 解除主程序在CountDownLatch上的等待阻塞
            System.out.println("countDownLatch方法执行了");
            //countDownLatch.countDown();
            // 创建节点 同步的方式创建节点
            try {
                createNoteSync();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (KeeperException e) {
                e.printStackTrace();
            }
        }
    }
    /*
     * 创建节点的方法
     * */
    public void createNoteSync() throws InterruptedException, KeeperException {
        /**
         * path ：节点创建的路径
         * data[] ：节点创建要保存的数据，是个byte类型的
         * acl ：节点创建的权限信息(4种类型)
         * ANYONE_ID_UNSAFE : 表示任何⼈
         * AUTH_IDS ：此ID仅可⽤于设置ACL。它将被客户机验证的ID替
         换。
         * OPEN_ACL_UNSAFE ：这是⼀个完全开放的ACL(常⽤)-->
         world:anyone
         * CREATOR_ALL_ACL ：此ACL授予创建者身份验证ID的所有权限
         * createMode ：创建节点的类型(4种类型)
         * PERSISTENT：持久节点
         * PERSISTENT_SEQUENTIAL：持久顺序节点
         * EPHEMERAL：临时节点
         * EPHEMERAL_SEQUENTIAL：临时顺序节点
         String node = zookeeper.create(path,data,acl,createMode);
         */
        System.out.println("创建节点");
        // 持久节点
        String note_persistent = zooKeeper.create("/meicilly-persistent", "持久节点内容".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        // 创建临时节点
        String note_ephemeral = zooKeeper.create("/meicilly-ephemeral", "临时节点".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
        // 持久顺序节点
        String note_sequential = zooKeeper.create("/meicilly-sequential", "持久顺序节点".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL);
        System.out.println("创建的持久节点" + note_persistent);
        System.out.println("创建临时节点" + note_ephemeral);
        System.out.println("持久顺序节点" + note_sequential);
    }
}
```
###### 获取节点的数据
```
import org.apache.zookeeper.*;
import java.io.IOException;
import java.util.List;
import java.util.concurrent.CountDownLatch;

public class GetNoteData implements Watcher{
    private static CountDownLatch countDownLatch = new CountDownLatch(1);
    private static ZooKeeper zooKeeper;
    public static void main(String[] args) throws IOException, InterruptedException {
        /*
            客户端可以通过创建⼀个zk实例来连接zk服务器
            new Zookeeper(connectString,sesssionTimeOut,Wather)
            connectString: 连接地址：IP：端⼝
            sesssionTimeOut：会话超时时间：单位毫秒
            Wather：监听器(当特定事件触发监听时，zk会通过watcher通知到客户端)
        */
        zooKeeper = new ZooKeeper("192.168.233.17:2181,192.168.233.16:2181,192.168.233.18:2181", 5000, new GetNoteData());
        // 获取连接zookeeper状态
        System.out.println(zooKeeper.getState());
        // 计数工具类：CountDownLatch:不让main方法结束，让线程等待
        //countDownLatch.await();
        Thread.sleep(Integer.MAX_VALUE);
    }

    /*
     * 回调方法：处理来自服务器端的watcher通知
     * */
    @Override
    public void process(WatchedEvent watchedEvent) {
        // 当子节点列表发生改变时，服务器端会发生noteChildrenChanged事件通知 要重新获取子节点列表 需要反复注册监听
        if(watchedEvent.getType() == Event.EventType.NodeChildrenChanged){
            List<String> children = null;
            try {
                children = zooKeeper.getChildren("/meicilly-persistent", true);
            } catch (KeeperException e) {
                e.printStackTrace();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(children);
        }
        // SyncConnected 表示会话创建成功
        if(watchedEvent.getState()== Watcher.Event.KeeperState.SyncConnected){
            // 解除主程序在CountDownLatch上的等待阻塞
            System.out.println("countDownLatch方法执行了");
            //countDownLatch.countDown();
            // 获取节点的方法
            try {
                getNoteData();
                // 获取节点的子节点列表
                //getChildrens();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (KeeperException e) {
                e.printStackTrace();
            }

        }
    }

    private void getNoteData() throws InterruptedException, KeeperException {
        /**
         * path : 获取数据的路径
         * watch : 是否开启监听
         * stat : 节点状态信息
         * null: 表示获取最新版本的数据
         * zk.getData(path, watch, stat);
         */
        byte[] data = zooKeeper.getData("/meicilly-persistent", false, null);
        System.out.println(new String(data));
    }
    /*
    * 获取某个节点的子节点列表的方法
    * */
    public static void getChildrens() throws InterruptedException, KeeperException {
        /*
            path:路径
            watch:是否要启动监听，当⼦节点列表发⽣变化，会触发监听
            zooKeeper.getChildren(path, watch);
        */
        List<String> children = zooKeeper.getChildren("/meicilly-persistent", true);
        System.out.println(children);
    }

}
```
###### 更新某个节点的数据
```
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.data.Stat;
import java.io.IOException;
import java.util.concurrent.CountDownLatch;

public class UpdateNoteData implements Watcher{
    private static CountDownLatch countDownLatch = new CountDownLatch(1);
    private static ZooKeeper zooKeeper;
    public static void main(String[] args) throws IOException, InterruptedException {
        /*
            客户端可以通过创建⼀个zk实例来连接zk服务器
            new Zookeeper(connectString,sesssionTimeOut,Wather)
            connectString: 连接地址：IP：端⼝
            sesssionTimeOut：会话超时时间：单位毫秒
            Wather：监听器(当特定事件触发监听时，zk会通过watcher通知到客户端)
        */
        zooKeeper = new ZooKeeper("192.168.233.17:2181,192.168.233.16:2181,192.168.233.18:2181", 5000, new UpdateNoteData());
        // 获取连接zookeeper状态
        System.out.println(zooKeeper.getState());
        // 计数工具类：CountDownLatch:不让main方法结束，让线程等待
        //countDownLatch.await();
        Thread.sleep(Integer.MAX_VALUE);
    }

    /*
     * 回调方法：处理来自服务器端的watcher通知
     * */
    @Override
    public void process(WatchedEvent watchedEvent) {
        // SyncConnected 表示会话创建成功
        if(watchedEvent.getState()== Event.KeeperState.SyncConnected){
            // 解除主程序在CountDownLatch上的等待阻塞
            System.out.println("countDownLatch方法执行了");
            //countDownLatch.countDown();
            //更新数据节点的方法
            try {
                updateNoteSync();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (KeeperException e) {
                e.printStackTrace();
            }


        }
    }
    // 更新数据节点内容的方法
    private void updateNoteSync() throws InterruptedException, KeeperException {
        /*
            path:路径
            data:要修改的内容 byte[]
            version:为-1，表示对最新版本的数据进⾏修改
            zooKeeper.setData(path, data,version);
        */
        byte[] data1 = zooKeeper.getData("/meicilly-persistent", false, null);
        System.out.println("修改前的值" + new String(data1));
        // 修改meicilly-persistent的数据
        Stat stat = zooKeeper.setData("/meicilly-persistent", "客户端修改了节点数".getBytes(), -1);
        byte[] data2 = zooKeeper.getData("/meicilly-persistent", false, null);
        System.out.println("修改后的值" + new String(data2));
    }
}
```
###### 删除某个节点的数据
```
import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;
import java.io.IOException;
import java.util.concurrent.CountDownLatch;
public class DeleteNode implements Watcher{
    private static CountDownLatch countDownLatch = new CountDownLatch(1);
    private static ZooKeeper zooKeeper;
    public static void main(String[] args) throws IOException, InterruptedException {
        /*
            客户端可以通过创建⼀个zk实例来连接zk服务器
            new Zookeeper(connectString,sesssionTimeOut,Wather)
            connectString: 连接地址：IP：端⼝
            sesssionTimeOut：会话超时时间：单位毫秒
            Wather：监听器(当特定事件触发监听时，zk会通过watcher通知到客户端)
        */
        zooKeeper = new ZooKeeper("192.168.233.17:2181,192.168.233.16:2181,192.168.233.18:2181", 5000, new DeleteNode());
        // 获取连接zookeeper状态
        System.out.println(zooKeeper.getState());
        // 计数工具类：CountDownLatch:不让main方法结束，让线程等待
        //countDownLatch.await();
        Thread.sleep(Integer.MAX_VALUE);
    }

    /*
     * 回调方法：处理来自服务器端的watcher通知
     * */
    @Override
    public void process(WatchedEvent watchedEvent) {
        // SyncConnected 表示会话创建成功
        if(watchedEvent.getState()== Event.KeeperState.SyncConnected){
            // 解除主程序在CountDownLatch上的等待阻塞
            System.out.println("countDownLatch方法执行了");
            //countDownLatch.countDown();
            // 删除节点
            try {
                deleteNoteSync();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (KeeperException e) {
                e.printStackTrace();
            }
        }
    }
    /*
    * 删除节点的方法
    * */
    private void deleteNoteSync() throws InterruptedException, KeeperException {
        /*
            zooKeeper.exists(path,watch) :判断节点是否存在
            zookeeper.delete(path,version) : 删除节点
         */
        Stat stat = zooKeeper.exists("/meicilly-persistent/c1", false);
        System.out.println(stat == null ? "该节点不存在":"该节点存在");
        if(stat != null){
            zooKeeper.delete("/meicilly-persistent/c1",-1);
        }
    }
}
```