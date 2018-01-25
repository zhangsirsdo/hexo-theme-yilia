����о���һ��java�бȽϳ�����map���ͣ���Ҫ��HashMap,HashTable,LinkedHashMap��concurrentHashMap���⼸��map�и��Ե����Ժ����ó�����ʹ�÷����Ļ����Ͳ�˵�ˣ������ص������ԭ��͵ײ��ʵ�֡������еĴ�����Դ��jdk1.9�汾��
# HashMap�ص㼰ԭ�����
## �ص�
HashMap��java��ʹ����ΪƵ����map���ͣ����дЧ�ʽϸߣ�������Ϊ���Ƿ�ͬ���ģ�����д�Ȳ�������û���������ģ������ڶ��̳߳������ǲ���ȫ�ģ����׳������ݲ�һ�µ����⡣�ڵ��̳߳����·ǳ��Ƽ�ʹ�á�
## ԭ��
HashMap������ṹ������ͼ��ʾ��
![ͼƬ����][1]

����ͼƬ���Ժ�ֱ�۵Ŀ�����HashMap��ʵ�ֲ����ѣ���������������������ݽṹ��϶��ɵģ���ڵ����;�Ϊ��ΪEntry��class����߻��Entry�����⣩�������������ݽ���������ۺ����������ݽ�����ŵ㣬���ܱ��ڶ�ȡ���ݣ�Ҳ�ܷ���Ľ������ݵ���ɾ��

ÿһ����ϣ���ڽ��г�ʼ����ʱ�򣬶�������һ������ֵ��capacity���ͼ������ӣ�loadFactor��������ֵָ�Ĳ����Ǳ����ʵ���ȣ������û�Ԥ����һ��ֵ����ʵ�ı��ȣ��ǲ�С��capacity��2���������ݡ�����������Ϊ�˼����ϣ����������ޣ������ϣ����Ľڵ������ﵽ���������ޣ���ϣ��ͻ�������ݵĲ��������ݵ�����Ϊԭ��������2����Ĭ������£�capacity��ֵΪ16��loadFactor��ֵΪ0.75���ۺϿ���Ч����ռ������ԣ���

* **����д��**����HashMap(String, String)Ϊ����������ÿһ���ڵ㣬��keyֵ����ΪString��valueֵ����ҲΪString�������ϣ���в������ݵ�ʱ�����Ȼ�����keyֵ��hashCode����key.hashCode()������hashCode������ʵ�֣�����Ȥ�����ѿ��Կ�һ��jdk��Դ�루֮ǰ������Ϣ˵��һ���������ʵ������֪ʶ�㣩���÷����᷵��һ��32λ��int���͵�ֵ����int h = key.hashCode()Ϊ������ȡ��h��ֵ֮�󣬻�����key��Ӧ�Ĺ�ϣ���е������λ�ã����㷽������ȡģ���㣬h%table.length����Ϊtable�ĳ���Ϊ2���������ݣ����Կ�����h��table.length-1ֱ�ӽ���λ�����㣬���ǣ�index = h & ��table.length-1�����õ���index���Ƿ��������ݵ�λ�á�
![ͼƬ����][2]
�������������ݣ����п��������������index����ͬ�ģ�����1��17�������index��Ϊ1����ʱ�������hash��ͻ��HashMap�����ϣ��ͻ�ķ�ʽ������ʹ������ÿ�������������index��ͬ�����ݡ�

* **���ݶ�ȡ��**�ӹ�ϣ���ж�ȡ����ʱ���ȶ�λ����Ӧ��index��Ȼ�������Ӧλ�õ������ҵ�keyֵ��hashCode��ͬ�Ľڵ㣬��ȡ��Ӧ��valueֵ��
* **����ɾ����** ��hashMap�У�����ɾ���ĳɱ��ܵͣ�ֱ�Ӷ�λ����Ӧ��index��Ȼ�����������ɾ����Ӧ�Ľڵ㡣��ϣ�������ݵķֲ�Խ���ȣ���ɾ�����ݵ�Ч��Խ��(���ǵ����˳��������ݾ����浽�������У��������������Ӷ�ΪO(1))��

## JDKԴ�����
### ���췽��
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
�ӹ��췽���п��Կ���

*  �����е�initialCapacity�����ǹ�ϣ�����ʵ��С����ʵ�ı��С���ǲ�С��initialCapacity��2���������ݡ�
* ��ϣ��Ĵ�С�Ǵ������޵ģ�����2��30���ݡ�����ϣ��Ĵ�С�������ֵʱ��֮��Ͳ��ٽ������ݣ�ֻ���������в��������ˡ�
### PUT ����

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
���Կ�����

* ����ϣ�����ռ�Ķ��������������ӵ�һ��Ԫ�ش����ģ��������ڹ�ϣ���ʼ����ʱ����еġ�
* �����Ӧindex������ֵΪnull���������indexλ�õĵ�һ��Ԫ�أ���ֱ������tab[i]��ֵ���ɡ�
* �鿴������indexλ�õ�node�Ƿ������ͬ��key��hash����У����޸Ķ�Ӧֵ���ɡ�
* ����������indexλ�õ���������ҵ��˾�����ͬkey��hash��node������ѭ��������value���²������������������Ľ�β����������������һ���ڵ㣬����Ӧ������ӽ�ȥ��
* �������漰����TreeNode��������ʱ�Ȳ���ע��