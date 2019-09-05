There's nothing special in these scripts.  
They contain a single command with each option on a single line for better reading.  
You may use them as a template.

_run_sheep.sh_

    sheep -n  /var/lib/sheepdog,/mnt/sheep/0 \
    -c zookeeper:192.168.2.45:2181,192.168.2.46:2181,192.168.2.47:2181 \
    -i host=192.168.10.4,port=3333

It uses a single device (but keeping metadata on the system partion);  
zookeeper as cluster manager;  
a dedicated I/O nic.  


_debian_lxde.sh_

    qemu-system-x86_64 \
    -name wheezy_lxde \
    -enable-kvm \
    -drive file=sheepdog:debian_lxde,if=virtio \
    -m 1024 \
    -smp 2 \
    -k us \
    -vga std \
    -vnc :1 \
    -usbdevice tablet \
    -boot order=c \
    -daemonize
