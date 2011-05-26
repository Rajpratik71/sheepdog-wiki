# General Protocol Support

Currently, there is no direct support for any client other than QEMU,
but there are some means to export Sheepdog volumes using more general
protocols.

## iSCSI
1. Setup a patched version of QEMU to support Sheepdog data preallocation
```
$ git clone git://sheepdog.git.sourceforge.net/gitroot/sheepdog/qemu -b iscsi
$ cd qemu
$ ./configure
$ make
# make install
```

1. Create a image with data preallocation.
```
$ qemu-img create sheepdog:image -o preallocation=data 1G
```

1. Install iSCSI target daemon (tgt) with Sheepdog support
```
$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/tomo/tgt.git -b sheepdog
$ cd tgt
$ make
# make install
```

1. Setup tgt
```
# tgtd
# tgtadm --op new --mode target --tid 1 --lld iscsi -T iqn.2001-04.com.example:storage.sr.rose.sys1.xyz
# tgtadm --op new --mode logicalunit --tid 1 --lun 1 -b image --bstype sheepdog
```

See also http://www.mail-archive.com/sheepdog@lists.wpkg.org/msg00679.html

## NBD
1. Create a Sheepdog image
```
$ qemu-img create sheepdog:image 4G
```

1. Start qemu-nbd on the one of Sheepdog servers
```
$ qemu-nbd sheepdog:image
```
