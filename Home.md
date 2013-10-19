# Sheepdog wiki

Sheepdog is a distributed storage system for QEMU and RESTful services. It provides highly available block level storage volumes that can be attached to QEMU-based virtual machines. Sheepdog scales to several hundreds nodes, and supports advanced volume management features such as snapshot, cloning, and thin provisioning. Stuff like volumes, snapshots, QEMU's vm-state (from live snapshot), even ISO files can be stored in the Sheepdog cluster.

Enjoy!

## User Guide

 * [[Getting Started]]
 * [[Cluster Management: good practices]]
 * [[Cluster-Management-Backends-and-dual-NIC]]
 * [[Backend Stores, Object Cache and Disk Cache]]
 * [[Sheepfs]]
 * [[Journaling]]
 * [[Image Backup]]
 * [[Multi disk on Single Node Support]]
 * [[Erasure Code Support]]
 * [[Discard Support]]
 * [[General Protocol Support]]
 * [[OpenStack]]
 * [[libvirt]]
 * [[Log formatter]]
 * [[stable branches]]
 * [[Make package on your own]]

## Release Plan

 * [[release schedule]]

 * [0.7.3](https://github.com/collie/sheepdog/tarball/v0.7.3) is the latest stable version:
   See the [[changelog]] for more details.

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

 * [Overview of Sheepdog (2013-4-27)]
(http://www.slideshare.net/multics/overview-of-sheepdog)