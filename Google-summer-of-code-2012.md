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
  * **Skill level:** Medium to High.
  * **Language:** C
  * **Mentor:** Liu Yuan or Kazutaka Morita

 * Add transient failure detection mechanism
  * **Brief explanation:**
  * **Expected results:**
  * **Component:** sheep
  * **Skill level:**
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
