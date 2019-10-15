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
                //获取e节点的前置，和后置引用对象
                LinkedHashMap.Entry<K,V> p =
                    (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
                p.after = null;// 置空e的尾结点
                //若e节点没有前置节点，则设e = 头节点 head
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

  ![](https://images.cnblogs.com/cnblogs_com/houhou929/1561921/o_Snipaste_2019-10-14_19-22-55.png)

- 元素删除

  ```java
  //HashMap中的实现
  final Node<K,V> removeNode(int hash, Object key, Object value,
                                 boolean matchValue, boolean movable) {
          Node<K,V>[] tab; Node<K,V> p; int n, index;
          if ((tab = table) != null && (n = tab.length) > 0 &&
              (p = tab[index = (n - 1) & hash]) != null) {
              Node<K,V> node = null, e; K k; V v;
              if (p.hash == hash &&
                  ((k = p.key) == key || (key != null && key.equals(k))))
                  node = p;
              else if ((e = p.next) != null) {
                  if (p instanceof TreeNode)
                      node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                  else {
                      do {
                          if (e.hash == hash &&
                              ((k = e.key) == key ||
                               (key != null && key.equals(k)))) {
                              node = e;
                              break;
                          }
                          p = e;
                      } while ((e = e.next) != null);
                  }
              }
              if (node != null && (!matchValue || (v = node.value) == value ||
                                   (value != null && value.equals(v)))) {
                  if (node instanceof TreeNode)
                      ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                  else if (node == p)
                      tab[index] = node.next;
                  else
                      p.next = node.next;
                  ++modCount;
                  --size;
                  afterNodeRemoval(node);//多态调用LinkedHashMap的实现
                  return node;
              }
          }
          return null;
      }
  
  //把从hashmap中删除的元素，断开双向链表的连接
  void afterNodeRemoval(Node<K, V> e) { // unlink
      TestLinkedHashMap.Entry<K, V> p =
              (TestLinkedHashMap.Entry<K, V>) e, b = p.before, a = p.after;
      p.before = p.after = null;
      if (b == null)
          head = a;
      else
          b.after = a;
      if (a == null)
          tail = b;
      else
          a.before = b;
  }
  ```

- 元素修改和查找

  对于查找get元素，这里linkedhashmap就直接重写了，但是里面调用getnode其实还是hashmap那一套，不过就是多了个判断accessOrder参数，如果为true就会调用afterNodeAccess

- containsValue()

  ```java
  // 查询是否包含 循环遍历双向链表
  public boolean containsValue(Object value) {
      for (TestLinkedHashMap.Entry<K,V> e = head; e != null; e = e.after) {
          V v = e.value;
          if (v == value || (value != null && value.equals(v)))
              return true;
      }
      return false;
  }
  
  // 遍历hashTbable(数组 + 链表)
  public boolean containsValue(Object value) {
          Node<K,V>[] tab; V v;
          if ((tab = table) != null && size > 0) {
              //先循环hash桶
              for (int i = 0; i < tab.length; ++i) {
                  //然后遍历链表
                  for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                      if ((v = e.value) == value ||
                              (value != null && value.equals(v)))
                          return true;
                  }
              }
          }
          return false;
      }
  ```

- 缓存LRU CACHE -- 要实现lru首先要明白这是个什么？

- 近期最少使用算法。 内存管理的一种页面置换算法，对于在内存中但又不用的数据块（内存块）叫做LRU，操作系统会根据哪些数据属于LRU而将其移出内存而腾出空间来加载另外的数据。

  为什么用linkedhashmap呢？因为这个容器事实是一个双向链表，而且里面带上参数的构造函数的时候，前面用的get方法会调用到afterNodeAccess方法，这个方法会吧最近get的数据重新指引向链表末尾

  基于这点我们只要吧accessOrder设置为true即可

  ```java
  package y2019.collection.map;
  
  import java.util.Iterator;
  import java.util.LinkedHashMap;
  import java.util.Map;
  import java.util.Set;
  
  /**
   * @ProjectName: cutter-point
   * @Package: y2019.collection.map
   * @ClassName: TestLRUCache
   * @Author: xiaof
   * @Description: 实现lru (最近最不常使用）缓存
   * 获取数据（get）和写入数据（set）。
   * 获取数据get(key)：如果缓存中存在key，则获取其数据值（通常是正数），否则返回-1。
   * 写入数据set(key, value)：如果key还没有在缓存中，则写入其数据值。
   * 当缓存达到上限，它应该在写入新数据之前删除最近最少使用的数据用来腾出空闲位置。
   * @Date: 2019/9/3 16:42
   * @Version: 1.0
   */
  public class TestLRUCache<K, V> {
  
      LinkedHashMap<K, V> cache = null;
      int cacheSize;
  
      public TestLRUCache(int cacheSize) {
          //默认负载因子取0.75
          this.cacheSize = (int) Math.ceil(cacheSize / 0.75f) + 1;//向上取整数
          cache = new LinkedHashMap<K, V>(this.cacheSize, 0.75f, true) {
              @Override
              protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                  //这里有个关键的负载操作，因为是lru，所以当长度超了的时候，不是扩容，而是吧链表头干掉
                  System.out.println("size=" + this.size());
                  return this.size() > cacheSize;
              }
          };
      }
  
      public V get(K key) {
          return cache.get(key);
      }
  
      public V set(K key, V value) {
          return cache.put(key, value);
      }
  
      public void setCacheSize(int cacheSize) {
          this.cacheSize = cacheSize;
      }
  
      public void printCache(){
          for(Iterator it = cache.entrySet().iterator(); it.hasNext();){
              Map.Entry<K,V> entry = (Map.Entry<K, V>)it.next();
              if(!"".equals(entry.getValue())){
                  System.out.println(entry.getKey() + "\t" + entry.getValue());
              }
          }
          System.out.println("------");
      }
  
      public void PrintlnCache(){
          Set<Map.Entry<K,V>> set = cache.entrySet();
          for(Map.Entry<K,V> entry : set){
              K key = entry.getKey();
              V value = entry.getValue();
              System.out.println("key:"+key+"value:"+value);
          }
  
      }
  
      public static void main(String[] args) {
          TestLRUCache<String,Integer> lrucache = new TestLRUCache<String,Integer>(3);
          lrucache.set("aaa", 1);
          lrucache.printCache();
          lrucache.set("bbb", 2);
          lrucache.printCache();
          lrucache.set("ccc", 3);
          lrucache.printCache();
          lrucache.set("ddd", 4);
          lrucache.printCache();
          lrucache.set("eee", 5);
          lrucache.printCache();
          System.out.println("这是访问了ddd后的结果");
          lrucache.get("ddd");
          lrucache.printCache();
          lrucache.set("fff", 6);
          lrucache.printCache();
          lrucache.set("aaa", 7);
          lrucache.printCache();
      }
  
  
  }
  ```

   