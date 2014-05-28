
# What is the issue in taking snapshots?

If you write some data on your guest and take a snapshot right after, you might not have this data saved in the snapshot as you may expect.
The reason is becasue **data is cached in guest memory**.
What you probably want is:

1. all cached data to be flushed before the snapshot is taken;
2. hold any other write request

**Mind that sheepdog has no control on the guest and it can't force it to sync its data to disk before taking the snapshot.**

# How to achieve the two points above?

Manually: by runnin 'sync' command inside the guest to flush data and/or 'fsfreeze' to hold new write requests.
This method is interactive so you can't script it.

Using **qemu-ga**: this is an agent that allow us to send some kind of requests from the host to the guest.

# Procedure

**The guest must be running qemu-ga.** <br />
It's possible to run it also on windows guests.<br />
The host must use qemu 0.16 or higher.<br />
Install socat on the host.

**Host side:**

run the guest with these options:

<pre>
-chardev socket,path=/tmp/qga.sock,server,nowait,id=qga0
-device virtio-serial
-device virtserialport,chardev=qga0,name=org.qemu.guest_agent.0
</pre>

**Guest side**

`qemu-ga -m virtio-serial -p /dev/virtio-ports/org.qemu.guest_agent.0 &`

**Host side**

<pre>
echo '{"execute":"guest-fsfreeze-freeze"}' | socat unix-connect:/tmp/qga.sock -
dog vdi snapshot yourvdi
echo '{"execute":"guest-fsfreeze-thaw"}' | socat unix-connect:/tmp/qga.sock -
</pre>

You can check the status any time by

`echo '{"execute":"guest-fsfreeze-status"}' | socat unix-connect:/tmp/qga.sock -`