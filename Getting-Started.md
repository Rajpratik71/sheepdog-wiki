# Getting Started

## Requirements
* One or more x86-64 machines.
* Linux kernel 2.6.27 or later
* glibc 2.9 or later
* The corosync and corosync lib package.
* QEMU 0.13 or later

## Compile-time dependencies
* GNU Autotools
* pkg-config
* corosync devel package
* git (when compiling from source repo)
* liburcu
* optional:fuse-devel (for sheepfs)

## Install

### Compile or install the Corosync packages
Nearly every modern Linux distribution has x86_64 corosync binaries pre-built available via their repositories.  We recommend you use these packages if they are available on your distribution.

For debian package based systems:
<pre>
$ sudo aptitude install corosync libcorosync-dev
</pre>

For RPM package based systems:
<pre>
$ sudo yum install corosynclib-devel
</pre>

For EL6 (RHEL, CentOS, SL, etc), the provided version of corosync is too old and you must install corosync from source. The following command will remove any existing copies of corosync.

<pre>
$ sudo yum remove corosync corosynclib corosynclib-devel 
</pre>

If your distribution doesn't provide the corosync packages, or you prefer to compile from source:
<pre>
$ git clone git://github.com/corosync/corosync.git
$ cd corosync
$ git checkout -b flatiron origin/flatiron
$ ./autogen.sh
$ ./configure --enable-nss
$ sudo make install
</pre>

See also: [[Corosync Config]]

### Download, build and install QEMU with Sheepdog support

QEMU 0.13 provides built-in support for sheepdog devices.  Some distributions provide pre-built versions of this newer version of QEMU.  If your distribution has an older version of QEMU or you prefer to compile from source, retrieve the latest QEMU and compile:
<pre>
$ git clone git://git.sv.gnu.org/qemu.git
$ cd qemu
$ ./configure
$ sudo make install
$ cd ..
</pre>

### Download, build and install the Sheepdog server and command line tools

If you want to use a stable sheepdog, please download our source archive of stable branch and build it. Stable branches of each version are listed [here] (https://github.com/sheepdog/sheepdog/wiki/Stable-branches#obtain-stable-release-based-on-the-stable-branches). Below is an example of v0.7.6.

<pre>
$ wget -O sheepdog-v0.7.6.tar.gz https://github.com/sheepdog/sheepdog/archive/v0.7.6.tar.gz
$ tar xvzf sheepdog-v0.7.6.tar.gz
$ cd sheepdog-v0.7.6
$ ./autogen.sh
$ ./configure
$ sudo make install
$ cd ..
</pre>

If you want to use every feature of the latest sheepdog, please clone our git repository.

<pre>
$ git clone git://github.com/sheepdog/sheepdog.git
$ cd sheepdog
$ ./autogen.sh
$ ./configure
$ sudo make install
$ cd ..
</pre>

Please note, sheepdog supports a "make rpm" target which will generate an rpm package that can be installed on the local machine.  To use this installation method, use the following instructions:
<pre>
$ git clone git://github.com/collie/sheepdog.git
$ cd sheepdog
$ ./autogen.sh
$ ./configure
$ make rpm
$ sudo rpm -ivh x86_64/sheepdog-0.*
$ cd ..
</pre>

## Usage

### Setup Sheepdog

1. To launch the sheepdog daemon, either start it using '/etc/init.d/sheepdog start' or launch the sheepdog daemon by directly calling 'sheepdog <store_dir>' on each machine of the cluster. Make sure that TCP port 7000 is open for sheepdog communication.

Note: At this time, the sheepdog init.d script is hard-coded to use /var/lib/sheepdog as the store directory. If you wish to use another directory, you will need to always start sheepdog using the 'sheep <store_dir>' call.

<pre>
$ sudo /etc/init.d/sheepdog start
</pre>

Or;

<pre>
$ sudo sheep /var/lib/sheepdog
</pre>

The /var/lib/sheepdog will be where sheepdog will store objects. The directory must be on a filesystem with an xattr support. In case of ext3 or ext4, you need to add 'user_xattr' to the mount options.

<pre>
$ sudo mount -o remount,user_xattr /var/lib/sheepdog
</pre>

1. Format sheepdog cluster
<pre>
$ collie cluster format --copies=3
</pre>
It is enough to run this command on just one machine.
"--copies" specifies the number of default data redundancy. In this case, the replicated data will be stored on three machines.

Sheepdog has the back-end store to store the data object persistently on the node basis. 

1. Check cluster state  
Following list shows that Sheepdog is running on 32 nodes.
<pre>
 $ collie node list
   Idx	  Node id (FNV-1a) - Host:Port
 ------------------------------------------------
  &nbsp;0	0308164db75cff7e - 10.68.13.15:7000
  &nbsp;1	03104d8b4315c8e4 - 10.68.13.1:7000
  &nbsp;2	0ab18c565bc14aea - 10.68.13.3:7000
  &nbsp;3	0c0d27f0ac395f5d - 10.68.13.16:7000
  &nbsp;4	127ee4802991f308 - 10.68.13.13:7000
  &nbsp;5	135ff2beab2a9809 - 10.68.14.5:7000
  &nbsp;6	17bd6240eab65870 - 10.68.14.4:7000
  &nbsp;7	1cf35757cbf47d7b - 10.68.13.10:7000
  &nbsp;8	1df9580b8960a992 - 10.68.13.11:7000
  &nbsp;9	29307d3fa5a04f78 - 10.68.14.12:7000
  10	29dcb3474e31d4f3 - 10.68.14.15:7000
  11	29e089c98dd2a144 - 10.68.14.16:7000
  12	2a118b7e2738f479 - 10.68.13.4:7000
  13	3d6aea26ba79d75f - 10.68.13.6:7000
  14	42f9444ead801767 - 10.68.14.11:7000
  15	562c6f38283d09fe - 10.68.14.2:7000
  16	5dd5e540cca1556a - 10.68.14.6:7000
  17	6c12a5d10f10e291 - 10.68.14.13:7000
  18	6dae1d955ca72d96 - 10.68.13.7:7000
  19	711db0f5fa40b412 - 10.68.14.14:7000
  20	7c6b95212ee7c085 - 10.68.14.9:7000
  21	7d010c31bf11df73 - 10.68.13.2:7000
  22	82c43e908b1f3f01 - 10.68.13.12:7000
  23	931d2de0aaf61cf5 - 10.68.13.8:7000
  24	961d9d391e6021e7 - 10.68.13.14:7000
  25	9a3ef6fa1081026c - 10.68.13.9:7000
  26	b0b3d300fed8bc26 - 10.68.14.10:7000
  27	b0f08fb98c8f5edc - 10.68.14.8:7000
  28	b9cc316dc5aba880 - 10.68.13.5:7000
  29	d9eda1ec29c2eeeb - 10.68.14.7:7000
  30	e53cebb2617c86fd - 10.68.14.1:7000
  31	ea46913c4999ccdf - 10.68.14.3:7000
</pre>

### Create a VM image
1. Create a 256 GB virtual machine image of Alice.
<pre>
$ qemu-img create sheepdog:Alice 256G # default as raw image
$ qemu-img create -f qcow2 sheepdog:Alice 256G # you can enjoy all the qcow2 features
</pre>

1. You can also convert from existing KVM images to Sheepdog ones.
<pre>
$ qemu-img convert -t directsync ~/amd64.raw sheepdog:Bob
</pre>

1. See Sheepdog images by the following command.
<pre>
$ collie vdi list
  name        id    size    used  shared    creation time  object id
--------------------------------------------------------------------
  Bob          0  2.0 GB  1.6 GB  0.0 MB 2010-03-23 16:16      80000
  Alice        0  256 GB  0.0 MB  0.0 MB 2010-03-23 16:16      40000
</pre>

### Boot the VM
1. Boot the virtual machine with sheep daemon started on local node
<pre>
$ qemu-system-x86_64 sheepdog:Alice
</pre>

2. Boot the virtual machine without sheep daemon started on local node
<pre>
$ qemu-system-x86_64 sheepdog:192.168.10.2:7000:Alice
</pre>
This suppose you have one sheep started up on the node with the IP 192.168.10.2 and port 7000

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

### Snapshot
1. Create a snapshot
<pre>
$ qemu-img snapshot -c name sheepdog:Alice
</pre>

1. After getting snapshot, a new virtual machine images are added as a not-current image.
<pre>
$ collie vdi list
&nbsp; name        id    size    used  shared    creation time  object id
--------------------------------------------------------------------
&nbsp; Bob          0  2.0 GB  1.6 GB  0.0 MB 2010-03-23 16:16      80000
&nbsp; Alice        0  256 GB  0.0 MB  0.0 MB 2010-03-23 16:21      c0000
s Alice        1  256 GB  0.0 MB  0.0 MB 2010-03-23 16:16      40000
</pre>

1. You can boot from the snapshot image by specifying snapshot id
<pre>
$ qemu-system-x86_64 sheepdog:Alice:1
</pre>

### Cloning from the snapshot
1. Create a Charlie's image as a clone of Alice's image.
<pre>
$ qemu-img create -b sheepdog:Alice:1 sheepdog:Charlie
</pre>

1. Charlie's image is added to the virtual machine list.
<pre>
$ collie vdi list
&nbsp; name        id    size    used  shared    creation time  object id
--------------------------------------------------------------------
&nbsp; Bob          0  2.0 GB  1.6 GB  0.0 MB 2010-03-23 16:16      80000
&nbsp; Alice        0  256 GB  0.0 MB  0.0 MB 2010-03-23 16:21      c0000
s Alice        1  256 GB  0.0 MB  0.0 MB 2010-03-23 16:16      40000
&nbsp; Charlie      0  256 GB  0.0 MB  0.0 MB 2010-03-23 16:23     100000
</pre>

### Shutdown Sheepdog
1. Run the shutdown command on the one of the cluster machines.
<pre>
$ collie cluster shutdown
</pre>

This command stops all the sheep processes on the cluster.

### Test Environment
* Debian squeeze amd64