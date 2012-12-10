# Changelogs

## recent versions

* [0.4.0](https://github.com/collie/sheepdog/tarball/v0.4.0):
  - a new store driver 'farm'
  - an object cache support 
  - sheepfs
  - a tracer support
  - support for variable virtual nodes
  - start sheep daemon as gateway
  - cluster driver cleanups
  - object list cache
  - zookeeper cluster driver
  - many bug fixes

* [0.5.0](https://github.com/collie/sheepdog/tarball/v0.5.0) Major features include:
  - manual object recovery
  - incremental backups of VDIs
  - testing framework
  - set virtual node numbers based on free disk spaces
  - reimplement farm over plain store driver
  - sheepdog disk cache

* [0.5.1](https://github.com/collie/sheepdog/tarball/v0.5.1) This release includes several fixes and the following features:
  - Version check and automatic upgrade of sheep store
  - A command line option to bind to a specified interface

* [0.5.2](https://github.com/collie/sheepdog/tarball/v0.5.2) This release fixes a bug of store upgrade and includes several cleanups.

* [0.5.3](https://github.com/collie/sheepdog/tarball/v0.5.3) has some critical bug fixes:
  - collie: exit with an error when there are too many arguments
  - sheep: fix buffer overflow of gateway_forward_request
  - sheep: enable disk cache when creating objects
  - tests: retry collie node list when sheep is not ready
  - sheep: handle older version client correctly

* [0.5.4](https://github.com/collie/sheepdog/tarball/v0.5.4):
  - sheep: add support for using unix domain socket
  - test: some spurious failures
  - vditest: refine so that we can use it for benchmark tests
  - logger: cleanup and improve performance
  - many bug fixes and cleanups

* [0.5.5](https://github.com/collie/sheepdog/tarball/v0.5.5) :
  - journaling supprot
  - one fatal deadlock during node change events fixed.
  - many bug fixes and cleanups

## v0.5.0

- manual object recovery

     We can disable automatic data recovery with
     'collie cluster recover disable'.  This is useful, for example,
     when you reboot a storage node for maintenance and don't want to
     run data recovery during the time.

- incremental backups of VDIs

     This release adds two commands 'vdi backup' and 'vdi restore' to
     collie.  With these commands, you can create incremental backup
     files between snapshots.  It is useful to reduce backup data size.

- testing framework
     We add a xfstests-style test suite in this release.  With this
     framework, we've fixed a lot of bugs and I believe sheepdog becomes
     much stable.

- set virtual node numbers based on free disk spaces
     With this feature, even if sheepdog cluster doesn't have the same
     sized disks, sheeps can ideally balance objects.

- reimplement farm over plain store driver
     The previous implementation of farm has a problem that sheep blocks
     I/O long time during object recovery.  This fixes it completely and
     makes farm implementation much simpler.

- sheepdog disk cache
     This is yet another write cache implementation.  With this feature,
     Sheepdog shows a good performance when the number of nodes is not
     big (less than 10 nodes).


## v0.4.0

- farm storage driver

  Sheepdog storage driver is replaced with the farm driver, which gives
  us much more benefits than the original simple store driver.  For more
  information, visit the below links:

    https://github.com/collie/sheepdog/wiki/Backend-Stores-and-Object-Cache

    https://github.com/collie/sheepdog/blob/master/doc/farm-internal.txt


- object cache

  Object cache is the first cache implementation of Sheepdog.  It caches
  object data on the gateway node, and the cached data will be
  replicated to storage nodes when the client sends a flush request.

  Object cache is something like a client-side caching, so please note
  that if cached data is not flushed, other clients on different
  gateways cannot see the latest data.  If you keep in mind it, the
  feature will really helpful to boost your VM performance.

  See also:
    https://github.com/collie/sheepdog/wiki/Backend-Stores-and-Object-Cache


- sheepfs

  With this feature, we can access and control Sheepdog with a
  filesystem interface.  You need to install libfuse development headers
  to compile sheepfs.  For more information, see also:
    https://github.com/collie/sheepdog/wiki/Sheepfs


- gateway node

  When you want to split Sheepdog nodes and VM nodes, it is not
  recommended to connect from qemu to one of sheep nodes directly
  because the connected node can be a SPOF.  The gateway node is
  necessary for such users.  If you run gateway sheeps on VM nodes and
  qemu connects to the local gateway sheep, no storage node can be a
  SPOF.


- object list cache

  When sheep stores lots of objects, it would be a bottleneck of object
  recovery to enumerate all objects.  The object list cache caches the
  list of stored objects and improves object recovery performance.


- cluster driver cleanups

  Cluster driver interface is largely redesigned and refined, and lots
  of relevant bugs are fixed from the previous version.


- zookeeper cluster driver

  Zookeeper driver is completely rewritten by Yunkai, and Taobao guys
  show that we can run Sheepdog with 1,000 nodes with ZooKeeper.


- many bug fixes

  There are so many bug fixes from 0.3.0.  See commit logs for more
  details.

