The architecture of Sheepdog is fully symmetric; there is no central
node such as a meta-data server. This design enables following
features.

* Linear scalability in performance and capacity  
When more performance or capacity is needed, Sheepdog can be grown
linearly by simply adding new machines to the cluster.

* No single point of failure  
Even if a machine fails, the data is still accessible through other
machines.

* Easy administration  
There is no config file about cluster's role. When administrators
launch the Sheepdog daemon at the newly added machine, Sheepdog
automatically detects the added machine and begins to configure it as
a member of the storage system.


## Architecture Overview

[[architecture.png]]
Sheepdog is a distributed storage system and provides an object
storage (something like simple key-value interface) to Sheepdog client
(QEMU block driver).  See the later sections for more details of each
Sheepdog component.

* Object storage  
Sheepdog is not general file system.  The Sheepdog daemons create a
distributed object storage system for QEMU (The daemon name of
Sheepdog is ''sheep'').  We can store ''object'' to the storage
system.  A object is flexible-sized data and has a globally unique
identifier.  We can do read/write/create/delete operations to objects
by specifying its identifier.  The object storage is composed of
''gateway'' and ''object manager''.

* Gateway  
The gateway receives I/O requests (object id, offset, length, and
operation type) from the QEMU block driver, calculates the target
nodes based on the consistent hashing algorithm, and forwards the I/O
requests to the target nodes.

* Object manager  
The object manager receives the forwarded I/O requests from the
gateway, and executes read/write operations to its local disk.

* Cluster manager  
The cluster manager manages node membership (detection of failed/added
nodes and notification of node membership changes) and some operations
which requires consensus between all nodes (vdi creation, snapshoting
vdi, etc).  Currently, we uses corosync cluster engine for the cluster
manager.

* QEMU block driver  
The QEMU block driver divides a VM image into fixed-sized objects (4 MB
by default) and store them on the object storage via its gateway.


## Object Storage
Each object is identified by globally unique 64 bit integer, and
replicated to multiple nodes.  QEMU block driver doesn't care about
where to store the objects.  The object storage system is responsible
for managing where to place objects.

### object types
Sheepdog objects are grouped into four types.

* data object  
This contains the actual data of virtual disk images.  The virtual
disk images are divided into fixed-sized data objects.  Sheepdog
client basically accesses this object.

* vdi object  
This contains the metadata of virtual disk images (e.g. image name,
disk size, creation time, data object IDs belonging to the vdi, etc)

* vmstate object  
This stores the vm state image of running VM, which is used when the
administrator takes live snapshot.

* vdi attr object  
We can store attributes for each VDI.  This object stores them.  The
attribute is key-value style, something like an extend attribute of a
regular file.

### object ID rules
*  0 - 31 (32 bits): object type specific space
* 32 - 55 (24 bits): vdi id
* 56 - 59 ( 4 bits): reserved
* 60 - 63 ( 4 bits): object type identifier

Each VDI has a globally unique ID (vdi id), which is calculated from
the hash value of the VDI name.  The usage of lower 32 bits is as
follows:

|**object type**|**the usage of lower 32 bits**|
|:-----------|:------------|
|data object|the index number in the virtual disk image|
|vdi object|not used (filled with zero)|
|vmstate object|the index number in the vm state image|
|vdi attr objects|the hash value of the key name|

### object format
* data object  
the chunk of the virtual disk image
* vdi object
<pre>
&nbsp;struct sheepdog_inode {
&nbsp;    char name[SD_MAX_VDI_LEN];              /* the name of this VDI*/
&nbsp;    char tag[SD_MAX_VDI_TAG_LEN];           /* the snapshot tag name */
&nbsp;    uint64_t ctime;                         /* creation time of this VDI */
&nbsp;    uint64_t snap_ctime;                    /* the time snapshot is taken */
&nbsp;    uint64_t vm_clock_nsec;                 /* vm clock (used for live snapshot) */
&nbsp;    uint64_t vdi_size;                      /* the size of VDI */
&nbsp;    uint64_t vm_state_size;                 /* the size of vm state (used for live snapshot) */
&nbsp;    uint16_t copy_policy;                   /* reserved */
&nbsp;    uint8_t  nr_copies;                     /* the number of object redundancy */
&nbsp;    uint8_t  block_size_shift;              /* info about the size of the data object */
&nbsp;    uint32_t snap_id;                       /* the snapshot id */
&nbsp;    uint32_t vdi_id;                        /* the vdi id */
&nbsp;    uint32_t parent_vdi_id;                 /* the parent snapshot vdi id of this VDI */
&nbsp;    uint32_t child_vdi_id[MAX_CHILDREN];    /* the children VDIs of this VDI */
&nbsp;    uint32_t data_vdi_id[MAX_DATA_OBJS];    /* the data object IDs this VDI contains*/
&nbsp;};
</pre>
If _snap_ctime_ is non-zero, the VDI is a snapshot.  If the length of
string _name_ is zero, the VDI is deleted.

* vmstate object  
the chunk of the vm state image

* vdi attr object  
The first SD_MAX_VDI_ATTR_KEY_LEN bytes (256 bytes) is the key name of this
attribute.  The rest of the object is the value of this attribute.

### read-only/writable objects
From the view of how to access objects, we can also categorize
Sheepdog objects into two groups.

* read-only object (e.g. data objects of a snapshot VDI)  
one VM can write and read the object but other VMs cannot access.
* writable object  
No VM can write the object but any VMs can read the object.

This means that virtual machines cannot share the same volume at the
same time.  This restriction avoids write-write conflicts and
significantly simplify the implementation of sheepdog storage system.


### other features
Sheepdog object storage can accepts copy-on-write requests; when the
clients send a create and write request, they can also specify the
base object (the source of the CoW operation).  This is used for
snapshot and clone operations.



## Gateway

### where to store objects
Sheepdog uses consistent hashing to decide where to store objects.
Consistent hashing is a scheme that provides hash table
functionality, and the addition or removal of nodes does not
significantly change the mapping of objects. I/O load is balanced
across the nodes by features of hash table.

### replication
Data replication of Sheepdog is simple.  We assume that there is only
one writer, so write collision cannot happen.  Clients can send write
request to the target nodes in parallel, and send read requests to one of
the target nodes if clients order the I/O requests in themselves.


### write I/O flow
The gateway calculates the target nodes with consistent hashing, and
sends write requests to all of the target nodes.  The write request is
successful only when all replicas can be successfully updated.
It is because if one of the replicated objects are not updated, the gateway
could read the old data from the not-updated object for the next time.

### read I/O flow
The gateway calculates the target nodes with consistent hashing,
and sends a read request to one of the target nodes.

* fix object consistency  
The consistency of the replication could be broken if the node crash
during forwarding write I/O requests, so the gateway tries to fix the
consistency when it reads object for the first time; read a whole
object data from one of the target node and overwrite all replicas
with it.

### retrying I/O requests
Sheepdog stores all node membership histories.  We call the version
number of the histories ''epoch'' (See also the 'Object Recovery'
section).  When the gateway forwards I/O requests to the target node and
the latest epoch number doesn't match between the gateway and the
target node, the I/O requests fail and the gateway retries the requests until
the epoch numbers match.  This is necessary to keep a strong
consistency of replicated objects.

I/O retry can also happen when the target nodes are down and failed to
complete I/O operations.

## Object Manager
The object manager stores objects to the local disk.
Currently, it stores
each object as one file.  This is quite simple.  We could use other
DBMS (like BerkeleyDB, Tokyo Cabinet, etc) as the object manager, but not
supported now.

### path name rule
Objects are stored in the following path:

<pre>
/store_dir/obj/[epoch number]/[object ID]
</pre>

All object files also have an extended attribute _sheepdog.copies_,
which specifies the number of the object redundancy.

### write journaling
When the sheep daemon fails during write operations, objects could be
partially updated.  Basically, this is not a problem because if the
VM doesn't receive the success, there is no guarantee about the
content of the written sectors.  However, with regards to vdi objects,
we must update in all or nothing way because if the vdi objects are updated
partially, the metadata of VDI could broken.  To avoid
this problem, we use journal for write operations against vdi
objects.  The processes of journaling is quite simple.

1. create a journal file "/store_dir/journal/[epoch]/[vdi object id]"
1. write a data to the journal file first
1. write a data to the vdi object
1. remove the journal file

## Cluster Manager
In most cases, Sheepdog clients can access their images independently
because we do not allow for clients to access the same image at the
same time.  But some VDI operations (e.g. cloning VDI, creating VDI)
must be done exclusively because the operations update global
information.  To implement this without central servers, we uses Corosync cluster engine.

In future, we'll extend Sheepdog to support other systems as the
cluster manager.

(this section is under construction)


## QEMU Block Driver

[[volume.png]]
Sheepdog volumes are divided into 4 MB data objects.  The object
of newly created volume are not allocated at all, that is, only
written objects are allocated.

### open
First of all, QEMU block driver reads a vdi object from the Object
storage through the gateway in bdrv_open().

### read/write
The block driver calculates the data object id from the requested sector offset and size,
and sends requests to the gateway.
When the block driver sends write requests to the data object which is not belongs to the current VDI,
the block driver sends CoW requests to allocate a new data object.

### write to snapshot vdi
We can attach a snapshot VDI to the QEMU.
When the block driver sends the write request to the snapshot VDI for
the first time, the block driver creates a new writable VDI as a child
of the snapshot, and sends requests against the new VDI.

## VDI Operations
### lookup
When looking up the VDI object,

1. calculate a vdi id from the hash value of the vdi name
1. calculate a vdi object id from the vdi id
1. send a read request to the vdi object
1. if the vdi is not the requested one, increment the vdi id and retry to send a read request

### snapshot, cloning
Snapshot and cloning operation is quite simple:
1. read a target VDI
1. create a new VDI which has the same content as the target object VDI
1. set the ''parent_vdi_id'' of the new VDI to the target VDI id
1. set the ''child_vdi_id'' of the target VDI to the new VDI id
1. set the ''snap_ctime'' of the target VDI to the current time
then, the new vdi becomes the current vdi object.

When we create a snapshot of the running VM (live snapshot), we also
stores the vm state info to the vmstate objects.

### delete

TODO: currently, reclaiming of unused data objects is not invoked
until all relevant VDI objects (all relative snapshot VDIs and cloned
VDIs) are deleted.

After all relevant VDIs are deleted, Sheepdog deletes all data objects
of the VDIs, and set the null string to the name of the vdi objects.

## Object Recovery
### epoch
Sheepdog stores histories of node membership in the stored directory.
The path name is
<pre>
/store_dir/epoch/[epoch number]
</pre>
Each file contains the list of nodes info (IP address, port number,
the number of virtual nodes) at the epoch.

### recovery process
1. Receive all stored object IDs from all nodes
1. Calculate which objects to have
1. Create the object IDs list file "/store_dir/obj/[the current epoch]/list"
1. Send a read requests to get objects whose ID is in the list file.  The requests are sent to the node which had the object at the previous epoch.
1. Store the object to the current epoch directory

### conflicts I/Os
If QEMU sends I/O requests against objects which are not recovered
yet, Sheepdog blocks the requests and recovers the objects
preferentially.

## Protocol
All requests of Sheepdog consist of the fixed-sized header part (48 bytes)
and the flexible-sized data part.  The header includes a protocol
version, an operation code, an epoch number, a length of the data, etc.

### between sheep and QEMU
|**operation code**|**description**|
|:-----------|:------------|
|SD_OP_CREATE_AND_WRITE_OBJ|Sends a request to create a new object and write data to it.  If the object already exists, this operation fails.|
|SD_OP_READ_OBJ|Sends a request to read data from a object.|
|SD_OP_WRITE_OBJ|Sends a request to write data to a object.  If the object does not exist, this operation fails|
|SD_OP_NEW_VDI|Sends a vdi name to the object storage and create a new vdi object. This returns the unique vdi id of the vdi in the response.|
|SD_OP_LOCK_VDI|Same as SD_OP_GET_VDI_INFO.|
|SD_OP_RELEASE_VDI|Not used now.|
|SD_OP_GET_VDI_INFO|Get information about the vdi (e.g. vdi id).|
|SD_OP_READ_VDIS|Get the list of vdi IDs which are already used.|
### between sheep and collie
|**operation code**|**description**|
|:-----------|:------------|
|SD_OP_DEL_VDI|Delete the requested VDI.|
|SD_OP_GET_NODE_LIST|Get the list of sheepdog nodes.|
|SD_OP_GET_VM_LIST|Not used now.|
|SD_OP_MAKE_FS|Create a sheepdog cluster.|
|SD_OP_SHUTDOWN|Stop a sheepdog cluster.|
|SD_OP_STAT_SHEEP|Get information about local disk usage.|
|SD_OP_STAT_CLUSTER|Get information about the sheepdog cluster|
|SD_OP_KILL_NODE|Abort the sheep daemon.|
|SD_OP_GET_VDI_ATTR|Get a vdi attr object id.|
### between sheeps
| **operation code**| **description** |
|:-----------|:------------|
|SD_OP_REMOVE_OBJ|Removes the object.|
|SD_OP_GET_OBJ_LIST|Get the list of object IDs which are stored on the target node.|

