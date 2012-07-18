# Sheepdog wiki

Sheepdog is a distributed storage system for QEMU. It provides highly available block level storage volumes that can be attached to QEMU-based virtual machines. Sheepdog scales to several hundreds nodes, and supports advanced volume management features such as snapshot, cloning, and thin provisioning.

Enjoy!

## User Guide

 * [[Getting Started]]
 * [[Cluster Management Backends]]
 * [[Backend Stores and Object Cache]]
 * [[Sheepfs]]
 * [[General Protocol Support]]
 * [[Migration from v0.3.0 (simple) to v0.4.0 (farm) - the right way?]]

## Release Plan

The latest pulished version is the [v0.4.0](https://github.com/collie/sheepdog/tarball/v0.4.0), with:
  
   - a new store driver 'farm'
   - an object cache support 
   - sheepfs
   - a tracer support
   - support for variable virtual nodes
   - start sheep daemon as gateway
   - cluster driver cleanups
   - object list cache
   - zookeeper cluster driver
   - many bug fixes

See [[Changelog]] for more details.

 * 0.5.0 is probably due to the end of september 2012, with:
   - differential backup of VDIs
   - variable object size
   - variable virtual nodes according to the free spaces
   - server-side write cache (write object without O_DSYNC, and sync
     them when qemu sends a flush request.  I think we can use it
     combined with object cache.)
   - redesign threads for better performance
   - testing framework

 * 1.0.0 is expected for the end of the year
 
## Developer Guide
 * [[Sheepdog Design]]
 * [[Submit a Patch]]
 
 * [[Google Summer of Code 2012]]

## External Articles
 * [Setting up a Sheepdog cluster and exporting a volume to a KVM guest -- Daniel P. Berrangé (2011-10-11)](http://berrange.com/posts/2011/10/11/setting-up-a-sheepdog-cluster-and-exporting-a-volume-to-a-kvm-guest/)