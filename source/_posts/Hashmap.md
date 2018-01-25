最近研究了一下java中比较常见的map类型，主要有HashMap,HashTable,LinkedHashMap和concurrentHashMap。这几种map有各自的特性和适用场景。使用方法的话，就不说了，本文重点介绍其原理和底层的实现。文章中的代码来源于jdk1.9版本。
# HashMap特点及原理分析
## 特点
HashMap是java中使用最为频繁的map类型，其读写效率较高，但是因为其是非同步的，即读写等操作都是没有锁保护的，所以在多线程场景下是不安全的，容易出现数据不一致的问题。在单线程场景下非常推荐使用。
## 原理
HashMap的整体结构，如下图所示：
![图片描述][1]

根据图片可以很直观的看到，HashMap的实现并不难，是由数组和链表两种数据结构组合而成的，其节点类型均为名为Entry的class（后边会对Entry做讲解）。采用这种数据结果，即是综合了两种数据结果的优点，既能便于读取数据，也能方便的进行数据的增删。

每一个哈希表，在进行初始化的时候，都会设置一个容量值（capacity）和加载因子（loadFactor）。容量值指的并不是表的真实长度，而是用户预估的一个值，真实的表长度，是不小于capacity的2的整数次幂。加载因子是为了计算哈希表的扩容门限，如果哈希表保存的节点数量达到了扩容门限，哈希表就会进行扩容的操作，扩容的数量为原表数量的2倍。默认情况下，capacity的值为16，loadFactor的值为0.75（综合考虑效率与空间后的折衷）。

* **数据写入**。以HashMap(String, String)为例，即对于每一个节点，其key值类型为String，value值类型也为String。在向哈希表中插入数据的时候，首先会计算出key值的hashCode，即key.hashCode()。关于hashCode方法的实现，有兴趣的朋友可以看一下jdk的源码（之前看到信息说有一次面试中问到了这个知识点）。该方法会返回一个32位的int类型的值，以int h = key.hashCode()为例。获取到h的值之后，会计算该key对应的哈希表中的数组的位置，计算方法就是取模运算，h%table.length。因为table的长度为2的整数次幂，所以可以用h与table.length-1直接进行位与运算，即是，index = h & （table.length-1）。得到的index就是放置新数据的位置。
![图片描述][2]
如果插入多条数据，则有可能最后计算出来的index是相同的，比如1和17，计算的index均为1。这时候出现了hash冲突。HashMap解决哈希冲突的方式，就是使用链表。每个链表，保存的是index相同的数据。

* **数据读取。**从哈希表中读取数据时候，先定位到对应的index，然后遍历对应位置的链表，找到key值和hashCode相同的节点，获取对应的value值。
* **数据删除。** 在hashMap中，数据删除的成本很低，直接定位到对应的index，然后遍历该链表，删除对应的节点。哈希表中数据的分布越均匀，则删除数据的效率越高(考虑到极端场景，数据均保存到了数组中，不存在链表，则复杂度为O(1))。

## JDK源码分析
### 构造方法
```
    /**
     * Constructs an empty {@code HashMap} with the specified initial
     * capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
```
从构造方法中可以看到

*  参数中的initialCapacity并不是哈希表的真实大小。真实的表大小，是不小于initialCapacity的2的整数次幂。
* 哈希表的大小是存在上限的，就是2的30次幂。当哈希表的大小到达该数值时候，之后就不再进行扩容，只是向链表中插入数据了。
### PUT 方法

```
     /**
     * Associates the specified value with the specified key in this map.
     * If the map previously contained a mapping for the key, the old
     * value is replaced.
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with {@code key}, or
     *         {@code null} if there was no mapping for {@code key}.
     *         (A {@code null} return can also indicate that the map
     *         previously associated {@code null} with {@code key}.)
     */
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

     /**
     * Implements Map.put and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
可以看到：

* 给哈希表分配空间的动作，是向表中添加第一个元素触发的，并不是在哈希表初始化的时候进行的。
* 如果对应index的数组值为null，即插入该index位置的第一个元素，则直接设置tab[i]的值即可。
* 查看数组中index位置的node是否具有相同的key和hash如果有，则修改对应值即可。
* 遍历数组中index位置的链表，如果找到了具有相同key和hash的node，跳出循环，进行value更新操作。否则遍历到链表的结尾，并在链表最后添加一个节点，将对应数据添加进去。
* 方法中涉及到了TreeNode，可以暂时先不关注。