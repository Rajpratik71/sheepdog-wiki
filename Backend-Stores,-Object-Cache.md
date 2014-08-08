# Introduction
Every node of Sheepdog cluster has a backend store that provides weighted storage for Sheepdog cluster to store objects (For e.g, VDI and data objects), whose location are determined by consistent hashing algorithm.

Object cache caches data and VDI objects on the local node which runs a sheep daemon. It is at higher level than backend store. This extra cache layer translates gateway requests (from VM) into local requests, largely reducing the network traffic and highly improving the IO performance, at the expense of data inconsistency between objects in object cache and backend store. These dirty objects will be flushed to cluster storage by 'sync' request from guest OS.

Object cache supports cache quota. As explained below, it can be specified with command line option.

In previous sheepdog, writeback or writethrough of object cache could be specified with command line option.
But in current sheepdog, writeback or writethrough is determined by request from QEMU.


If you run QEMU without a local sheep daemon, you need be aware that objects won't be cached at local node, instead will be cached at the node QEMU is remotely connected to.

Let's put it all together from the perspective of the request from VM (with object cache enabled):

IO req (from VM) <--> object cache on local node <--> gateway req <--> targeted sheep backend store

## Farm
In the old sheep (before 0.6.0), farm referred to one of the backend store implementations. Now it refers to the mechanism collie uses to do cluster wide snapshot backup.

### Cluster backup
We can use collie command to save all the snapshots of the running cluster into a single directory (currently directory in the local file system but other protocols are planned). Replicated snapshots will only store one copy in the backup directory so the suggested storage for backup data are RAID disk arrays or other dedicated backup storage.

Data of the snapshot volume and  memory state from live snapshot are both backed up.

Backup data are automatically
*  deduplicated while storing into medium (from test, we notice up-to 50% dedup ratio)
*  self-validated of data while reading from backup medium
*  incrementally backed up

Usage
<pre>
$ collie cluster snapshot save your-tag /path/to/backup # save the backup and tag it
$ collie cluster snapshot list /path/tobackup # list the backups with tag names
$ collie cluster snapshot load your-tag /path/to/backup # restore the tagged backup
</pre>

In current master branch, save or load specified VDIs is supported. A new subcommand, cluster snapshot show, is introduced to show VDIs in the tagged backup.

Usage
<pre>
$ collie cluster snapshot save your-tag /path/to/backup vdi1 vdi2 vdi3 # save specified vdis only
$ collie cluster snapshot show your-tag /path/to/backup # show vdis in the tagged backup
$ collie cluster snapshot load your-tag /path/to/backup vdi3 # restore vdi3 only
</pre>

## Object cache
Generally speaking, we support _writeback_, which caches write update for a period of time and flushes dirty bits by a 'sync' request from Guest OS, routed by QEMU and _writethrough_, which means we don't need care about cache and backend consistency.

For writeback mode, you'll enjoy near native local image performance and for writethrough mode, we can also enjoy performance boost for read requests. (You can think it as read only cache)

We can control which mode object cache is on by manipulating the 'cache' option in QEMU command. For example,

<pre>
$ qemu-system-x86_64 --enable-kvm -m 1024 -drive file=sheepdog:test,cache=writeback
</pre>
enables writeback mode for the dedicated virtual disk image (VDI) named "test" and
<pre>
$ qemu-system-x86_64 --enable-kvm -m 1024 -drive file=sheepdog:test
</pre>
or 
<pre>
$ qemu-system-x86_64 --enable-kvm -m 1024 -drive file=sheepdog:test,cache=writethrough
</pre>
enables writethrough mode.

Object cache is disabled by default in Sheepdog. '-w' is used for enabling object cache in local node. Example of '-w' is like this:
<pre>
   sheep -w size=200G # enable object cache with 200G space
   sheep -w size=50G,directio # enable object cache with 50G space with O_DIRECT for cached objects
</pre>

There are some more options to do finer control over how object cache does read/write internally

As default, object cache layer tries to utilize page cache (memory cache) as much as possible, so if you want a more durable cache, you can specify '-w directio' option. This means we don't use kernel's page cache to further cache data before it reaching to disk, and thus those data can survive the host OS crash.

For users that want to use a faster disk to accelerate the IO performance with object cache, you can also specify where the object cache directory is located by:
<pre>
   sheep -w size=200G,dir=/path/to/cache # enable object cache with 200G to /path/to/cache directory
</pre>

### How object cache works
Let me start with local file as backend of block device of QEMU. It
basically uses host memory pages to cache blocks of emulated device.
Kernel internally maps those blocks into pages of file (A.K.A page
cache) and then we relies on the kernel memory subsystem to do writeback
of those cached pages. When VM read/write some blocks, kernel allocate
pages on demand to serve the read/write requests operated on the pages.

<pre>
QEMU 《----》 VM
  ^
  |                                   writeback/readahead pages
  V                                                 |
POSIX file 《 --- 》 page cache 《 --- 》 disk
                                    |
          kernel does page wb/ra and reclaim
</pre>
Object cache of Sheepdog do the similar things, the difference is that
we map those requested blocks into objects (which is plain fixed size
file on each node) and the sheep daemon play the role of kernel that
doing writeback of the dirty objects and reclaim of the clean objects to
make room to allocate objects for other requests.
<pre>
QEMU 《----》VM
  ^
  |                                                       push/pull objects
  V                                                              |
Sheepdog device 《----》 object cache 《---》 Sheepdog replicated object storage.
                                                  |
               Sheep daemon does object push/pull and reclaim
</pre>

Object is implemented as fixed size file on disks, so for object cache,
those objects are all fixed size files on the node that sheep daemon
runs and sheep does directio on them. In this sense that we don't
consume memory, except those objects' metadata(inode & dentry) on the node.
### Snapshot and Convert
Since **qemu-img** use 'writeback ' or 'unsafe' mode as its default option, with object cached added into Sheepdog, we should pass explicitly a cache control option to stop it from doing anything wrong.

To convert an image to VDI 'test':
<pre>
$ qemu-img convert -t directsync linux-0.2.img sheepdog:test
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