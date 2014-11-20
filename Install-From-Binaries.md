# Debian

## Wheezy

Add the backports repository

<pre>nano /etc/apt/sources.list
deb http://http.debian.net/debian wheezy-backports main
aptitude update</pre>

<pre>
# Sheedog
aptitude -t wheezy-backports --without-recommends install sheepdog
# Qemu
aptitude -t wheezy-backports install qemu-kvm
# Zookeeper
aptitude install zookeeper zookeeperd
</pre>

# Jessie

<pre>
# Sheedog
aptitude install sheepdog
# Qemu
aptitude qemu-kvm
# Zookeeper
aptitude install zookeeper
</pre>