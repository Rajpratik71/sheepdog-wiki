## Quick start guide to use sheepdog images from libvirt

1. prepare a file containing an XML domain description
```
$ cat > sheepdog.xml
<domain type='qemu'>
  <name>testvm</name>
  <memory>1048576</memory>
  <os>
    <type arch='x86_64'>hvm</type>
  </os>
  <devices>
    <disk type='network'>
      <source protocol="sheepdog" name="testvdi"/>
      <target dev='hda' bus='ide'/>
    </disk>
    <graphics type='vnc' port='-1' autoport='yes'/>
  </devices>
</domain>
```

2. boot from testvdi with virsh
```
 $ virsh create sheepdog.xml
```

3. connect to a VNC console of the running VM
```
 $ vncviewer localhost
```
