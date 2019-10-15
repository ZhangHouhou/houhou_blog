# HashTable_详解

### 简述

- 实现了什么接口
  - 继承Dictionary 和 Map接口
  - Cloneable
  - Serializable
- 核心内容
  - Hashtable 使用 **synchronized** 来进行同步
  - Hashtable **不**可以插入``key``和``value``为 **null** 的 Entry
  - 数据结构：拉链法实现的hash表与hashmap类似
- 优缺
  - 它是遗留类，不应该去使用它。现在可以使用 ConcurrentHashMap 来支持线程安全，并且 ConcurrentHashMap 的效率会更高

### 重要的属性

- 默认常量

```java
    /**
     * The hash table data.
     */
    private transient Entry<?,?>[] table;

    /**
     * The total number of entries in the hash table.
     */
    private transient int count;

    /**
     * The table is rehashed when its size exceeds this threshold.  (The
     * value of this field is (int)(capacity * loadFactor).)
     *
     * @serial
     */
    private int threshold;

    /**
     * The load factor for the hashtable.
     *
     * @serial
     */
    private float loadFactor;

    /**
     * The number of times this Hashtable has been structurally modified
     * Structural modifications are those that change the number of entries in
     * the Hashtable or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the Hashtable fail-fast.  (See ConcurrentModificationException).
     */
    private transient int modCount = 0;

```



### 方法

- put()

  ```java
  // Neither the key nor the value can be null 
  // key value 都不能为 null
  public synchronized V put(K key, V value) {
          // Make sure the value is not null
          if (value == null) {
              throw new NullPointerException();
          }
  
          // Makes sure the key is not already in the hashtable.
          Entry<?,?> tab[] = table;
          // key若为空  会抛异常
          int hash = key.hashCode();
          int index = (hash & 0x7FFFFFFF) % tab.length;
          @SuppressWarnings("unchecked")
          Entry<K,V> entry = (Entry<K,V>)tab[index];
          for(; entry != null ; entry = entry.next) {
              if ((entry.hash == hash) && entry.key.equals(key)) {
                  V old = entry.value;
                  entry.value = value;
                  return old;
              }
          }
  
          addEntry(hash, key, value, index);
          return null;
      }
  ```

  
