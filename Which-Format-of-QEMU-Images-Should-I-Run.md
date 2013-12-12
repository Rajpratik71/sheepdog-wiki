Sheepdog block driver of QEMU is implemented at a protocol layer, the lowest layer in the QEMU software. This is similar to QEMU's NBD but end up more powerful. Sitting at the first floor, we get the benefit that we can store whatever formats we want and many fancy features like live migration, snapshot, clone is supported natively by the protocol. This means you can not only store 'raw'(our default format) image in the sheepdog to enjoy best performance but also enjoy advanced features like snapshot, clone, thin provision(some other protocols like 'file' doesn't support) with 'raw' format.

But for any reason you want to run other formats like qcow2(which supports encryption and compression that sheepdog don't) on sheepdog, you can just do it to enjoy the full qcow2 features you desire, at the cost of lower performance compared to 'raw'.

Sitting at protocol layer, you can actually connect to sheepdog volume by other options, such as '-cdrom sheepdog:volume', by which you can store ISOs directly in Sheepdog and boot from it.

To conclude, formats over sheepdog protocol

                    snapshot/clone  | thin-provsion  |   DISCARD    |  encryption  |  compression
 raw over file    :      NO         |      YES       |      NO      |      NO      |       NO
 raw over sheepdog:      YES        |      YES       |      YES     |      NO      |       NO
 qcow2 over sheepdog:    YES        |      YES       |      YES     |      YES     |       YES