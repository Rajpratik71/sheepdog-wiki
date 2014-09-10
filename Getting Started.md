# Getting Started

## Requirements
* One or more x86-64 machines.
* Linux kernel 2.6.27 or later
* glibc 2.9 or later
* Zookeeper or corosync
* QEMU 0.13 or later

## Install

A cluster has 3 key compenents:

1. a cluster manager (zookeeper);
1. sheepdog;
1. qemu.

Not all of them are required on each nodes.
Usually sheepdog amd qemu are installed on every node.
Zookeer on three nodes.

* [Install From Binaries](Install From Binaries)
* [Install From Sources](Install From Sources)

## Configure the cluste manager

This is an example using three nodes with ip 192.168.2.45,192.168.2.46,192.168.2.47.

<pre>
# Same on every node
nano /etc/zookeeper/conf/zoo.cfg
server.45=192.168.2.45:2888:3888
server.46=192.168.2.46:2888:3888
server.47=192.168.2.47:2888:3888
</pre>

<pre>
# Give each node set a different id
# On 192.168.2.45
nano /etc/zookeeper/conf/myid
45
# On 192.168.2.46
nano /etc/zookeeper/conf/myid
46
# On 192.168.2.47
nano /etc/zookeeper/conf/myid
47
</pre>

<pre>
# Restart the service on every node
service zookeeper restart
</pre>

## Usage

### Setup Sheepdog

**Launch the sheepdog daemon**: there's an init script '/etc/init.d/sheepdog' but it doesn't make use of zookeeper. Use it only if your cluster manager is corosync.
It also use /var/lib/sheepdog as default and you can't specify other directories.

In this example, is shown a real scenario that:
1. keeps metatada on the same device of the operation system (/var/lib/sheepdog)
1. uses a dedicated mount point (/mnt/sheep/0)
1. uses zookeeper as cluster manager

<pre>
$ sheep -n  /var/lib/sheepdog,/mnt/sheep/0 -c zookeeper:192.168.2.45:2181,192.168.2.46:2181,192.168.2.47:2181
</pre>

The directory that sotres object (/mnt/sheep/0) must be on a filesystem with an xattr support. In case of ext3 or ext4, you need to add 'user_xattr' to the mount options.

<pre>
# fstab entry example
/dev/vg00/sheep0        /mnt/sheep/0    ext4     defaults,noatime,user_xattr      0       0
# To change it in run time
$ sudo mount -o remount,user_xattr /mnt/sheep/0
</pre>

**Format sheepdog cluster**
<pre>
$ dog cluster format
</pre>
It is enough to run this command on just one machine.  
"--copies" specifies the number of default data redundancy.  
In this case, the replicated data will be stored on three machines.

Sheepdog has the back-end store to store the data object persistently on the node basis. 

**Check cluster state**

The following list shows that Sheepdog is running on 4 nodes.
<pre>
 $ dog node list
  Id   Host:Port         V-Nodes       Zone
   0   192.168.2.4:7000        126   67807424
   1   192.168.2.5:7000        129   84584640
   2   192.168.2.6:7000        129  101361856
   3   192.168.2.7:7000        129  118139072
</pre>

### Create an empty VDI
1. Create a 256 GB virtual machine image of Alice.
<pre>
$ dog vdi create Alice 256G
</pre>

1. You can also convert from existing KVM images to Sheepdog ones.
<pre>
$ qemu-img convert -t directsync ~/Alice.raw sheepdog:Alice
</pre>

2. See Sheepdog images by the following command.
<pre>
$ dog vdi list
  Name        Id    Size    Used  Shared    Creation time   VDI id  Copies  Tag
  Alice        0  5.0 GB  864 MB  0.0 MB 2014-09-10 14:57   15d167      3
</pre>
Play with more vdi
<pre>
$ dog vdi list
  Name        Id    Size    Used  Shared    Creation time   VDI id  Copies  Tag
  Alice        0  5.0 GB  864 MB  0.0 MB 2014-09-10 14:57   15d167      3
  test5        0  5.0 GB  864 MB  0.0 MB 2014-09-10 13:04   fd2c30      3              
  test4        0  5.0 GB  864 MB  0.0 MB 2014-09-10 13:04   fd2de4      3              
  test6        0  5.0 GB  864 MB  0.0 MB 2014-09-10 13:05   fd3149      3              
  test1        0  5.0 GB  864 MB  0.0 MB 2014-09-10 09:48   fd32fc      3              
  test3        0  5.0 GB  864 MB  0.0 MB 2014-09-10 13:03   fd3663      3              
  test2        0  5.0 GB  864 MB  0.0 MB 2014-09-10 13:03   fd3816      3
</pre>

### Boot the VM
1. Boot the virtual machine with sheep daemon started on local node
<pre>
$ qemu-system-x86_64 sheepdog:Alice
</pre>

2. Boot the virtual machine without sheep daemon started on local node
<pre>
$ qemu-system-x86_64 sheepdog:192.168.2.2:7000:Alice
</pre>
This suppose you have one sheep started up on the node with the IP 192.168.2.2 and port 7000

3. Sheepdog supports local cache called object cache.
Object cache caches data and vdi objects on the local node. It is at
higher level than backend store. This extra cache layer translate gateway
requests into local requests, largely reducing the network traffic and highly
improve the IO performance.
Dirty objects will be flushed to the cluster storage by 'sync' request from
guest OS.
You have to run latest QEMU to work with object cache. To enable the object cache,
you can start QEMU by following command:
<pre>
$ qemu-system-x86_64 -drive file=sheepdog:Alice,cache=writeback
</pre>
Note: be carefull using object cache.
Note2: sheep daemon has to be running with (-w, --cache) option. Check the syntax.

### Snapshot
1. Create a snapshot
<pre>
$ dog vdi snapshot -s preupgrade sheepdog:test2
</pre>

1. After getting snapshot, a new virtual machine images are added as a not-current image.
<pre>
$ dog vdi list
  Name        Id    Size    Used  Shared    Creation time   VDI id  Copies  Tag
  Alice        0  5.0 GB  864 MB  0.0 MB 2014-09-10 14:57   15d167      3
  test5        0  5.0 GB  864 MB  0.0 MB 2014-09-10 13:04   fd2c30      3              
  test4        0  5.0 GB  864 MB  0.0 MB 2014-09-10 13:04   fd2de4      3              
  test6        0  5.0 GB  864 MB  0.0 MB 2014-09-10 13:05   fd3149      3              
  test1        0  5.0 GB  864 MB  0.0 MB 2014-09-10 09:48   fd32fc      3              
  test3        0  5.0 GB  864 MB  0.0 MB 2014-09-10 13:03   fd3663      3              
s test2        1  5.0 GB  864 MB  0.0 MB 2014-09-10 13:03   fd3816      3    preupgrade
  test2        0  5.0 GB  0.0 MB  864 MB 2014-09-10 14:22   fd3817      3       
</pre>

### Cloning from the snapshot
1. Create a Charlie's image as a clone of test2 image.
<pre>
$ dog vdi clone -s preupgrade test2 Charlie
</pre>

1. Charlie's image is added to the virtual machine list.
<pre>
$ dog vdi list
  Name        Id    Size    Used  Shared    Creation time   VDI id  Copies  Tag
  Alice        0  5.0 GB  864 MB  0.0 MB 2014-09-10 14:57   15d167      3              
c Charlie      0  5.0 GB  0.0 MB  864 MB 2014-09-10 15:02   3ca881      3              
  test5        0  5.0 GB  864 MB  0.0 MB 2014-09-10 13:04   fd2c30      3              
  test4        0  5.0 GB  864 MB  0.0 MB 2014-09-10 13:04   fd2de4      3              
  test6        0  5.0 GB  864 MB  0.0 MB 2014-09-10 13:05   fd3149      3              
  test1        0  5.0 GB  864 MB  0.0 MB 2014-09-10 09:48   fd32fc      3              
  test3        0  5.0 GB  864 MB  0.0 MB 2014-09-10 13:03   fd3663      3              
s test2        1  5.0 GB  864 MB  0.0 MB 2014-09-10 13:03   fd3816      3    preupgrade
  test2        0  5.0 GB  0.0 MB  864 MB 2014-09-10 14:22   fd3817      3
</pre>

### Qemu-img

All the operation on vdi can be done by qemu-img plus some other.
<pre>
# Create empty vdi (default as raw image)
qemu-img create sheepdog:Alice 256G
# Create empty vdi with qcow2 format: you can enjoy all the qcow2 features
qemu-img create -f qcow2 sheepdog:Alice 256G 
# Snapshot
qemu-img create -b sheepdog:test2:1 sheepdog:Charlie
# Cloning
qemu-img snapshot -c preupgrade sheepdog:test2
</pre>

### Shutdown Sheepdog
1. Run the shutdown command on the one of the cluster machines.
<pre>
$ dog cluster shutdown
</pre>

This command stops all the sheep processes on the cluster.

### Test Environment
* Debian wheezy amd64