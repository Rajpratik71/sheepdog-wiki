# Getting Started

## Requirements
* One or more x86-64 machines
* Linux kernel 2.6.27 or later
* glibc 2.9 or later
* Zookeeper or corosync
* QEMU 0.13 or later

## Install

A cluster has 3 key components:

1. A cluster manager (like zookeeper or corosync)
1. sheepdog
1. QEMU

Not all of them are required on each node.
Usually sheepdog and QEMU are installed on every node, and the cluster manager on three nodes.

* [Install From Binaries](Install From Binaries)
* [Install From Sources](Install From Sources)
* [Install By Sheepog Utils](Install By Sheepog Utils)

## Configure the cluster manager

This is an example using zookeper on three nodes with the following IPs: 192.168.2.45, 192.168.2.46 and 192.168.2.47.

<pre>
# Same on every node
nano /etc/zookeeper/conf/zoo.cfg
server.45=192.168.2.45:2888:3888
server.46=192.168.2.46:2888:3888
server.47=192.168.2.47:2888:3888
</pre>

<pre>
# Give each node a unique id
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

**Launch the sheepdog daemon**: There's an init script '/etc/init.d/sheepdog', but it doesn't make use of zookeeper. Use it only if your cluster manager is corosync.
The init script also uses /var/lib/sheepdog as a default storage location and you can't reconfigure this.

In this example we have a scenario that:

1. Keeps metadata on the same device as of the operation system (/var/lib/sheepdog)
1. Uses a dedicated mount point for objects (/mnt/sheep/0)
1. Uses zookeeper as cluster manager

<pre>
$ sheep -n  /var/lib/sheepdog,/mnt/sheep/0 -c zookeeper:192.168.2.45:2181,192.168.2.46:2181,192.168.2.47:2181
</pre>

The directory that stores objects (/mnt/sheep/0) must be on a filesystem with xattr support. In case of ext3 or ext4, you need to add 'user_xattr' as a mount option.

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

Formatting the cluster, Sheepdog is going to use the devices you provided for its backend store (or simply store).
Cluster capacity = store size / copies. 

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
1. Create a 5 GB virtual machine image for Alice.
<pre>
$ dog vdi create Alice 5G
</pre>

1. You can also convert from existing KVM images to Sheepdog ones.
<pre>
$ qemu-img convert -t directsync ~/Alice.raw sheepdog:Alice
</pre>

2. See Sheepdog images by running the following command:
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

2. Boot the virtual machine with a sheepdog daemon running on a remote node
<pre>
$ qemu-system-x86_64 sheepdog:192.168.2.45:7000:Alice
</pre>
This suppose you have one sheep started up on the node with the IP 192.168.2.45 and port 7000

3. Sheepdog supports a local cache called the object cache
The object cache caches data and vdi objects on the local node. It is at a
higher level than the backend store. This extra cache layer translates gateway
requests into local requests, largely reducing the network traffic and highly
improves the IO performance.
Dirty objects will be flushed to the cluster storage by 'sync' requests from the
guest OS.
You have to run the latest QEMU version to work with the object cache. To enable the object cache,
you can start QEMU by using the following command:
<pre>
$ qemu-system-x86_64 -drive file=sheepdog:Alice,cache=writeback
</pre>
Note: Be careful using object cache.  
Note2: sheep daemon has to be running with (-w, --cache) option. Check the syntax.

### Snapshot
1. Create a snapshot
<pre>
$ dog vdi snapshot -s preupgrade test2
</pre>

1. After getting a snapshot, a new virtual machine image is added as a not-current image.
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
1. Create a "Charlie" image as a clone of the test2 image.
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

### qemu-img

All the operations on vdi can be done by qemu-img plus some other.
<pre>
# Create empty vdi (default as raw image)
qemu-img create sheepdog:Alice 256G
# Create empty vdi with qcow2 format so you can enjoy all the qcow2 features
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