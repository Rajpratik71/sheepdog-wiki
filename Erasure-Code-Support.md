# Introduction
  Erasure Coding(EC) is a redundancy scheme that achieves high available of data with much less storage overhead compared to complete replication. Erasure code is a jargon from information theory, which consist of two processes, decode and encode. When encoding EC takes several source bytes and generate some parity bytes. When decoding, EC generates the missing bytes (can be parity or source bytes) by looking at the remaining source bytes and parity bytes. For sheepdog storage as an example, with the scheme 8 data strips and 3 parity strips specified, we can maintain data integrity while at most 3 nodes fail down in the same time, with just extra (3/8 = 0.375) times of data overhead. For comparison, we need 4 replication copy to achieve the same effect with replication scheme with extra 3x times of data overhead. 

  Based on this property, we can now choose to store data in sheepdog cluster with Reed–Solomon error correction algorithm which allows creating any given number of parity bytes.

# Usage
  Sheepdog support user-define redundancy on a per VDI basis both for replication and erasure coding schemes. To create a erasure coded VDI
  
    $ dog vdi create -c x:y vdi size # Create a erasure coded VDI

  You can also format the cluster to set erasure code as default redundancy policy

    $ dog cluster format -c x:y 

  X represent number of data strips, must be one of {2,4,8,16} while Y represents number of parity strips, can be any number between 1 and 15 inclusive. For example, you can

    $ dog vdi create -c 2:1 test 10G # 0.5 redundancy and can tolerate 1 node failure at the same time
    $ dog vdi create -c 2:3 test 10G # 1.5 redundancy and can tolerate 3 node failure at the same time
    $ dog vdi create -c 4:2 test 10G # 0.5 redundancy and can tolerate 2 nodes failure at the same time
    $ dog vdi create -c 8:3 test 10G # 0.375 redundancy and can tolerate 3 nodes failure at the same time
    $ dog vdi create -c 16:4 test 10G # 0.25 redundancy and can tolerate 4 nodes failure at the same time
    $ dog vdi create -c 16:15 test 10G # 0.9375 redundancy and can tolerate 15 nodes failure at the same time

  To create a fully replicated vdi as before

    $ dog vdi create -c 3 test 10G # 2x redundancy and can tolerate 2 nodes failure at the same time

# Characteristics
  Erasure coding is seamlessly functional with all other features such as snapshot/clone/cluster-wide snapshot/multi-disk/auto-healing e.c with following characteristics:
  
  1. Data is erasure coded automatically while being written to sheepdog storage, no extra operations.
  2. Support random read/write, in-place-update, misaligned read/write
  3. Support to run any type VM images or attach as a vdisk of a VM
  4. User-defined coding scheme on a VDI basis
  5. Better read/write performance compared to fully replication scheme
  6. A single cluster can both support replicated VDI or erasure-coded VDI

# Performance 
  For a simple test on my box, aligned-4k write get 1.5x faster than replication
at most, while read get 1.15x faster, compared with copies=3 (4:2 scheme)
    
  For 6 nodes cluster with 1000Mb/s NIC, I got the following result:
    
    replication(3 copies): write 36.5 MB/s, read 71.8 MB/s
    erasure code(4 : 2)  : write 46.6 MB/s, read 82.9 MB/s

# Relations with least number of nodes to service the request
  You need at least X alive nodes (e.g, 4 nodes in 4:2 scheme) to serve the read/write request. If number of nodes drops to below X, the cluster will deny of service. Note that if you only X nodes in the cluster, it means you don't have any redundancy parity generated.