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

The latest pulished version is the [v0.3.0](https://github.com/collie/sheepdog/tarball/v0.3.0).

 * 0.4.0 is due to the end of june 2012, with:
  
   - a new store driver 'farm' (done)
   - en object cache support (done)
   - a tracer support (done)
   - support for variable virtual nodes (done)
   - start sheep daemon as gateway (done)
   - support larger object size
   - use differential copy for fast object recovery
   - add hierarchical zones support
   - reclaim unused objects automatically
   - more doc
   - more tests
  
 * 1.0.0 is expected for the end of the year
 
## Developer Guide
 * [[Sheepdog Design]]
 * [[Submit a Patch]]
 
 * [[Google Summer of Code 2012]]

## External Articles
 * [Setting up a Sheepdog cluster and exporting a volume to a KVM guest -- Daniel P. Berrangé (2011-10-11)](http://berrange.com/posts/2011/10/11/setting-up-a-sheepdog-cluster-and-exporting-a-volume-to-a-kvm-guest/)
