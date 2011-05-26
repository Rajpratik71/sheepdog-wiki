Corosync provides IPC and cluster multicast membership to Sheepdog. It must be configured, and running before Sheepdog will work. 

## Config
Your distribution's default corosync.conf settings may not be compatible with Sheepdog, or may have unneeded components. See below for a basic example configuration. 
```
# Please read the corosync.conf 5 manual page
compatibility: whitetank
totem {
  version: 2
  secauth: off
  threads: 0
  # Note, fail_recv_const is only needed if you're 
  # having problems with corosync crashing under 
  # heavy sheepdog traffic. This crash is due to 
  # delayed/resent/misordered multicast packets. 
  # fail_recv_const: 5000
  interface {
    ringnumber: 0
    bindnetaddr: -YOUR IP HERE-
    mcastaddr: 226.94.1.1
    mcastport: 5405
  }
}
logging {
  fileline: off
  to_stderr: no
  to_logfile: yes
  to_syslog: yes
  # the pathname of the log file
  logfile: /var/log/cluster/corosync.log
  debug: off
  timestamp: on
  logger_subsys {
    subsys: AMF
    debug: off
  }
}
amf {
  mode: disabled
}
```

To access corosync daemon without root priviledge, you need to create a file in /etc/corosync/uidgid.d/.
If your username and groupname are 'sdog', write below to /etc/corosync/uidgid.d/sdog
```
uidgid {
   uid: sdog
   gid: sdog
}
```
