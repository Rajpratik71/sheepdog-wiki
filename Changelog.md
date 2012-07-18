# Changelogs

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

