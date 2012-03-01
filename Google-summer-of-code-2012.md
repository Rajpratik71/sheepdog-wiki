# Google summer of code 2012

## Project ideas
 *  Create sheepdog client library
  * **Brief explanation:**
  * **Expected results:**
  * **Component:** sheep
  * **Skill level:**
  * **Language:** C
  * **Mentor:** Kazutaka Morita or Liu Yuan

 * Support variable object size
  * **Brief explanation:**
  * **Expected results:**
  * **Component:** sheep
  * **Skill level:**
  * **Language:** C
  * **Mentor:** Kazutaka Morita or Liu Yuan

 * Add writeback support
  * **Brief explanation:**
  * **Expected results:**
  * **Component:** sheep
  * **Skill level:**
  * **Language:** C
  * **Mentor:** Kazutaka Morita or Liu Yuan

 * Get differences between VDIs for efficient backup
  * **Brief explanation:**
  * **Expected results:**
  * **Component:** sheep
  * **Skill level:**
  * **Language:** C
  * **Mentor:** Kazutaka Morita or Liu Yuan

 * Support hierarchical zone
  * **Brief explanation:**
  * **Expected results:**
  * **Component:** sheep
  * **Skill level:**
  * **Language:** C
  * **Mentor:** Kazutaka Morita or Liu Yuan

 * Add LevelDB to backend store
  * **Brief explanation:** 
Every node of Sheepdog cluster has a backend store that provides storage for both VDI and data objects, whose location are determined by consistent hashing algorithm. The implementation of sheep can be logically split into two parts, one is gateway and the other, backend store. There is an abstract interface sitting between gateway and backend store, allowing different backend store implementation. Currently, we have Simple store and Farm store. LevelDB is a fast key-value storage library written at Google that provides an ordered mapping from string keys to string values. Your task is to implement a LevelDB driver, that enables gateway to communicate with LevelDB to store objects on the node basis. You can choose other DBM if you find it more suitable, such as Kyoto Cabinet, BDB.

  * **Expected results:** Objects can be stored reliably to LevelDB. The IO performance and recovery process can be comparable to original stores. The snapshot and any features you think of are optional.
  * **Component:** sheep
  * **Skill level:** Medium to high.
  * **Language:** C
  * **Mentor:** Liu Yuan or Kazutaka Morita

 * Add transient failure detection mechanism
  * **Brief explanation:** Sheepdog cluster uses redundant data to achieve the high availability. When one of nodes fails, the cluster need to restore the replica level and when a node joins, those replica need to rebalance on the nodes. That is, when there happens node change event, those replica distribution need to rebuild. Currently this rebuilding process is triggered immediately by the node change event. That is, the cluster doesn't distinguish between transient and permanent node failure, thus transient node failure will be counted as two node change event, wasting network bandwidth. Your task is to implement transient failure detection mechanism.
  * **Expected results:** The cluster can deal with transient node failure. We might benefit from this mechanism by upgrading sheepdog binary without wasting bandwidth. 
  * **Component:** sheep
  * **Skill level:** Medium to high
  * **Language:** C
  * **Mentor:** Liu Yuan or Kazutaka Morita

 * Add user snapshot support for farm
  * **Brief explanation:** Farm is an object store for Sheepdog on node basis. It consists of backend store, which caches the snapshot objects, and working directory, storing objects that Sheepdog currently operates. Now it has a raw snapshot operation support. Your task is to make use of current low level infrastructure to add user snapshot feature.
  * **Expected results:** Users can snapshot the cluster at any moment. The system state can be restored to the correct state despite of node change events.
  * **Component:** sheep/farm
  * **Skill level:** Medium to high
  * **Language:** C
  * **Mentor:** Liu Yuan or Kazutaka Morita

 * Make Accord more configurable
  * **Brief explanation:**
Currently, we can find so many hard-coded constant values in the source code of accord.  Our task is to make it all configurable. 
  * **Expected results:**
1) Create a configure template file.
2) add some code to parse it and use these configurable options.
  * **Component:** accord
  * **Skill level:** Simple
  * **Language:** C
  * **Mentor:** Yunkai Zhang or Kazutaka Morita

 * Improving Accord logging system
  * **Brief explanation:**
We need a more powerful logging system, it can help me analysis complicated problems, maybe libqb can help me achieve this target, but you can use any library or skill as long as you like.
  * **Expected results:**
1) logging message should have multiple levels and can be output to multiple target, just like log4j.
2) generate core dump file in a specify directory after accord crash, such as received segment fault signal.
  * **Component:** accord
  * **Skill level:** Medium to high
  * **Language:** C
  * **Mentor:** Yunkai Zhang or Kazutaka Morita

 * Add Accord tools to show inner states
  * **Brief explanation:**
Currently, accord just like a black box when it work. We need to  create some tools to help us monitor its inner states, such as all connected client information.
  * **Expected results:**
1) can show us connected client list.
2) can read/search messages stored in accord.
3) ...
  * **Component:** accord
  * **Skill level:** Medium to high
  * **Language:** C
  * **Mentor:** Yunkai Zhang or Kazutaka Morita