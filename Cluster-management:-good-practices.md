## Environment

Sheepdog often uses more file descriptors than you might expect. The larger the scale of the deployment, the bigger the set of FDs that will be used in any given period of time. So make sure your systems have a high enough limit on FDs:

    $ ulimit -n

If you see '1024' which is the default value on most distributions, set it to a higher value.  For example, 65536.

To help debug crashed Sheepdog instances, please enable your systems to dump corefiles. And most importantly, before you point blame at Sheepdog, cut and paste gdb output from core file (the core file of Sheep is default located at /path/to/store/core) by

    $ gdb /path/to/sheep /path/to/core_file
    $ (gdb console) bt full

## General remarks

Sheepdog uses a "zone" concept for data replication. In the default
configuration, each node in the cluster is one zone and sheepdog replicates
the vdi objects to the number of zones defined by the number of copies (X).

To keep a working cluster, never kill sheeps in more than X-1 zones at
a time, or during recovery is not finished. If you kill X or more zones
at a time, you will have no access to some of the objects. It does not matter
if you kill the complete zone, or only a few sheeps in X different zones
at a time, there will '''always''' some objects, that are only hold by
these specific sheeps, only the amount of inaccessible objects will differ.

Including up to version 0.4 you will '''lose your data''' in this case!
With the introduction of plain_store and the rework of farm to use it as
core in version 0.5, you won't lose data, but as long as the zones are not
restarted, you can't access the objects that are only available on this
zones and your VM get a I/O Error.

Make sure your UPS auto-shutdown (eg. apcupsd config) scripts do the correct job!  

## Upgrading the nodes  (**outdated**, help is needed to work out a better process for latest sheep)

The update scenario depends if you need a running cluster the
whole time, or if you can plan a complete shutdown for some time.

### Without downtime

If you need avoid downtime, you have to:

- Stop the sheeps on one node, perform the update and restart the sheeps.

- After this, **wait for recovery to complete** and proceed with the next node.

- After finishing with all nodes, run ''collie cluster cleanup'', this removes objects no longer needed on the
nodes after a successful recovery.

### With downtime

If you have a maintenance window to shutdown the cluster completely:

- use ''collie cluster shutdown'' (shut down
all connected qemu instances before) to stop all sheeps on all
nodes which leaves the cluster in a clean state, then

- make the updates on all nodes and

- restart the sheeps - the cluster will start working again, if all the original inhabitants are
back alive on the farm.

## Migration

 * [[Migration from v0.3.0 (simple) to v0.4.0 (farm) - the right way?]]