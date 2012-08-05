Sheepdog uses a cluster management backend to manage
membership and broadcast messages to the cluster nodes.

For now, sheepdog can use local driver (for development on a single box), [corosync](http://corosync.org/doku.php) (the default), [zookeeper](http://zookeeper.apache.org/) and
[Accord](http://www.osrg.net/accord/).

# Local driver
This driver just makes use of Unix IPC mechanism to management the membership on a single box, where we start multiple 'sheep' processes to simulate the cluster. It is very easy and fast setup and especially useful to test functionality and for developer without involving any other software.

To set up a 3 node using local driver in one liner bash with debug mode (mkdir /path/to/store first):

    $ for i in 0 1 2; do sheep -c local -d /path/to/store/$i -z $i -p 700$i;sleep 1;done

# Corosync

Corosync runs on every node as a daemon.

Corosync can only work reliably with less than 15 nodes,
due to its implementation and design goal for small sized cluster.

# Zookeeper

Zookeeper runs as a standalone cluster. This means you need to set up zookeeper cluster
first, then pass the IP:PORT list to sheep start-up option.

If you want to add more nodes into sheepdog cluster, you should run
sheepdog against zookeeper. Some users (eg. at Taobao.com) have been working with
the scalability of the sheepdog and currently running with around 1000
nodes in their test environment. For several month expediencies with
zookeeper, they have found that it works well for node number below 1000,
with object cache enabled.

To enable zookeeper support (Suppose we have a zookeeper cluster with 3 nodes):

## Install zookeeper (by source tarball)
Fetch the newest tarball(>= 3.3.4)
<pre>
$ wget http://mirror.bjtu.edu.cn/apache/zookeeper/zookeeper-3.3.4/zookeeper-3.3.4.tar.gz
</pre>

Install zookeeper C client library(used by sheep)
<pre>
$ tar -zxvf zookeeper-3.3.4.tar.gz
$ cd zookeeper-3.3.4/src/c
$ ./configure --prefix=/usr
$ make
$ make install
</pre>

Set configuration(how to configure [zookeeper](http://zookeeper.apache.org/doc/r3.3.3/zookeeperAdmin.html#sc_configuration))
<pre>
$ cd zookeeper-3.3.4/conf
$ mv zoo_sample.cfg zoo.cfg
//Setting this to 0 entirely removes the limit on concurrent connections.
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

Start sheepdog with zookeeper
<pre>
$ sudo sheep -d /store/29 -z 29 -p 7029 -c zookeeper:127.0.0.1:2181
</pre>

**Note**: We should not start multiple sheeps at the same time when use zookeeper driver. In fact, we *just* need to start _the first_ sheep separately, after that, we can start other sheeps *concurrently*. Let's say, you want to start 100 sheeps:
<pre>
- start the fist sheep alone, and sleep 2 seconds:
$ sheep -d /store/0 -z 0 -p 7000 -c zookeeper:localhost:2181
$ sleep 2

- start other sheeps simultaneously(need not to sleep between them):
$ for i in {1..99}; do sheep -d /store/$i -z $i -p $((7000 + $i)) -c zookeeper:localhost:2181
</pre>

## Install zookeeper (Debian-based distribution)
<pre>
$ sudo apt-get install zookeeper
</pre>
Start the zookeeper
<pre>
$ /usr/share/zookeeper/bin/zkServer.sh start # The default port is 2181
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

**Note**: We should not start multiple sheeps at the same time when use zookeeper driver. In fact, we *just* need to start _the first_ sheep separately, after that, we can start other sheeps *concurrently*. Let's say, you want to start 100 sheeps:
<pre>
- start the fist sheep alone, and sleep 2 seconds:
$ sheep -d /store/0 -z 0 -p 7000 -c zookeeper:localhost:2181
$ sleep 2

- start other sheeps simultaneously(need not to sleep between them):
$ for i in {1..99}; do sheep -d /store/$i -z $i -p $((7000 + $i)) -c zookeeper:localhost:2181
</pre>

# Accord 

For a cluster more than 1000 nodes, I think Accord would come up to our
rescue, but it is currently in a unstable development state. When the
sheepdog scales up to 1000 nodes reliably, we might go to look at Accord
and refine it to be a working state.