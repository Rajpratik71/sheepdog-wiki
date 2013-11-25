# Introduction
  The journal mechanism is aimed to help keep on-disk data consistent after crash as well as boost write performance.

   Journaling in Sheepdog is simply log all the IO operations in a sequential manner in a dedicated place (be it on the same device as backend store or a different device) before any write to the inode(vdi) object and data objects. Journaling is performed on the node basis.
   
   For crash recovery, we can simply replay the log to restore the node state into the last valid write before the crash. If we use a dedicated device to host journaling, we can get a rather big performance boost because we log the write sequentially, we have turned random writes into sequential writes, which is much more faster on rotating devices. Because of logging ahead the writes, we are safe to drop O_DSYNC for backend writes, which boost a performance a lot (nearly up to 80 times faster write on my SAS disks machine test).

# Usage
   Because of internal design, you need to pass the path of a
directory instead of a file entry. This means we can't operate on raw
device file

     $ sheep -j dir=/path/to/dir,size=256M, # enable external journaling with the size 256M
     $ sheep -j dir=/path/to/dir,size=256M,skip #like above, but skip recovery at startup
     $ sheep -j size=512M # enable internal journaling with the size 512M

   Note 'size' is mandatory parameter to enable journaling.