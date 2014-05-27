This is similar to Ceph's RBD. The main motivation is to replace complex
and ineffecient middle ware (such as iscsi softwafe) with simple software stacks
to expose sheepdog storage as Linux block device interface, which means that we
can make use of page cache as a client cache for buffered read/write optionally
and behaves as a normal Linux block device(s) in your local file system.

With single major allocation scheme, we support 31 partitions for a sheep block
device at most and 32768 devices can be attached to local fs for a single node.

##To complile:
<pre>
   $ cd sheepdog/sbd/;make
   $ insmod sbd.ko
</pre>

##Usage:

We control the device the same way as RBD.

<pre>
# associate vdi 'test' to /dev/sbd0, assume your have sheep daemon listening on localhost
$ echo 127.0.0.1 7000 test > /sys/bus/sbd/add

# list the mapped devices
$ cat /sys/buf/sbd/list
0 test

# remove the device sbd0
$ echo 0 > /sys/bus/sbd/remove
</pre>

To get best of performance,
<pre>
# echo 4096 > /sys/block/sbd0/queue/max_sectors_kb
</pre>

Which means io scheduler will try its best to handle us 4MB request.

TODO
- support cloned sheep vdi
- auto-reconncect to sheep daemon if connection is off/crashed
- better error handling
- block device multi-queue support for recent kernel
- live snapshot of sbd
- support hyper volume