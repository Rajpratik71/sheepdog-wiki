Some people might want to read images out of Sheepdog cluster and store them as local images which can be copied to somewhere else, we can actually convert it by:

    $ qemu-img sheepdog:image -O qcow2 backup_name

Then you can see a qcow2(you can specify other formats) image named backup_name on your local disk.

Besides 'qemu-img convert', we can also use 'vdi read/write/setattr/getattr' the sheepdog images. And we can even export Sheepdog images as a pseudo block file in the local file system hierarchy via [[Sheepfs]] and use common tools like scp to directly transport to wherever you want as a raw image (This will save one extra conversion to local file).

Sheepdog supports yet another backup mechanism: backup differential data between two snapshots. See more info at [incremental snapshot backup](http://comments.gmane.org/gmane.comp.file-systems.sheepdog/6749)