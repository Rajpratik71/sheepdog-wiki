Sheepdog uses a cluster management backend to manage
membership and broadcast messages to the cluster nodes.

For now, sheepdog can use local driver (for development on a single box), [corosync](http://corosync.org/doku.php) (the default), [zookeeper](http://zookeeper.apache.org/) and
[Accord](http://www.osrg.net/accord/).  Corosync is not recommended for production use as the method used for synchronization is susceptible to packet loss under load.  For reliable operation, zookeeper is highly recommended.

# Local Driver
This driver just makes use of Unix IPC mechanism to management the membership on a single box, where we start multiple 'sheep' processes to simulate the cluster. It is very easy and fast setup and especially useful to test functionality and for developer without involving any other software.

To set up a 3 node using local driver in one liner bash with debug mode (mkdir /path/to/store first):

    $ for i in 0 1 2; do sheep -c local -d /path/to/store/$i -z $i -p 700$i;sleep 1;done

# Corosync

Corosync runs on every node as a daemon.

Corosync can only work reliably with less than 15 nodes,
due to its implementation and design goal for small sized cluster.

# Zookeeper

Zookeeper runs as a standalone cluster. This means you need to set up zookeeper cluster
first, then pass the IP:PORT list to sheep start-up option. For most of use cases, zookeeper
cluster of 3~5 nodes will be good enough to driver hundreds of sheep daemons. 

If you want to add more nodes into sheepdog cluster, you should run
sheepdog against zookeeper. Some users (eg. at Taobao.com) have been working with
the scalability of the sheepdog and currently running with around 1000
nodes in their test environment. For several month expediencies with
zookeeper, they have found that it works well for node number below 1000,
with object cache enabled.

To enable zookeeper support (Suppose we have a zookeeper cluster with 3 nodes):

## Install zookeeper (by source tarball)
Fetch the newest stable tarball(3.4.5 as of  Dec. 25, 2012)
<pre>
$ wget http://mirror.bjtu.edu.cn/apache/zookeeper/zookeeper-3.4.5/zookeeper-3.4.5.tar.gz
</pre>

Install zookeeper C client library(used by sheep)
<pre>
$ tar -zxvf zookeeper-x.x.x.tar.gz
$ cd zookeeper-x.x.x/src/c
$ ./configure --prefix=/usr
$ make
$ make install
</pre>

Set configuration(how to configure [zookeeper](http://zookeeper.apache.org/doc/r3.4.5/zookeeperAdmin.html#sc_configuration))
<pre>
$ cd zookeeper-x.x.x/conf
$ mv zoo_sample.cfg zoo.cfg
#Setting this to 0 entirely removes the limit on concurrent connections.
$ echo "maxClientCnxns=0" >> zoo.cfg
$ mkdir -p /tmp/zookeeper
</pre> 

Start zookeeper
<pre>
$ sudo ./bin/zkServer.sh start 
</pre>

Enable zookeeper driver in sheepdog
<pre>
$ git clone git://github.com/collie/sheepdog.git
$ cd sheepdog/
$ ./configure --enable-zookeeper
$ make
$ sudo make install
</pre>

Start sheepdog
<pre>
$ sudo sheep /store  -c zookeeper:IP1:2181,IP2:2181,IP3:2181 # use default 30s heartbeat or
$ sudo sheep /store  -c zookeeper:IP1:2181,IP2:2181,IP3:2181,timeout=10s # use 10s heartbeat for small sized cluster like 30 nodes.
</pre>

Lower timeout will reduce the latency to propagate the leave events of failed node, but will consume more network bandwidth. Larger timeout will help reduce the false alarm of failed down in a busy network.

It is recommended to configure dataDir of zookeeper to a  {ramfs, tmpfs, another light use disk} to reduce the chances of operation timeout of zookeeper because of its dumping snapshots onto disk, which might cause several problems of Sheepdog cluster, such as segfault or inconsistent epoch.

## Install zookeeper (Debian-based distribution)
<pre>
$ sudo apt-get install zookeeper zookeeperd
</pre>
Start the zookeeper
<pre>
$ /usr/share/zookeeper/bin/zkServer.sh start # The default port is 2181
</pre>
or
<pre>
$ /etc/init.d/zookeeper start
</pre>
To compile sheep, firstly install zookeeper files (Debian-based distribution)
<pre>
$ sudo apt-get install libzookeeper-dev
</pre>
Then configure, make the sheep source
<pre>
$ ./configure --enable-zookeeper
$ make
$ sudo make install
</pre>

Start the sheep
<pre>
$ sheep -c zookeeper:IP1:PORT1,IP2:PORT2,IP3:PORT3 ...other...option...
</pre>

# Dual NIC

We can choose a dedicated NIC for IO requests, which we call IO NIC, and then cluster management software can use a dedicated NIC, which we call it non-IO NIC,  to pass messages such as node heartbeat to detect if a node is alive.

We also take advantage of redundant NICs. That is, when IO NIC is configured,  we can also fallback on the non-io connection to handle IO requests when IO NIC is down. But this is not true for non-IO NIC. When non-IO NIC is down, our cluster is brain split.

Even we support cluster with heterogeneous NIC configuration but the main purpose is to
allow separation of IO requests and cluster membership heartbeat messages to get
better reliability and scalability if secondary NIC is provided on all the nodes.

Usage:
<pre>
$ sheep -i host=yyy{,port=xxx} ... # this add a dedicated io nic
</pre>