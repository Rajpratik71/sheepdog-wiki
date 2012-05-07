sheepdog use a cluster management backend to manage
memebership and broadcast messages to the cluster nodes.

For now, sheepdog can use [corosync](http://corosync.org/doku.php) (the default), [zookeeper](http://zookeeper.apache.org/) and
[Accord](http://www.osrg.net/accord/).

# Corosync

Corosync can only works reliably with less than 15 nodes,
due to its implementation and design goal for small sized cluster.

# zookeeper

If you want to add more nodes into sheepdog cluster, you should run
sheepdog against zookeeper. Some users (eg. at Taobao.com) have been working with
the scalability of the sheepdog and currently running with around 1000
nodes in their test environment. For several month expediencies with
zookeeper, they have found that it works well for node number below 1000,
with object cache enabled.

# Accord 

For a cluster more than 1000 nodes, I think Accord would come up to our
rescue, but it is currently in a unstable development state. When the
sheepdog scales up to 1000 nodes reliably, we might go to look at Accord
and refine it to be a working state.

