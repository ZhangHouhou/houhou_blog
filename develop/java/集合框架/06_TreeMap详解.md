# TreeMap_详解

### 简述

- 实现接口和继承父类
  - NavigableMap：意味着它支持一系列的导航方法，具备针对给定搜索目标返回最接近匹配项的导航方法
  - 抽象类AbstractMap：表明TreeMap为一个Map即支持key-value的集合
  - Cloneable
  - Serializable
- 核心内容
  - TreeMap是**非线程安全**的
    可以设置为同步的：``Map m = Collections.synchronizedSortedMap(new TreeMap(…));``
  - TreeMap是用``key``来进行升序顺序来排序的。通过Comparable 或 Comparator来排序
  - 如果在创建（获取）迭代器之后的任何时间对结构进行结构修改，则除了迭代器自己的``{@code remove}``方法，则迭代器将抛出``{@link ConcurrentModificationException}``。
    因此，面对并发修改，迭代器会快速干净地失败，而不会在未来的不确定时间内冒任意，不确定的行为的风险
- 优缺
  - 插入查询性能低于 HashMap

### 源码

- 重要属性

    ```java
    //比较器，因为TreeMap是有序的，通过comparator接口我们可以对TreeMap的内部排序进行精密的控制
    private final Comparator<? super K> comparator;
    
    //TreeMap红-黑节点，为TreeMap的内部类
    private transient Entry<K,V> root = null;
    
    //容器大小
    private transient int size = 0;
    
    //TreeMap修改次数
    private transient int modCount = 0;
    
    //红黑树的节点颜色–红色
    private static final boolean RED = false;
    
    //红黑树的节点颜色–黑色
    private static final boolean BLACK = true;
    ```

- 构造器

  ```java
  	/**
       * 1.插入到映射中的所有键必须实现{@link Comparable}接口
       * 2.所有这些键必须是相互可比的
       * 3.构造一个空tree map，使用key的自然顺序
       */
      public TreeMap() {
          comparator = null;
      }
  
      /**
       * 1.构造一个空tree map，使用key的个特定的顺序 comparator
       *
       * @param comparator the comparator that will be used to order this map.
       *        If {@code null}, the {@linkplain Comparable natural
       *        ordering} of the keys will be used.
       */
      public TreeMap(Comparator<? super K> comparator) {
          this.comparator = comparator;
      }
  
      public TreeMap(Map<? extends K, ? extends V> m) {
          comparator = null;
          putAll(m);
      }
  
      /**
       * 根据有序的Map构造tree map,该方法在线性时间内运行
       */
      public TreeMap(SortedMap<K, ? extends V> m) {
          comparator = m.comparator();
          try {
              buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
          } catch (java.io.IOException cannotHappen) {
          } catch (ClassNotFoundException cannotHappen) {
          }
      }
  ```

- 查询方法

    ```java
    /**
         * Returns the value to which the specified key is mapped,
         * or {@code null} if this map contains no mapping for the key.
         *
         * <p>More formally, if this map contains a mapping from a key
         * {@code k} to a value {@code v} such that {@code key} compares
         * equal to {@code k} according to the map's ordering, then this
         * method returns {@code v}; otherwise it returns {@code null}.
         * (There can be at most one such mapping.)
         *
         * <p>A return value of {@code null} does not <em>necessarily</em>
         * indicate that the map contains no mapping for the key; it's also
         * possible that the map explicitly maps the key to {@code null}.
         * The {@link #containsKey containsKey} operation may be used to
         * distinguish these two cases.
         *
         * @throws ClassCastException if the specified key cannot be compared
         *         with the keys currently in the map
         * @throws NullPointerException if the specified key is null
         *         and this map uses natural ordering, or its comparator
         *         does not permit null keys
         */
        public V get(Object key) {
            Entry<K,V> p = getEntry(key);
            return (p==null ? null : p.value);
        }
    
    
    /**
         * Returns this map's entry for the given key, or {@code null} if the map
         * does not contain an entry for the key.
         *
         * @return this map's entry for the given key, or {@code null} if the map
         *         does not contain an entry for the key
         * @throws ClassCastException if the specified key cannot be compared
         *         with the keys currently in the map
         * @throws NullPointerException if the specified key is null
         *         and this map uses natural ordering, or its comparator
         *         does not permit null keys
         */
        final Entry<K,V> getEntry(Object key) {
            // Offload comparator-based version for sake of performance
            if (comparator != null)
                return getEntryUsingComparator(key);
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            Entry<K,V> p = root;
            while (p != null) {
                int cmp = k.compareTo(p.key);
                if (cmp < 0)
                    p = p.left;
                else if (cmp > 0)
                    p = p.right;
                else
                    return p;
            }
            return null;
        }
    ```

    可以看出查找也是基于二叉查找树的性质，二分法来实现的。插入，删除也都是基于红黑树的结构来实现的，就不赘述。可以参考数据结构-RB树
