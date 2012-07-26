
## General remarks

Do never kill more than X sheep daemons (X being the number of copies you formatted your cluster with) at a time, or you will **loose your data**. If you need to do some sysadmin task, shutdown gracefully the cluster (all qemu IOs must have been stopped before), using ''collie cluster shutdown''.

At the time of writing this, it is very easy to loose the data stored in a sheepdog cluster. (Hopefully, it will be safer for the 1.0.0 release, and init scripts will be smarter than they are now. Help is always welcome!)


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
