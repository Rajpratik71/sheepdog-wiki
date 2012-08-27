## General remarks

Sheepdog uses a zone concept for data replication. In the default
configuration, each node in the cluster is one zone and sheepdog replicate
the vdi objects to the number of zones defined by the number of copies (X).

To keep a working cluster, never kill sheeps in more than X-1 zones at
a time, or during recovery is not finished. If you kill X or more zones
at a time, you will have no access to some of the objects. It never mind,
if you kill the complete zone, or only a few sheeps in X different zones
at a time, there will '''always''' some objects, that are only hold by
these specific sheeps, only the amount of inaccessible objects will differ.

Including up to version 0.4 you will '''loose your data''' in this case!
With the introduction of plain_store and the rework of farm to use it as
core in version 0.5, you wont loose data, but as long as the zones are not
restarted, you cant access the objects that are only available on this
zones and your VM get a I/O Error.

Make sure your UPS auto-shutdown (eg. apcupsd config) scripts do the correct job!  

## Upgrading the nodes

The update scenario depends if you need a running cluster the
whole time, or if you can plan a complete shutdown for some time.

### Without downtime

If you need to run the cluster all the time, you have to:

- kill the sheeps on one node, make the update and restart the sheeps.

- After this, **wait for recovery to complete** and proceed with
the next node. 

- After finishing with all nodes run ''collie cluster cleanup'', this removes obj no longer needed on the
nodes after successful recovery.

### With downtime

If you have a timeframe to shutdown the cluster completely:

- use ''collie cluster shutdown'' (shut down
all connected qemu instances before) to stop all sheeps on all
nodes which leaves the cluster in a clean state,

- then make the updates on all nodes and

- restart the sheeps, the cluster starts working again, if all original inhabitants are
back alive on the farm.

## Migration

 * [[Migration from v0.3.0 (simple) to v0.4.0 (farm) - the right way?]]
