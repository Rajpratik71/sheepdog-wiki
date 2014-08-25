# Sheepdog wiki

Sheepdog is a distributed storage system for QEMU, iSCSI clients and RESTful services. It provides highly available block level storage volumes that can be attached to QEMU-based virtual machines. The volumes can also be attached to other virtual machines and operating systems run on baremetal hardware if they support iSCSI protocol. Sheepdog scales to several hundreds nodes, and supports advanced volume management features such as snapshot, cloning, and thin provisioning. Stuff like volumes, snapshots, QEMU's vm-state (from live snapshot), even ISO files can be stored in the Sheepdog cluster.

Enjoy!

## User Guide

 * [[Getting Started]]
 * **Sheep Daemon Related**
   - [[Cluster Management: good practices]]
   - [[Cluster-Management-Backends-and-dual-NIC]]
   - [[Log formatter]]
 * **Backend Storage Related**
   - [[Backend Stores, Object Cache]]
   - [[Discard Support]]
   - [[Journaling]]
   - [[Multi disk on Single Node Support]]
   - [[Erasure Code Support]]
 * **Other Software Support**
   - [[General protocol support (iSCSI and NBD)]]
   - [[OpenStack]]
   - [[libvirt]]
 * **Other Interface**
   - [[Sheepfs]]
   - [[HTTP Simple Storage]] 
   - [[Sheepdog Block Device (SBD)]]
 * [[Image Backup]]
 * [[stable branches]]
 * [[Make package on your own]]
 * **Advice**
   - [[Why The Performance Of My Cluster Is Bad]]
   - [[Which Format of QEMU Images Should I Run]]
   - [[How to make a live snapshot properly]]
 * **For developers**
   - [[clang analysis of sheepdog]]

## Release Plan

 * [[release schedule]]

 * [0.8.0](https://github.com/collie/sheepdog/tarball/v0.8.0) is the latest stable version:
   See the [changelog](https://github.com/sheepdog/sheepdog/blob/master/CHANGELOG.md#080) for more details.

 * 0.8.0 is planned for the end of 2013.11.15, with:
   - erasure code support
   - MD support unlimited disks and max node number bump to 6k+
   - basic stats added
   - toward better hash performance

## Developer Guide
 * [[Sheepdog Design]]
 * [[Submit a Patch]]
 
 * [[Google Summer of Code 2012]]

## External Articles
 * [Setting up a Sheepdog cluster and exporting a volume to a KVM guest -- Daniel P. Berrangé (2011-10-11)](http://berrange.com/posts/2011/10/11/setting-up-a-sheepdog-cluster-and-exporting-a-volume-to-a-kvm-guest/)

 * [Overview of Sheepdog (2013-04-27)]
(http://www.slideshare.net/multics/overview-of-sheepdog)

* [Introduction Video (2014-08-20)]
(http://youtu.be/zlORo0TETgU)  
[[Scripts Uded In "Introduction Video"]]