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
Every node of Sheepdog cluster has a backend store that provides storage for both VDI and data objects, whose location are determined by consistent hashing algorithm. The implementation of sheep can be logically split into two parts, one is gateway and the other, backend store. There is an abstract interface sitting between gateway and backend store, allowing different backend store implementation. Currently, we have Simple store and Farm store. LevelDB is a fast key-value storage library written at Google that provides an ordered mapping from string keys to string values. Your task is to implement a LevelDB driver, that enables gateway to communicate with LevelDB to store objects on the node basis.

  * **Expected results:** Objects can be stored reliably to LevelDB. The IO performance and recovery process can be comparable to original stores. The snapshot and any features you think of are optional.
  * **Component:** sheep
  * **Skill level:** Medium to high.
  * **Language:** C
  * **Mentor:** Liu Yuan or Kazutaka Morita

 * Add transient failure detection mechanism
  * **Brief explanation:** Sheepdog cluster uses redundant data to achieve the high availability. When one of nodes fails, the cluster need to restore the replica level and when a node joins, those replica need to rebalance on the nodes. That is, when there happens node change event, those replica distribution need to rebuild. Currently this rebuilding process is triggered immediately. That is, the cluster doesn't distinguish the transient and permanent node failure, thus transient node failure will be counted as two node change event, wasting network bandwidth. Your task is to implement transient failure detection mechanism.
  * **Expected results:** The cluster can deal with transient node failure. We might benefit from this mechanism by upgrading sheepdog binary without wasting bandwidth. 
  * **Component:** sheep
  * **Skill level:** Medium to high
  * **Language:** C
  * **Mentor:** Liu Yuan or Kazutaka Morita

 * Add user snapshot support for farm
  * **Brief explanation:**
  * **Expected results:**
  * **Component:** sheep/farm
  * **Skill level:**
  * **Language:** C
  * **Mentor:** Liu Yuan or Kazutaka Morita

 * Make Accord more configurable
  * **Brief explanation:**
  * **Expected results:**
  * **Component:** accord
  * **Skill level:**
  * **Language:** C
  * **Mentor:** Yunkai Zhang or Kazutaka Morita

 * Improving Accord logging system
  * **Brief explanation:**
  * **Expected results:**
  * **Component:** accord
  * **Skill level:**
  * **Language:** C
  * **Mentor:** Yunkai Zhang or Kazutaka Morita

 * Add Accord tools to show inner states
  * **Brief explanation:**
  * **Expected results:**
  * **Component:** accord
  * **Skill level:**
  * **Language:** C
  * **Mentor:** Yunkai Zhang or Kazutaka Morita
