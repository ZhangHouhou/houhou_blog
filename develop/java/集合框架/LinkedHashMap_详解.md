# LinkedHashMap_详解

### 简述

- 实现接口和继承父类
  - HashMap的直接子类
  - Map接口
- 核心内容
  - 在 HashMap 的基础上，采用**双向链表**（doubly-linked list）的形式将所有 entry 连接起来，为保证元素的迭代顺序跟插入顺序相同
  - 如果要将自定义的对象放入到 LinkedHashMap 或 LinkedHashSet 中，需要 重写``hashCode()`` 和 ``equals()`` 方法。
  - 出于性能原因，LinkedHashMap 是**非同步**的（**not synchronized**）,可通过获取：
    ``Map m = Collections.synchronizedMap(new LinkedHashMap(...));``
- 优缺
  - 遍历LinkedHashMap 时只需要遍历存储的双向链表（大小为实际保存元素的size），而跟table的大小无关。

### 源码

- put()方法
  ![](https://images.cnblogs.com/cnblogs_com/houhou929/1561921/o_LinkedHashMap_addEntry.png)
  put()继承自HashMap，其中``afterNodeAccess(e)``和``afterNodeInsertion(evict)``，在LinkedHashMap中给出了实现

    ```java
    //HashMap中的实现
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
  
    //LinkedHashMap中的实现  add新元素后被执行
    void afterNodeInsertion(boolean evict) { // possibly remove eldest
            LinkedHashMap.Entry<K,V> first;
            if (evict && (first = head) != null && removeEldestEntry(first)) {
                K key = first.key;
                removeNode(hash(key), key, null, false, true);
            }
        }
  
    //put重复key，覆盖value后被执行
    //把节点e 拼接到双向链表上去了
    void afterNodeAccess(Node<K,V> e) { // move node to last
            LinkedHashMap.Entry<K,V> last;// 拿到尾结点
            if (accessOrder && (last = tail) != e) {
                LinkedHashMap.Entry<K,V> p =
                    (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
                p.after = null;// 置空e的尾结点
                if (b == null)
                    head = a;
                else
                    b.after = a;
                if (a != null)
                    a.before = b;
                else
                    last = b;
                if (last == null)
                    head = p;
                else {
                    p.before = last;
                    last.after = p;
                }
                tail = p;
                ++modCount;
            }
        }
    ```

