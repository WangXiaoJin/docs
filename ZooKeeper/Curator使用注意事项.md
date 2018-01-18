## Curator使用注意事项

1. 分布式锁使用注意事项
    
    Curator中`所有`分布式锁（例：`InterProcessMutex`/`InterProcessSemaphoreMutex`等）不能在`Watcher`/`CuratorWatcher`
    里面使用。因为分布式锁自身也会用到`watcher`，根据`watcher`来激活线程并获取锁。默认情况下执行`watcher`是单线程的，
    所以分布式锁自身的`watcher`要等到自有程序中的`watcher`执行完成后才能执行，而自有线程中`watcher`一直处于获取锁的状态，
    最终导致出现死锁现象。
    
    源码：
    ```java
    while ( (client.getState() == CuratorFrameworkState.STARTED) && !haveTheLock )
    {
        List<String>        children = getSortedChildren();
        String              sequenceNodeName = ourPath.substring(basePath.length() + 1); // +1 to include the slash
    
        PredicateResults    predicateResults = driver.getsTheLock(client, children, sequenceNodeName, maxLeases);
        if ( predicateResults.getsTheLock() )
        {
            haveTheLock = true;
        }
        else
        {
            String  previousSequencePath = basePath + "/" + predicateResults.getPathToWatch();
    
            synchronized(this)
            {
                try 
                {
                    // use getData() instead of exists() to avoid leaving unneeded watchers which is a type of resource leak
                    //【注意】此处用到了watcher
                    client.getData().usingWatcher(watcher).forPath(previousSequencePath);
                    if ( millisToWait != null )
                    {
                        millisToWait -= (System.currentTimeMillis() - startMillis);
                        startMillis = System.currentTimeMillis();
                        if ( millisToWait <= 0 )
                        {
                            doDelete = true;    // timed out - delete our node
                            break;
                        }
    
                        wait(millisToWait);
                    }
                    else
                    {
                        wait();
                    }
                }
                catch ( KeeperException.NoNodeException e ) 
                {
                    // it has been deleted (i.e. lock released). Try to acquire again
                }
            }
        }
    }
    ```
    
    `错误`示例代码，当有节点变化时，你会发现线程被阻塞，出现死锁现象：
    ```java
    public class Test {
        
        public static void main(String[] args) {
            for (int i = 0; i < 5; i++) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        final CuratorFramework client = CuratorFrameworkFactory.builder()
                                .connectString("192.168.200.126:2181")
                                .retryPolicy(new ExponentialBackoffRetry(1000, 3))
                                .build();
                        client.start();
                        final InterProcessMutex lock = new InterProcessMutex(client, "/locks");
                        CuratorWatcher watcher = new CuratorWatcher() {
                            @Override
                            public void process(WatchedEvent event) throws Exception {
                                if (event.getState() == KeeperState.SyncConnected
                                        && event.getType() == EventType.NodeChildrenChanged) {
                                    System.out.println(Thread.currentThread().getName() + " : 获取 ZK-LOCK ING.");
                                    lock.acquire();
                                    System.out.println(Thread.currentThread().getName() + " : 获取 ZK-LOCK SUC.");
                                    children(client, this);
                                    lock.release();
                                    System.out.println(Thread.currentThread().getName() + " : 释放 ZK-LOCK.");
                                }
                            }
                        };
                        children(client, watcher);
                    }
                }).start();
            }
            
            try {
                int read = System.in.read();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        
        public static void children(CuratorFramework client, CuratorWatcher watcher) {
            try {
                List<String> strings = client.getChildren().usingWatcher(watcher).forPath("/nodes");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    ```
    
    > It is strongly recommended that you add a ConnectionStateListener and watch for SUSPENDED and LOST 
    state changes. If a SUSPENDED state is reported you cannot be certain that you still hold the lock 
    unless you subsequently receive a RECONNECTED state. If a LOST state is reported it is certain that 
    you no longer hold the lock.

2. 注册`watcher`

    * 相同的watcher对象注册多次时，只会执行一次
    
    * 当连接状态从`LOST`恢复至`RECONNECTED`时，之前注册过的`watcher`会无效，需要重新注册。所以这种情况下，
    始终都会先执行`ConnectionStateListener`，在`ConnectionStateListener`中绑定`watcher`，
    然后再触发`watcher`事件
    
    * 当连接状态从`SUSPENDED`恢复至`RECONNECTED`时，之前注册过的`watcher`持续有效，不需要重新注册

3. `LeaderLatch`注意事项

    * 在一个全新leader节点下启动项目时，`LeaderLatch` bean创建完成，并不会立即选举出leader。因为选举需要一段时间，
    在这段时间差内是没有leader的，如果项目启动时你需要leader来做一些逻辑时，你必须在项目启动时主动等待leader选举完成后，
    再判断当前应用是不是leader，如果是leader再执行对应的逻辑。等待选举leader的简单示例代码如下（**建议使用`LeaderSelector`实现**）：
    
        ```java
        private void waitElectLeader() throws Exception {
            log.info("LeaderLatch state : " + leaderLatch.getState());
            while (!Thread.currentThread().isInterrupted()
                    && leaderLatch.getState() != State.CLOSED
                    && !leaderLatch.getLeader().isLeader()) {
                try {
                    log.info("Waiting for elect leader.....");
                    TimeUnit.MILLISECONDS.sleep(1000);
                } catch (InterruptedException e) {
                    log.error("waitElectLeader sleep InterruptedException.", e);
                    Thread.currentThread().interrupt();
                }
            }
        }
        ```
        > 注：`leaderLatch.getLeader().isLeader()`实时发请求，查询leader是否已经选举成功  
        
    * 在`ConnectionStateListener`中使用`LeaderLatch`
    
        `ConnectionStateListener`默认使用单线程执行。LeaderLatch会注册ConnectionStateListener监听器，监听连接状态，
        当连接状态发生变化时，做出相应措施（如：重新创建节点、删除已有节点）。当你在`ConnectionStateListener`使用
        `LeaderLatch`时且用默认线程注册此`ConnectionStateListener`时，会有一个有趣的现象：当此`ConnectionStateListener`
        需要等待`LeaderLatch`选举完成之后才能执行一些逻辑，没选举成功则一直等待。如果此时断网一段时间后重连，
        `LeaderLatch`和自有的`ConnectionStateListener`都将会被触发执行，由于`ConnectionStateListener`此时是
        单线程执行，且如果自有`ConnectionStateListener`先被执行（执行顺序与注册顺序无关，因底层是使用ConcurrentHashMap存储的），
        自有`ConnectionStateListener`会一直等待Leader选举成功，而LeaderLatch`的`ConnectionStateListener`需要等待自
        有`ConnectionStateListener`执行完成后才能触发选举leader，此时恭喜你的程序将会被卡住。
        
        如遇上述情况时只能把自有`ConnectionStateListener`程序里的代码写成异步或者在注册自有`ConnectionStateListener`
        使用`Executor`。

    * `leaderLatch.hasLeadership()`和`leaderLatch.getLeader()`不一定相等
    
        `leaderLatch.getLeader()`为实时去ZK获取最新数据，`leaderLatch.hasLeadership()`只有在`leaderLatch`执行完
        自身`watcher`逻辑后才会同步Leader状态。

    * 在`watcher`中使用`LeaderLatch`
    
        因为`LeaderLatch`使用`watcher`监听leader节点变化，监控的节点有变化后，再获取最新的leader节点。如果最新的leader
        节点就是本身，则会把自身标记为leader。由于默认情况下`watcher`使用单线程执行，所以当有`watcher`没跑完，想在此`watcher`中验证是否是leader时，
        就算此时自身节点在ZooKeeper中是leader，通过`leaderLatch.hasLeadership()`返回是`false`，因为只有等到当前`watcher`
        跑完后才有机会执行`LeaderLatch`的`watcher`来更新本地缓存数据。这也验证了：`leaderLatch.hasLeadership()`和
        `leaderLatch.getLeader()`不一定相等
        
4. 总结

    * 不能在`Watcher/CuratorWatcher`中使用`InterProcessMutex/InterProcessSemaphoreMutex`等分布式锁
    * 不能在`Watcher/CuratorWatcher`中使用`LeaderLatch`，同样适用于`PathChildrenCache/NodeCache/TreeCache`等，因其内部
    还是使用`Watcher/CuratorWatcher`来操作。否则在网络急剧抖动时，会出现leader切换，导致部署数据丢失或更新换乱。
    
        这种场景下建议使用`LeaderSelector`，但`LeaderSelector`存在两个不足之处：
        1. 在获取Leader权限后，当执行完`LeaderSelectorListener#takeLeadership()`逻辑就会释放Leader权限，虽然可以通过
        `autoRequeue`来再次排队获取Leader权限，但这种切换Leader这么频繁的方式不能覆盖所以场景。我需要的是当此应用获得
        Leader权限后，就一直占有，除非自己手动释放或网络断掉才释放Leader权限。
        2. `LeaderSelector`只能传递一个`LeaderSelectorListener`，意味着我每次使用都需要创建一个`LeaderSelector`,这样会
        无缘无故多出很多的`LeaderSelector`监控不同的节点，即浪费资源也增加开发难度。

        针对上述两个不足之处，我扩展了`LeaderSelector`类，实现了相关功能：
        ```java
        import com.google.common.base.Preconditions;
        import org.apache.curator.framework.CuratorFramework;
        import org.apache.curator.framework.recipes.leader.*;
        import org.apache.curator.framework.state.ConnectionState;
        import org.apache.curator.utils.ThreadUtils;
        import org.slf4j.Logger;
        import org.slf4j.LoggerFactory;
        
        import java.io.Closeable;
        import java.io.IOException;
        import java.util.List;
        import java.util.concurrent.*;
        import java.util.concurrent.atomic.AtomicReference;
        
        /**
         * 此类对{@link LeaderSelector}做了封装，使用起来更方便，功能区别如下：<br/>
         * 1. 原有{@link LeaderSelectorListener#takeLeadership(CuratorFramework)}执行完后会自动回收leader权限。
         * 而此类会持续占有leader权限，除非连接状态变更为{@link ConnectionState#SUSPENDED}或{@link ConnectionState#LOST}时，
         * 会释放leader权限，释放leader权限后可通过{@link #autoRequeue()}配置使其自动排队重新获取leader，
         * 否则永远都不会再获取leader了。<br/>
         * 2. {@link LeaderSelector}类只能传递一个{@link LeaderSelectorListener}监听器，而此类可传递多个{@link LeaderLifecycleListener}，
         * 方便多处逻辑共享一个leader
         * @author WangXiaoJin
         */
        public class LastingLeaderSelector implements Closeable {
            
            private final Logger log = LoggerFactory.getLogger(getClass());
            
            private LeaderSelector leaderSelector;
            private ExecutorService executorService;
            
            private boolean shutdownExecutorOnClose = true;
            
            private static final ThreadFactory defaultThreadFactory = ThreadUtils.newThreadFactory("LeaderLifecycleListener");
            
            public LastingLeaderSelector(CuratorFramework client, String leaderPath, List<LeaderLifecycleListener> listeners) {
                this(client, leaderPath, listeners, false);
            }
            
            public LastingLeaderSelector(CuratorFramework client, String leaderPath, List<LeaderLifecycleListener> listeners, boolean cancelled) {
                this(client, leaderPath, null, Executors.newSingleThreadExecutor(defaultThreadFactory), listeners, cancelled);
            }
            
            public LastingLeaderSelector(CuratorFramework client, String leaderPath, ExecutorService executorService, List<LeaderLifecycleListener> listeners) {
                this(client, leaderPath, null, executorService, listeners, false);
            }
            
            /**
             * @param client
             * @param leaderPath
             * @param leaderId
             * @param executorService   用于执行listeners
             * @param listeners leader状态监听器
             * @param cancelled 当连接状态为{@link ConnectionState#SUSPENDED}或{@link ConnectionState#LOST}时，
             *                  取消当前Leadership，中断当前正在执行的任务
             */
            public LastingLeaderSelector(CuratorFramework client, String leaderPath, String leaderId,
                                         ExecutorService executorService, List<LeaderLifecycleListener> listeners, boolean cancelled) {
                Preconditions.checkNotNull(executorService, "executorService cannot be null");
                this.executorService = executorService;
                leaderSelector = new LeaderSelector(
                        client,
                        leaderPath,
                        new LastingLeaderSelectorListener(executorService, listeners, cancelled)
                );
                if (leaderId != null) {
                    leaderSelector.setId(leaderId);
                }
            }
            
            /**
             * 释放leader权限后自动排队再次获取leader权限
             */
            public void autoRequeue() {
                leaderSelector.autoRequeue();
                log.debug("AutoRequeue leaderSelector.");
            }
            
            public void start() {
                leaderSelector.start();
                log.debug("Start leaderSelector.");
            }
            
            @Override
            public void close() throws IOException {
                leaderSelector.close();
                log.debug("Close leaderSelector.");
                if (shutdownExecutorOnClose) {
                    executorService.shutdown();
                    log.debug("Shutdown executorService.");
                }
            }
            
            public LeaderSelector getLeaderSelector() {
                return leaderSelector;
            }
            
            public boolean isShutdownExecutorOnClose() {
                return shutdownExecutorOnClose;
            }
            
            public void setShutdownExecutorOnClose(boolean shutdownExecutorOnClose) {
                this.shutdownExecutorOnClose = shutdownExecutorOnClose;
            }
            
            /**
             * 一旦获得leader之后就不会释放leader权限，除非连接状态变成{@link ConnectionState#SUSPENDED}或{@link ConnectionState#LOST}
             */
            class LastingLeaderSelectorListener implements LeaderSelectorListener {
            
                private final Logger log = LoggerFactory.getLogger(getClass());
                
                private volatile boolean running = true;
                
                /**
                 * Leader状态（获取/释放）的监听器
                 */
                private List<LeaderLifecycleListener> listeners;
                
                private ExecutorService executorService;
                
                /**
                 * 当连接状态为{@link ConnectionState#SUSPENDED}或{@link ConnectionState#LOST}时，取消当前Leadership，
                 * 中断当前正在执行的任务
                 */
                private boolean cancelled;
                
                public LastingLeaderSelectorListener(ExecutorService executorService, List<LeaderLifecycleListener> listeners, boolean cancelled) {
                    Preconditions.checkNotNull(executorService, "executorService cannot be null");
                    Preconditions.checkNotNull(listeners, "listeners cannot be null");
                    this.executorService = executorService;
                    this.listeners = listeners;
                }
                
                @Override
                public void takeLeadership(CuratorFramework client) throws Exception {
                    log.debug("Executing LastingLeaderSelectorListener.takeLeadership().");
                    for (final LeaderLifecycleListener listener : listeners) {
                        executorService.submit(new Callable<Void>() {
                            @Override
                            public Void call() throws Exception {
                                listener.take();
                                return null;
                            }
                        });
                    }
                    synchronized (this) {
                        while (running) {
                            try {
                                log.debug("holding leader...");
                                this.wait();
                            } catch (Throwable e) {
                                ThreadUtils.checkInterrupted(e);
                            }
                        }
                    }
                    log.debug("Executed LastingLeaderSelectorListener.takeLeadership() end. Release leader.");
                }
                
                @Override
                public void stateChanged(CuratorFramework client, ConnectionState newState) {
                    if (newState == ConnectionState.CONNECTED || newState == ConnectionState.RECONNECTED) {
                        synchronized (this) {
                            log.debug("Reset leader listener state.");
                            running = true;
                            for (LeaderLifecycleListener listener : listeners) {
                                listener.resetState();
                            }
                        }
                    } else if (newState == ConnectionState.SUSPENDED || newState == ConnectionState.LOST) {
                        log.debug("Executing LastingLeaderSelectorListener.stateChanged(). release-leader begin.");
                        for (final LeaderLifecycleListener listener : listeners) {
                            executorService.submit(new Callable<Void>() {
                                @Override
                                public Void call() throws Exception {
                                    listener.release();
                                    return null;
                                }
                            });
                        }
                        synchronized (this) {
                            running = false;
                            log.debug("Release the holding leader.");
                            this.notifyAll();
                        }
                        if (cancelled) {
                            log.debug("Cancel the leadership.Interrupt the inner task.");
                            throw new CancelLeadershipException();
                        }
                    }
                }
                
            }
            
            private enum State {
                LATENT,
                TAKED,
                RELEASED
            }
            
            /**
             * Leader状态（获取/释放）的监听器
             */
            public static abstract class LeaderLifecycleListener {
            
                private final Logger log = LoggerFactory.getLogger(getClass());
                
                private final AtomicReference<State> state = new AtomicReference<>(State.LATENT);
                
                public void resetState() {
                    state.set(State.LATENT);
                }
                
                /**
                 * 获得leader时执行的逻辑
                 */
                public synchronized void take() throws Exception {
                    log.debug("Trigger LeaderLifecycleListener take event. Current state : {}", state.get());
                    Preconditions.checkState(state.compareAndSet(State.LATENT, State.TAKED),
                            "Cannot take leadership, because of current state is" + state.get());
                    log.debug("Executing LeaderLifecycleListener.doTake().");
                    doTake();
                    log.debug("Executed LeaderLifecycleListener.doTake() end.");
                }
                
                /**
                 * 获得leader时执行的逻辑
                 * @throws Exception
                 */
                protected abstract void doTake() throws Exception;
                
                
                /**
                 * 释放leader时执行的逻辑
                 */
                public synchronized void release() throws Exception {
                    log.debug("Trigger LeaderLifecycleListener release event. Current state : {}", state.get());
                    State preState = state.getAndSet(State.RELEASED);
                    if (preState == State.TAKED) {
                        log.debug("Executing LeaderLifecycleListener.doRelease().");
                        doRelease();
                        log.debug("Executed LeaderLifecycleListener.doRelease() end.");
                    }
                }
                
                /**
                 * 释放leader时执行的逻辑
                 * @throws Exception
                 */
                protected abstract void doRelease() throws Exception;
                
            }
        }
        ```
        
        Demo：
        ```java
        final CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString("192.168.200.126:2181")
                .retryPolicy(new ExponentialBackoffRetry(1000, 3))
                .build();
        client.start();
        
        List<LeaderLifecycleListener> listeners = new ArrayList<>();
        listeners.add(new LeaderLifecycleListener() {
            
            private PathChildrenCache cache;
            
            @Override
            protected void doTake() throws Exception {
                cache = new PathChildrenCache(client, "/svc-node", false);
                cache.getListenable().addListener(new PathChildrenCacheListener() {
                    @Override
                    public void childEvent(CuratorFramework client, PathChildrenCacheEvent event) throws Exception {
                        if (event.getType() == Type.CHILD_ADDED || event.getType() == Type.CHILD_REMOVED) {
                            switch (event.getType()) {
                                case CHILD_ADDED:
                                    log.info("【EXECUTE】 : Add({})", event.getData().getPath());
                                    break;
                                case CHILD_REMOVED:
                                    log.info("【EXECUTE】 : Remove({})", event.getData().getPath());
                                    break;
                            }
                        }
                    }
                });
                cache.start();
            }
            
            @Override
            protected void doRelease() throws Exception {
                cache.close();
            }
        });
        
        LastingLeaderSelector leaderSelector = new LastingLeaderSelector(client, "/leader", listeners);
        leaderSelector.autoRequeue();
        leaderSelector.start();
        ```