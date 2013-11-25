Sheepdog provide some mechanisms, such as object cache, journaling device, etc., which affect the cluster's performance in teams of IO(VM read/write). It is up to the users to enforce policies on these building blocks. Basically, there are following building blocks user can make use:

1. object cache (client side mechanism)
2. journaling device (server side mechanism)
3. directio of read/write for both object cache and backend
4. sync flag for write for backend

As default setup

    $ sheep /store

which set up the cluster with the safest mechanisms, that with object cache disabled, no journaling device, buffered read and make sure every write of backend with SYNC flag set. No surprise, this will result in _worst_ performance. The biggest penalty from default setup is SYNC-flagged write. We can verify how bad the normal synced write is by

    $ dd if=/dev/zero of=test oflag=sync bs=1M count=100

which typically limits the throughout from about 10M/s to 20M/s even for local disk write, which runs in terms of mechanical speed. So with overhead of network and replication, you'll probably experience a less than 10M/s throughput with this setup of sheepdog cluster.

To boost the performance, you should consider object cache at the first place because it allows sheepdog to execute requests in terms of local disk(either sata or ssd) speed without 'sync' flagged (that ranges from 50M/s ~ 500M/s), even in terms of memory speed (that ranges from 500M/s to 2000M/s).

A lesser mechanism is journaling device, which you can double or triple the write performance without object cache enabled, or sync(like you call 'sync' in VM) performance with object cache enabled.

Another option is '-n, --nosync' for sheep daemon, which drops O_SYNC for write of backend. It literally means we don't set 'sync' flag for write of backend. This will dramatically improve write performance if you don't have object cache enabled, at the cost of possibility to lost some data in the case of power failure of the _whole_ cluster. In other words, if only some nodes in the cluster crash, there is no damage of data at all even if you have '--onsync' enabled.(assume the number for failed nodes is protected by the redundancy level). If any one of following conditions goes with your cluster,

* your data center promises there is no power outage
* all the disks are battery-backed
* you don't care about the very low possibility of power outrage and want the best performance

you can enable '--nosync' option for sheep daemon to enjoy write boost.