# Introduction
Discard command allows an operating system to inform underlying storage drive which blocks of data are no longer considered in use and can be wiped internally. This is one sort of mechanism to provide thin-provisioning, and mostly useful for SSD and Virtual Machines(VM).

Sheepdog support the discard command in a way that objects as the backing storage for discarded space of one particular volume  will be removed from host file system.   The discard command issuer in the VM is file system like EXT4 or XFS which support discard operation. The deletion of one particular file will *transparently* trigger a discard request to the underlying storage system to discard the space that deleted file occupies. Command like mkfs will actually issue a discard request that asks to release the whole storage space.

Even though Sheepdog provides sparse volumes to the VM, the discard can be also useful for the case that, the objects that are allocated for the volume at some time can be de-allocated later if Sheepdog is informed later that those objects are discarded by the VM.

## Enable discard for the VM

Requirements: QEMU version >= 1.5, Kernel version of Guest: vanilla kernel >= 3.4 or RHEL-based kernel >= 6.3, Sheepdog latest from git repo.

For QEMU device, all kinds of IDE and scsi (include virtio-scsi) support discard operation, note virtio-blk isn't supported. Discard support is off as default. You need to explicitly enable it at start-up command by add 'discard=on' for drive parameter. For e.g,

<pre>
# for IDE device backed by sheepdog volume named 'data'
$ qemu-system-x86_64 -drive if=ide,file=sheepdog:data,cache=writeback,discard=on
</pre>

<pre>
# for viriot-scsi device backed by sheepdog volume named 'data'
$ qemu-system-x86_64 -drive if=none,id=disk,file=sheepdog:data,cache=writeback,discard=on -device scsi-hd,drive=disk
</pre>

