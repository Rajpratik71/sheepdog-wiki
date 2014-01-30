# Introduction
   Sheepfs is a FUSE-based pseudo file system in userland to access both
sheepdog's internal state (for e.g, cluster info, vdi list) as well as sheepdog's high
reliable storage.

   The idea here is that its sometimes useful that we can envision our interaction with
an sheepdog's object in terms of a directory structure and filesystem operations.

   People mighte be mostly intrested into sheepfs's volume directory, which export
VM's volume as a pseudo block file in your local file system hierarchy, which can be used as

   1. a big file abstraction, which is actually backed by Sheepdog's storage, distributed in
   the cluster.
   2. a loop device file, which you can mount wherever you want to use it as a file system
   backed up by Sheepdog.
   3. a loop device file for some VM's image, which you want to access(RW) its internal data.
   4. stroage media for other hypervisor, such as XEN

This file abstraction integrates well into kernel's pagecache.

## Enable sheepfs
   To compile sheepfs, you need install FUSE devel files.

    $ sudo apt-get install libfuse-dev
 
Then sheepfs will be automatically compiled in. The binary will be located in sheepfs/sheepfs.

## Example
   Some example to show how we can use it. sheepfs -h for more help.

    $ sheepfs /your/mountpoint

   The mountpoint is where we mount our sheepfs, you need to mkdir it first on your own.

   To get cluster info:

    $ cat sheepfs_dir/cluster/info
    Cluster status: running
    
    Cluster created at Mon May 14 15:45:37 2012
    
    Epoch Time           Version
    2012-05-14 15:45:38      1 [127.0.0.1:7000, 127.0.0.1:7001, 127.0.0.1:7002]

   To attach the the volume named of 'test':

    $ echo test > sheepfs_dir/vdi/mount

   Then it will show up at sheepfs_dir/volume/test as a normal file which you can read/write in whatever way you want.

   We can boot it if it is a bootable image:

    $ qemu-system-x86_64 --enable-kvm -m 1024 -drive file=sheepfs_dir/volume/test,cache=writeback.

   Or we can even format it as a file system and mount it somewhere to host our data:

    $ mkfs.ext4 sheepfs_dir/volume/test
    $ sudo mount -o loop sheepfs_dir/volume/test /somewhere

   When the connected sheep daemon crashes, we can re-connect to another live sheep deamon on the fly:

    $ echo ip:port > sheepfs_dir/config/sheep_info

   When you are done with sheepfs, you can umount the sheepfs directory

    $ umount sheepfs_dir

## Sheepfs for swift
   We can also use swift interface for sheepfs.
   
   First set address of swift server:

    $ sheepfs /your/mountpoint
    $ echo "swift_server_ip:swfit_server_port" > /your/mountpoint/http/address

   Then we can echo full path of object (if we have account "robin", container "fruit", object "one"):
    
    $ echo "/robin/fruit/one" > /your/mountpoint/http/object

   Then we can see a file as "/your/mountpoint/http/robin/fruit/one"
