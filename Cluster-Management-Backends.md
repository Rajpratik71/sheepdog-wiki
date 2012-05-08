Sheepdog uses a cluster management backend to manage
memebership and broadcast messages to the cluster nodes.

For now, sheepdog can use [corosync](http://corosync.org/doku.php) (the default), [zookeeper](http://zookeeper.apache.org/) and
[Accord](http://www.osrg.net/accord/).

# Corosync

Corosync runs on every node as a dademon.

Corosync can only works reliably with less than 15 nodes,
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
$ ./configure
$ make
$ make install
</pre>

Set configuration(how to configure [zookeeper](http://zookeeper.apache.org/doc/r3.3.3/zookeeperAdmin.html#sc_configuration))
<pre>
$ cd zookeeper-3.3.4/conf
$ mv zoo_sample.cfg zoo.cfg
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
$ ./configure --enable-debug --enable-zookeeper
$ make
$ sudo make install
</pre>

Start sheepdog
<pre>
$ sudo sheep -d /store/29 -z 29 -p 7029 -c zookeeper:127.0.0.1:2181
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

# Accord 

For a cluster more than 1000 nodes, I think Accord would come up to our
rescue, but it is currently in a unstable development state. When the
sheepdog scales up to 1000 nodes reliably, we might go to look at Accord
and refine it to be a working state.