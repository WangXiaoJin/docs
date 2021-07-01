# ZooKeeper注意事项

* `3.4.13`以下版本不建议配置ZooKeeper url为域名，建议用IP，除非你的域名解析非常稳定（基于3.4.8版本测试）

    ZooKeeper首先会把域名解析成对应的IP，解析成功后再持续监听连接状态，当网络断开后恢复时能使ZooKeeper重新恢复连接状态。
    如果网络恢复时，此时ZooKeeper会close之前的连接，重新创建新的连接，在创建新的连接时会继续进行域名解析。如果此时你的
    DNS服务不稳定，解析域名失败，导致创建新的连接失败，固ZooKeeper就不能恢复网络连接，哪怕后面域名解析正常了，也不会恢复。

    > `3.4.13`版本修复此问题

* each znode is usually small, in the byte to kilobyte range.

* The replicated database is an in-memory database containing the entire data tree. Updates are logged to disk for recoverability,
and writes are serialized to disk before they are applied to the in-memory database.
 
* Every ZooKeeper server services clients. Clients connect to exactly one server to submit requests. Read requests are serviced 
from the local replica of each server database. Requests that change the state of the service, write requests, are processed by 
an agreement protocol.

* As part of the agreement protocol all write requests from clients are forwarded to a single server, called the leader. 
The rest of the ZooKeeper servers, called followers, receive message proposals from the leader and agree upon message delivery. 
The messaging layer takes care of replacing leaders on failures and syncing followers with leaders.

* ZooKeeper uses a custom atomic messaging protocol. Since the messaging layer is atomic, ZooKeeper can guarantee that 
the local replicas never diverge. When the leader receives a write request, it calculates what the state of the system
 is when the write is to be applied and transforms this into a transaction that captures this new state.

* 经雅虎团队测试，部署7台Zk时性能最佳，[吞吐量图表](http://zookeeper.apache.org/doc/r3.5.5/zookeeperOver.html#zkPerfRW)

* Paths to nodes are always expressed as canonical, absolute, slash-separated paths; there are no relative reference. 
Any unicode character can be used in a path subject to the following constraints:

    * The null character (\u0000) cannot be part of a path name. (This causes problems with the C binding.)
    * The following characters can't be used because they don't display well, or render in confusing ways: \u0001 - \u001F and \u007F - \u009F.
    * The following characters are not allowed: \ud800 - uF8FF, \uFFF0 - uFFFF.
    * The "." character can be used as part of another name, but "." and ".." cannot alone be used to indicate a node along a path, because ZooKeeper doesn't use relative paths. The following would be invalid: "/a/b/./c" or "/a/b/../c".
    * The token "zookeeper" is reserved.

* **Time in ZooKeeper** - ZooKeeper tracks time multiple ways:

    * `Zxid` - Every change to the ZooKeeper state receives a stamp in the form of a zxid (ZooKeeper Transaction Id). 
    This exposes the total ordering of all changes to ZooKeeper. Each change will have a unique zxid and if zxid1 is smaller 
    than zxid2 then zxid1 happened before zxid2.
    * `Version numbers` - Every change to a node will cause an increase to one of the version numbers of that node. 
    The three version numbers are version (number of changes to the data of a znode), cversion (number of changes to the children of a znode), 
    and aversion (number of changes to the ACL of a znode).
    * `Ticks` - When using multi-server ZooKeeper, servers use ticks to define timing of events such as status uploads, session timeouts, 
    connection timeouts between peers, etc. The tick time is only indirectly exposed through the minimum session timeout (2 times the tick time); 
    if a client requests a session timeout less than the minimum session timeout, the server will tell the client that the session timeout 
    is actually the minimum session timeout.
    * `Real time` - ZooKeeper doesn't use real time, or clock time, at all except to put timestamps into the stat structure on znode creation and znode modification.

* **ZooKeeper Stat Structure** - The Stat structure for each znode in ZooKeeper is made up of the following fields:

    * `czxid` - The zxid of the change that caused this znode to be created.
    * `mzxid` - The zxid of the change that last modified this znode.
    * `pzxid` - The zxid of the change that last modified children of this znode.
    * `ctime` - The time in milliseconds from epoch when this znode was created.
    * `mtime` - The time in milliseconds from epoch when this znode was last modified.
    * `version` - The number of changes to the data of this znode.
    * `cversion` - The number of changes to the children of this znode.
    * `aversion` - The number of changes to the ACL of this znode.
    * `ephemeralOwner` - The session id of the owner of this znode if the znode is an ephemeral node. If it is not an ephemeral node, it will be zero.
    * `dataLength` - The length of the data field of this znode.
    * `numChildren` - The number of children of this znode.

* 从`3.5.0`版本开始，ZooKeeper支持动态配置，这方便了ZooKeeper动态增加、删除Server节点。在`3.5.0`版本之前ZK的各服务间的关系及配置都是静态的，
只能通过重启来加载新的配置。

  `3.5.0`版本之前如果需要动态增加Server节点，只能一个节点一个节点的增加，新节点在所有节点上都增加且重启完成后才能生效，
  且节点的重启是有顺序的，否则会出现服务没Leader或出现脑裂。

  > [ZooKeeper Dynamic Reconfiguration - 官方文档](http://zookeeper.apache.org/doc/r3.5.5/zookeeperReconfig.html)