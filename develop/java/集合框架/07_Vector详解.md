# Vector_详解

### 简述

- 实现了什么接口

  - List 和 抽象类AbstractList
  - Cloneable
  - Serializable
  - RandomAccess（标记随机访问）

- 核心内容

  - stack是它的子类

- 优缺

  - 支持随机访问，自动扩容（每次扩大为2倍）
  - Vector 是同步的，因此开销就比 ArrayList 要大，访问速度更慢。最好使用 ArrayList 而不是 Vector，因为同步操作完全可以由程序员自己来控制

- 已经过时

  - **问题：**

    为什么java `Vector`类被认为是一个遗留的，过时的或废弃的类？在并发操作时，使用它是无效的吗？

    如果我不想手动对对象实现同步，只想用一个线程安全的集合而无需创建底层数组的全新副本（如`CopyOnWriteArrayList`一样）。这种情况下，我使用`Vector`合理吗?

    然后就是关于栈的问题，它是Vector的一个子类，我应该用什么代替它？

  - **回答：**

    Vector中对每一个独立操作都实现了同步，这通常不是我们想要的做法。对单一操作实现同步通常不是线程安全的（举个例子，比如你想遍历一个Vector实例。你仍然需要申明一个锁来防止其他线程在同一时刻修改这个Vector实例。如果不添加锁的话

    通常会在遍历实例的这个线程中导致一个`ConcurrentModificationException`）同时这个操作也是十分慢的(在创建了一个锁就已经足够的前提下，为什么还需要重复的创建锁)

    当然，即使你不需要同步，Vector也是有锁的资源开销的。

    总的来说，在大多数情况下，这种同步方法是存在很大缺陷的。正如Mr Brain Henk指出，你可以通过调用`Collections.synchronizedList`来装饰一个集合 -事实上 `Vector` 将“可变数组”的集合实现与“同步每一个方法”结合起来的做法是另一个糟糕的设计；

    各个装饰方法能够更明确的指示其关注的功能实现。

    对于`Stack`这个类-我更乐于使用`Deque/ArrayDeque`来实现

    stackoverflow讨论地址: <http://stackoverflow.com/questions/1386275/why-is-java-vector-class-considered-obsolete-or-deprecated>

#### 源码

- 属性

  ```java
   protected Object[] elementData;
   
   protected int elementCount;
   
   protected int capacityIncrement;
  ```

- 扩容方法

  ```java
  //增加为原来的2倍 or 增加一个 增加量capacityIncrement
  private void grow(int minCapacity) {
          // overflow-conscious code
          int oldCapacity = elementData.length;
          int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                           capacityIncrement : oldCapacity);
          if (newCapacity - minCapacity < 0)
              newCapacity = minCapacity;
          if (newCapacity - MAX_ARRAY_SIZE > 0)
              newCapacity = hugeCapacity(minCapacity);
          elementData = Arrays.copyOf(elementData, newCapacity);
      }
  ```

  
