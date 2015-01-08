The Apache ZooKeeper system for distributed coordination is a high-performance service for building distributed applications
##download & install
download zookeeper package from:
```
wget http://apache.fayea.com/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
```
extend the package 
```
tar zxvf zookeeper-3.4.6.tar.gz ~/
```
install c binding
```
cd ~/zookeeper-3.4.6/src/c/
./configure&&make&&make install
```
##config
config PATH
```
PATH=$PATH:~/zookeeper-3.4.6/bin/
```
Directory creation
```
mkdir /var/lib/zookeeper
```
config service
```
cp ~/zookeeper-3.4.6/conf/zoo_sample.cfg ~/zookeeper-3.4.6/conf/zoo.cfg ##cp zoo.cfg
```
edit zoo.cfg
```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/var/lib/zookeeper
# the port at which the clients will connect
clientPort=2181 
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.1=##zookeeper server1 ip##:2888:3888 ##2888 is Quorum Port and 3888 is Leader Election Port
server.2=##zookeeper server2 ip##:2888:3888
server.3=##zookeeper server3 ip##:2888:3888
```
create myid file 
server1:
```
touch /var/lib/zookeeper/myid
echo 1>/var/lib/zookeeper/myid
```
server2:
```
touch /var/lib/zookeeper/myid
echo 2>/var/lib/zookeeper/myid
```
...

##startup zookeeper service
```
zkServer.sh start
```
##startup sheep service
```
sheep -c zookeeper:IP1:PORT1,IP2:PORT2,IP3:PORT3 ... ##PORT is ClientPort config in the zoo.cfg
```
