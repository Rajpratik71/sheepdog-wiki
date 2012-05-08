sheepdog use a cluster management backend to manage
memebership and broadcast messages to the cluster nodes.

For now, sheepdog can use [corosync](http://corosync.org/doku.php) (the default), [zookeeper](http://zookeeper.apache.org/) and
[Accord](http://www.osrg.net/accord/).

# Corosync

Corosync runs on every node as a dademon.

Corosync can only works reliably with less than 15 nodes,
due to its implementation and design goal for small sized cluster.

# zookeeper

Zookeeper runs as a standalone cluster. This means you need to set up zookeeper cluster
first, then pass the IP:PORT list to sheep start-up option.

If you want to add more nodes into sheepdog cluster, you should run
sheepdog against zookeeper. Some users (eg. at Taobao.com) have been working with
the scalability of the sheepdog and currently running with around 1000
nodes in their test environment. For several month expediencies with
zookeeper, they have found that it works well for node number below 1000,
with object cache enabled.

To enable zookeeper support (Suppose we have a zookeeper cluster with 3 nodes):

Install zookeeper (Debian-based distribution)
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

