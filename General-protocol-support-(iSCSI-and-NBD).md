## iSCSI

iSCSI support of sheepdog is provided by tgt, Linux SCSI target framework ( https://github.com/fujita/tgt ). If you want to attach VDIs of sheepdog via iSCSI, you need to install tgt.

### 1. Install and launch tgt

<pre>
$ git clone https://github.com/fujita/tgt.git
$ cd tgt
$ make
# make install
</pre>


### 2. Create a sheepdog VDI

There is no special procedures in this step.
<pre>
$ dog vdi create tgt0 100G
</pre>

Of course you can use existing VDIs as iSCSI logical units.

### 3. Setup iSCSI target provided by tgt

One logical unit corresponds to one VDI of sheepdog. In this step, we create iSCSI target and logical unit which can be seen by iSCSI initiator.

<pre>
# tgtd
# tgtadm --op new --mode target --tid 1 --lld iscsi --targetname iqn.2013-10.org.sheepdog-project
# tgtadm --op new --mode lu --tid 1 --lun 2 --bstype sheepdog --backing-store unix:/sheep_store/sock:tgt0
# tgtadm --lld iscsi --op bind --mode target --tid 1 -I ALL
</pre>

The most important parameter is --backing-store which is required by tgtadm when we create the logical unit in the target (the third line of the above commands). With this parameter, we tell the tgtd process how to connect to the sheep process, which VDI we use as the logical unit, which snapshot ID of the VDI we use.
The form of the --backing-store option is like this:

<pre>
unix:path_of_unix_domain_socket:vdi
unix:path_of_unix_domain_socket:vdi:tag
unix:path_of_unix_domain_socket:vdi:snapid
tcp:host:port:vdi
tcp:host:port:vdi:tag
tcp:host:port:vdi:snapid
</pre>

We use a special case of the first one in the above command sequence. It means "connect to sheep process via unix domain socket protocol, path of unix domain socket is /sheep_store/sock, use VDI named tgt0 as a backing store of newly created logical unit".

You can also specify sheep which is running on remote host via TCP. But if tgtd process is running on a same host of sheep process, there is no benefit of using TCP. Unix domain socket can provide faster connection than TCP.

You can also specify tag or ID of snapshots via suffix of the --backing-store option.

### 4. Setup iSCSI session (example of the open-iscsi initiator on Linux)

After setting up iSCSI target, you can use the VDIs from any virtual machines and operating systems which supports iSCSI initiator. Many of popular hypervisors and operating systems support it (e.g. VMware ESX Family, Linux, Windows, etc). In this example, the way of Linux + open-iscsi is described.

At first, you have to install open-iscsi ( http://www.open-iscsi.org/ ) and launch it. Major linux distros provide their open-iscsi package. Below is a way of installation in Debian and Ubuntu based systems.
<pre>
# apt-get install open-iscsi
# /etc/init.d/open-iscsi start
</pre>

Next, we need to let iscsid discover and login to the target we've already created in the above sequence. If the initiator is running on different host from the target, you have to change the IP addresses in the below commands.

<pre>
# iscsiadm -m discovery -t st -p 127.0.0.1
# iscsiadm -m node --targetname iqn.2013-10.org.sheepdog-project --portal 127.0.0.1:3260 --login
</pre>

New device files, e.g. /dev/sdb, will be created on your system after login completion. Now your system can use the speepdog VDIs like ordinal HDDs.

## NBD
1. Create a Sheepdog image
<pre>
$ qemu-img create sheepdog:image 4G
</pre>

1. Start qemu-nbd on the one of Sheepdog servers
<pre>
$ qemu-nbd sheepdog:image
</pre>