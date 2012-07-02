# Introduction
Every node of Sheepdog cluster has a backend store that provides weighted storage for Sheepdog cluster to store objects (For e.g, VDI and data objects), whose location are determined by consistent hashing algorithm.

Object cache caches data and VDI objects on the local node which runs a sheep daemon. It is at higher level than backend store. This extra cache layer translates gateway requests (from VM) into local requests, largely reducing the network traffic and highly improving the IO performance, at the expense of data inconsistency between objects in object cache and backend store. These dirty objects will be flushed to cluster storage by 'sync' request from guest OS.

If you run QEMU without a local sheep daemon, you need be aware that objects won't be cached at local node, instead will be cached at the node QEMU is remotely connected to.

Let's put it all together from the perspective of the request from VM (with object cache enabled):

IO req (from VM) <--> object cache on local node <--> gateway req <--> targeted sheep backend store

## Farm
Currently Sheepdog supports two kinds of backend store, one is Simple store (removed) and the other called Farm (as of now, default).

Simple store, as the name suggests, it simply translate requests into system calls to store data into files backed by local file systems.

Farm is yet another backend store with much more code lines, which is supposed to provide advanced features such as cluster-wide snapshot, faster recovery, better stale object handling, data de-duplication(not implemented yet) and so on, and tries to keep comparable IO performance as Simple store.

Sheepdog user from Taobao.com uses Farm as its default store since its inception and would continue to tune it into better performance and add more features. You can also request any feature you think of proper in the mailing list.

### Cluster snapshot

### Deal with stale objects

## Object Cache
We can control whether use object cache or not by manipulating the 'cache' option in QEMU command. For example,
<pre>
$ qemu-system-x86_64 --enable-kvm -m 1024 -drive file=sheepdog:test,cache=writeback
</pre>
enables object cache for the dedicated virtual disk image (VDI) named "test" and
<pre>
$ qemu-system-x86_64 --enable-kvm -m 1024 -drive file=sheepdog:test
</pre>
doesn't enable object cache for the 'test'.

There are some more options to do finer control over how object cache does read/write internally
<pre>
  -a, --asyncflush        flush the object cache asynchronously
  -D, --directio          use direct IO when accessing the object from object cache
</pre>

As mentioned above, sheep needs to flush dirty objects to cluster storage. This kind of operation does its job mostly fine, but when the node event(node join/leave) happens, there is a small chance to fail the operation. Even though sheepdog has a built-in retry mechanism to handle this case, unfortunately some flush requests are asked to be finished in a time window. For e.g, journal file system such as Ext4 in Linux kernel will issue a sync request to update its meta periodically and timeout on it, any single failure of such kind of requests will put the file system into read-only. So if you don't have a strong consistency for your VM, you can specify '-a' option to sheep start-up command.

Specify asyncflush means you can tolerate the transient failure of sync request from the guest OS because sheep will try to flush the dirty objects until it succeeds. (Later sync request will trigger flush again)

As default, object cache layer tries to utilize page cache (memory cache) as much as possible, so if you want a more durable cache, you can specify '-D' option. This means we don't use kernel's page cache to further cache data before it reaching to disk, and thus those data can survive the host OS crash.

### Snapshot and Convert
Since **qemu-img** use 'writeback ' or 'unsafe' mode as its default option, with object cached added into Sheepdog, we should pass explicitly a cache control option to stop it from doing anything wrong.

To convert an image to VDI 'test':
<pre>
$ qemu-img convert -t writethrough linux-0.2.img sheepdog:test
</pre>

To snapshot an image named 'test' tagged as 'snap':
<pre>
$ collie vdi snapshot -s snap test
</pre>

Snapshot images can be used as base images for cloned VMs. To clone an existing snapshot 'test' as 'cloned_vm':
<pre>
$ collie vdi clone -s snap test cloned_vm
</pre>

Cloned VMs are implemented by Copy On Write semantics in the Sheepdog cluster, this means cloning operation is very fast and storage-wise cheap, those cloned VMs will share as much as possible data objects from base image. Sheepdog also supports tree structured cloning: you can snapshot the cloned VM and have it as a new base and so on.