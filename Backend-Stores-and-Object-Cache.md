# Introduction
Every node of Sheepdog cluster has a backend store that provides weighted storage for Sheepdog cluster to store objects (For e.g, VDI and data objects), whose location are determined by consistent hashing algorithm.

Object cache caches data and VDI objects on the local node. It is at higher level than backend store. This extra cache layer translates gateway requests (from VM) into local requests, largely reducing the network traffic and highly improving the IO performance, at the expense of data inconsistency between objects in object cache and backend store. These dirty objects will be flushed to cluster storage by 'sync' request from guest OS.

Let's put it all together from the perspective of the request from VM (with object cache enabled):

IO req (from VM) <--> object cache on local node <--> gateway req <--> targeted sheep backend store

## Farm
Currently Sheepdog supports two kinds of backend store, one is Simple store (default) and the other called Farm.

Simple store, as the name suggests, it simply translate requests into system calls to store data into files backed by local file systems.

Farm is yet another backend store with much more code lines, which is supposed to provide advanced features such as cluster-wide snapshot, faster recovery, better stale object handling, data de-duplication(not implemented yet) and so on, and tries to keep comparable IO performance as Simple store.

Sheepdog user from Taobao.com uses Farm as its default store since its inception and would continue to tune it into better performance and add more features. You can also request any feature you think of proper in the mailing list.

### Cluster snapshot

### Deal with stale objects

## Object Cache