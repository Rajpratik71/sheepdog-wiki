# Introduction

Multi-disk support means we can run a single sheep daemon to mange multiple disks on the local node. Previously, we have to run sheep per disk. The management of local disks is as intelligent as our node management, that is, sheep daemon will automatically handle the failures of disks and re-balance/recover data objects on the alive disks and you can hot-plug/hot-unplug the disks on the fly, no manual manage of disks or configuration is needed. 

The basic idea of this multi-disk(MD) is implement RAID-0 like mechanism that distributes sheep objects on the local disks without parity or replicating, which instead relies on the sheepdog's replicated storage to recover the lost objects on the faulty disk or migrate objects to newly add disks for data re-balance. To conclude, MD will make best performance out of underlying disks like RAID-0 and have recovery capability without using extra parity storage (RAID5) or replicating storage (RAID10).

The MD module use a private consistent hash ring per sheep for object distributing, which allow MD layer completely transparent to sheep node managent. This means that hot plug/unplug the disk (include faulty disks) to the local sheep won't cause unnecessary object movements between the nodes. 

MD can automatically handle all the cases you can think of, support different sized disks, serve VM IO requests while handling disk failures(or new disk added), several disks fail at the same time, disk failure while plugging new disk, disk failure in tandem with node failure , etc. All MD handling is transparent to VM.  When all the local disks are broken, the local sheep will act as gateway-only node like before.

## Enable MD

MD is built-in function of sheep. You don't need special options to enable it. The recommended way to start sheep with multiple disk is

 ```
$ sheep /path/to/meta-store,/path/to/disk1{,/path/to/disk2,...}
 ```

Meta-store is a single point of failure as our old /store directory for this sheep daemon, we store epoch file and config file in meta-store, so it only needs several Kilo-bytes of storage.  It is recommended to put mete-store to the partition where OS lives.

You can actually put meta-store and object-store in the same path (This is just intended for easy upgrading) if you only specify one path:

 ```
$ sheep /path/to/store # both meta-store and object-store are located in /path/to/store as old sheep
 ```

## Upgrading from old sheep
The easiest way to upgrade sheep to the new sheep with MD feature is 

```
$ collie cluster shutdown
# upgrade the sheep binary 
$ sheep /path/to/store # works as before, put objects and meta data in the same directory.
```

If you want to separate meta-store and object-store as the recommended way, you have to manually doing some objects movement as following steps (suppose we have two sheep started as 'sheep /store1', 'sheep /store2':

<pre>
1. $ collie cluster shutdown
2. $ mkdir /object-store1
3. $ mv /store1/obj/* /object-store1
4. $ sheep /store1,/object-store1
5. repeat {2,3,4} on node2
</pre>

## Hot-plug and Hot-unplug

We expose disks storage by directory so you need to have a file system which supports xattr on the device, this means you cannot plug raw device file.

You can (un)plug one disk or group of disks in to the sheep on the fly by collie command.

```
Three command added:
 $ collie node md info # show information about md of this node
 $ collie node md info --all # show information about md of all nodes
 $ collie node md plug path1{,path2,...} # plug disk(s) into node
 $ collie node md unplug path1{,path2,...} # unplug disk(s) into node
```

## How MD interact with node change events

MD recovery share the same logic of node change events, so the next event(disk or node) will supersede the previous one and multiple events will fold into single event to minimize the objects recovery.

## FAQ
 1. Can I stop automatic disk recovery temporarily for replacement of disk on the fly ?
  A: Sure. You can stop automatic recover by

```
 $ collie cluster recover disable # stop the recovery if node or disk is failed
 $ #do what you want to do#
 $ collie cluster recover enable # enable the automatic recovery  
```
 
  In this way, you can minimize the unnecessary objects ping-pong between disks or nodes