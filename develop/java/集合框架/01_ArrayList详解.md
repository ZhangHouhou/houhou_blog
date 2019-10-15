# ArrayList_详解


### 简述
- 实现了什么接口
  - RandomAccess 支持随机访问(标记接口)
  - List 和 抽象类AbstractList
  - Cloneable
  - Serializable
- 核心内容
  - **线程不同步**。默认初始容量为 10，当数组大小不足时容量扩大为 1.5 倍。为追求效率，ArrayList 没有实现同步（synchronized），如果需要多个线程并发访问，用户可以手动同步，也可使用 Vector 替代
- 优缺
  - 随机访问
  - 删除插入代价大
  - 支持自动扩容

### 源码

```java
/**JDK 1.8.0_201**/
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    // 默认容量
    private static final int DEFAULT_CAPACITY = 10;
    transient Object[] elementData; // non-private to simplify nested class access
    private int size;
    // 数组被修改的次数
    protected transient int modCount = 0;
    // 两个空数组是用来 区分：
    private static final Object[] EMPTY_ELEMENTDATA = {};
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    // 最大size 比Int上限 小 8
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;


    // 构造函数
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    // 固定容量 初始化
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: " +
                    initialCapacity);
        }
    }

    // 现有集合 入参初始化
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }


    // 遍历查找元素  只返回 第一次出现的位置
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i] == null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    // set方法是有返回值的 oldValue
    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

    // 返回被删除的元素
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index + 1, elementData, index,
                    numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

    public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    // 扩容，然后在末端 +元素
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    //若这个list是空参构造初始化的空list，默认容量与minCapacity取大 
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

				//考虑溢出 得出的代码
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
	
	/**
	 * 最关键的扩容
	 * new容量 = old容量 * (1 + 1/2)
	 */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        
        // 正常情况下，1.5倍的扩容 肯定大于 元素个数，肯定不需要频繁的扩容，所以一次来大一点，至于为什么是1.5倍，这个就不知道了。
        // 当 1.5倍扩容发生int的溢出时，就每次进行最小扩容(+1)就可以了。
        // 1.5倍扩容大小  -   元素个数
        if (newCapacity - minCapacity < 0)
        {
        	newCapacity = minCapacity;
        }
        
        if (newCapacity - MAX_ARRAY_SIZE > 0) {
          	newCapacity = hugeCapacity(minCapacity);
        {
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    
    private static int hugeCapacity(int minCapacity) {
    	//溢出的情况
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

		public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
    
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
}

//------- 批量删除 和 序列化方法
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, false);
}			

public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }

private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            //removeAll [0 - w] 存入了 c 中没有的元素 -- 不要被删除的元素
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // 与ArrayList的父类保持一致，当c.contains()这个方法抛出异常才会 r != size
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            //将不删除的存在[0-w]，将[w+1, size-1]的清空，缩小size
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }

private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }


private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
    

}
```


### Tip
- 数组修改操作的 主要方法

    ```java
    public final class System {
        /**
         *  将src的srcPos位置开始，长度为length的部分复制到dest中，从destPos开始覆盖
         */
        public static native void arraycopy(Object src,  int  srcPos,
                                                Object dest, int destPos,
                                                int length);
    }
    ```

- int 加法溢出，位移运算

    ```java
    // Integer.MAX_VALUE + 1 = Integer.MIN_VALUE
    // Integer.MIN_VALUE - 1 = Integer.MAX_VALUE
    // Integer.MIN_VALUE + Integer.MAX_VALUE = -1
            public static void main(String[] args) {
                int maxValue = Integer.MAX_VALUE;
                int minValue = Integer.MIN_VALUE;
    
                System.out.println(" " + maxValue);
                System.out.println(maxValue + 1);
                System.out.println(minValue);
                System.out.println(" " + (minValue - 1));
            }
    
    //输出
     2147483647
    -2147483648
    -2147483648
     2147483647
    
    2147483647  0  -2147483648
    ```

- grow方法中溢出情况的讨论

    ```java
    //溢出情况01   --   if (newCapacity - minCapacity < 0) 
    //当 1.5倍扩容发生int的溢出时，就每次进行最小扩容(+1)就可以了
    
    //溢出情况02
    //数组原大小oldCapacity 已经为int的最大值，此时再次调用grow()扩容minCapacity发生溢出，进入hugeCapacity() 抛出异常。   这是size会溢出的情况，只有size大于nteger.MAX_VALUE - 8 时，才开始考虑是不是抛出溢出异常
    
    public static void main(String[] args) {
            int maxArraySize = Integer.MAX_VALUE - 8;
            int oldCapacity = Integer.MAX_VALUE;
            int minCapacity = oldCapacity + 1;
    
            // overflow-conscious code
            int newCapacity = oldCapacity + (oldCapacity >> 1);
    
            // 1.5倍扩容大小  -   元素个数
            // 正常情况下，1.5倍的扩容 肯定大于 元素个数，肯定不需要频繁的扩容，所以一次来大一点，
                // 至于为什么是1.5倍，这个就不知道了。
            // 当 1.5倍扩容发生int的溢出时，就每次进行最小扩容(+1)就可以了。
            if (newCapacity - minCapacity < 0) {
                newCapacity = minCapacity;
            }
    
            //newCapacity大于maxArraySize 才考虑进入这个方法
            if (newCapacity - maxArraySize > 0) {
                newCapacity = hugeCapacity(minCapacity);
            }
            System.out.println(newCapacity);
        }
        private static int hugeCapacity(int minCapacity) {
            if (minCapacity < 0) // overflow
            {
                throw new OutOfMemoryError();
            }
            return (minCapacity > Integer.MAX_VALUE - 8) ?
                    Integer.MAX_VALUE :
                    Integer.MAX_VALUE - 8;
        }
    }
    ```

- **Fail-Fast**（异常）
    fail-fast 机制在遍历一个集合时，当集合结构被修改，会抛出 **Concurrent Modification Exception**

    fail-fast 会在以下两种情况下抛出 **Concurrent Modification Exception**

    （1）单线程环境

    - 集合被创建后，在遍历它的过程中修改了结构。
    - 注意 remove() 方法会让 expectModcount 和 modcount 相等，所以是不会抛出这个异常。

    （2）多线程环境

    - 当一个线程在遍历这个集合，而另一个线程对这个集合的结构进行了修改。

    **modCount** 用来记录 ArrayList 结构发生变化的次数。结构发生变化是指**添加**或者**删除**至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。

    在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出 **Concurrent Modification Exception**。

    ```java
    private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();
    
        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);
    
        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }
    
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
    ```
