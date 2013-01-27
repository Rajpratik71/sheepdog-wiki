## Quick start guide
### Boot VMs from sheepdog volumes
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

### Manage sheepdog with libvirt
1. make sure that sheepdog is running
2. create a pool for sheepdog
```
$ cat > pool.xml
      <pool type="sheepdog">
        <name>mysheeppool</name>
        <source>
          <name>mysheeppool</name>
          <host name='localhost' port='7000'/>
        </source>
      </pool>
$ virsh pool-create pool.xml
```
3. create a volume
```
$ cat > vol.xml
       <volume>
         <name>myvol</name>
         <key>sheep/myvol</key>
         <source>
         </source>
         <capacity unit='bytes'>53687091200</capacity>
         <allocation unit='bytes'>53687091200</allocation>
         <target>
           <path>sheepdog:myvol</path>
           <format type='unknown'/>
           <permissions>
             <mode>00</mode>
             <owner>0</owner>
             <group>0</group>
           </permissions>
         </target>
       </volume>
$ virsh vol-create vol.xml
```