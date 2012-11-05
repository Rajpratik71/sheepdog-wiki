# Sheepdog wiki

Sheepdog is a distributed storage system for QEMU. It provides highly available block level storage volumes that can be attached to QEMU-based virtual machines. Sheepdog scales to several hundreds nodes, and supports advanced volume management features such as snapshot, cloning, and thin provisioning.

Enjoy!

## User Guide

 * [[Getting Started]]
 * [[Cluster Management: good practices]]
 * [[Cluster Management Backends]]
 * [[Backend Stores, Object Cache and Disk Cache]]
 * [[Sheepfs]]
 * [[General Protocol Support]]

## Release Plan

 * [0.5.4](https://github.com/collie/sheepdog/tarball/v0.5.4) is the latest version:
   - sheep: add support for using unix domain socket
   - test: some spurious failures
   - vditest: refine so that we can use it for benchmark tests
   - logger: cleanup and improve performance
   - many bug fixes and cleanups

   See the [[changelog]] for more details.

 * 0.6.0 is planned for the end of november 2012, with:
   - journaling to boost performance
   - sheepkeeper: yet another cluster manager for large-scale cluster
   - object reclaim based on reference counting
   - flexible options to control backend cache
   - variable object size


 * 1.0.0 is expected for the end of the year
 
## Developer Guide
 * [[Sheepdog Design]]
 * [[Submit a Patch]]
 
 * [[Google Summer of Code 2012]]

## External Articles
 * [Setting up a Sheepdog cluster and exporting a volume to a KVM guest -- Daniel P. Berrangé (2011-10-11)](http://berrange.com/posts/2011/10/11/setting-up-a-sheepdog-cluster-and-exporting-a-volume-to-a-kvm-guest/)