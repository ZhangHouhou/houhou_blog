# LinkedList_详解

### 简述

- 实现了什么接口

  - List、Deque（即能将LinkedList当作双端队列使用） 和 抽象类AbstractSequentialList
  - Cloneable
  - Serializable
- 核心内容

  - **LinkedList 底层是基于双向链表实现的**，也是实现了 List 接口，所以也拥有 List 的一些特点 (JDK1.7/8 之后取消了循环，修改为双向链表) 
- 优缺

  - **不**支持随机访问（get()方法是分半遍历链表查找的）
  - LinkedList 在任意位置添加删除元素更搞笑
  - 查找需要进行遍历查询，效率较低。
  - 继承了Deque类，可以提供stack和queue的功能，但是关于栈或队列，现在的首选是 ArrayDeque，它有着比 LinkedList （当作栈或队列使用时）有着更好的性能
- JDK的描述
  ``{@code List}``和``{@code Deque}``接口的双链列表实现。实现所有可选的列表操作，并允许所有元素（包括**null**）。
         所有操作均按双向链接列表的预期执行。索引到列表中的操作将从开头或结尾遍历列表，以更接近指定索引的位置为准。 
         **请注意，此实现未同步。**如果多个线程同时访问链表，并且至少一个线程在结构上修改了链表，则它必须同步。在外部。 （结构修改是添加或删除一个或多个元素的任何操作；仅设置元素的值不是结构修改。）通常通过对自然封装列表的某个对象进行同步来完成此操作。
         如果不存在这样的对象，则应使用``{@link Collections＃synchronizedList Collections.synchronizedList}``方法“wrapped（包装）”列表。最好在创建时完成此操作，以防止意外地对列表进行非同步访问：``List list = Collections.synchronizedList(new LinkedList(...));``
         该类的``{ @code iterator}``和``{@code listIterator}``方法是**fail-fast**：如果在创建迭代器后的任何时间对列表进行结构修改，则可以通过迭代器自己的``{@code remove }``或``{@code add}``方法，迭代器将抛出``{@link ConcurrentModificationException}``。因此，面对并发修改，迭代器会快速干净地失败，而不会在未来的不确定时间内冒任意，不确定的行为的风险。 
         请注意，不能保证迭代器的快速失败行为，因为通常来说，在存在不同步的并发修改的情况下，不可能做出任何严格的保证。快速失败的迭代器会尽最大努力抛出``{@code ConcurrentModificationException}``。因此，编写依赖于此异常的程序来确保程序的正确性是错误的：迭代器的快速失败行为应仅用于检测错误。

#### 源码

- 属性

  ```java
  //保存的元素数量
  transient int size = 0;
  
  /**
   * Pointer to first node.
   * Invariant: (first == null && last == null) ||
   *            (first.prev == null && first.item != null)
   */
  transient Node<E> first;
  
  transient Node<E> last;
  
  // 节点数据类型
  private static class Node<E> {
          E item; 		//数据
          Node<E> next;   //后继
          Node<E> prev;	//前驱
  
          Node(Node<E> prev, E element, Node<E> next) {
              this.item = element;
              this.next = next;
              this.prev = prev;
          }
      }
  ```

- 构造方法

  ```java
  public LinkedList() {
  }
  
  public LinkedList(Collection<? extends E> c) {
      this();
      addAll(c);
  }
  
  // 从list尾部拼接new集合
  public boolean addAll(Collection<? extends E> c) {
          return addAll(size, c);
      }
  ```

- addAll()方法

  ```java
  public boolean addAll(int index, Collection<? extends E> c) {
      	// index >= 0 && index <= size
          checkPositionIndex(index);
  
          Object[] a = c.toArray();
          int numNew = a.length;
          if (numNew == 0)
              return false;
  
          Node<E> pred, succ;
          if (index == size) {
              succ = null;
              pred = last;
          } else {
              succ = node(index);
              pred = succ.prev;
          }
  
          for (Object o : a) {
              @SuppressWarnings("unchecked") E e = (E) o;
              Node<E> newNode = new Node<>(pred, e, null);
              if (pred == null)
                  first = newNode;
              else
                  pred.next = newNode;
              pred = newNode;
          }
  
          if (succ == null) {
              last = pred;
          } else {
              pred.next = succ;
              succ.prev = pred;
          }
  
          size += numNew;
          modCount++;
          return true;
      }
  ```

- 添加方法，有两个参数列表：末尾添加（常数-渐进时间复杂度），指定index添加（o(N) 时间复杂度）

  ```java
  public boolean add(E e) {
          linkLast(e);
          return true;
  }
  
  //尾插
  void linkLast(E e) {
      final Node<E> l = last;
      final Node<E> newNode = new Node<>(l, e, null);
      last = newNode;
      if (l == null)
  		//无尾结点 即为 空list
          first = newNode;
      else
          l.next = newNode;
      size++;
      modCount++;
  }
  ```

  ```java
  public void add(int index, E element) {
      	// index属于[0,size]
          checkPositionIndex(index);
  
          if (index == size)
              //插入到最后一个元素后面 == 尾插
              linkLast(element);
          else
              linkBefore(element, node(index));
  }
  
  //插入到元素 succ 前一位
  void linkBefore(E e, Node<E> succ) {
          // assert succ != null;
          final Node<E> pred = succ.prev;
          final Node<E> newNode = new Node<>(pred, e, succ);
          succ.prev = newNode;
          if (pred == null)
              first = newNode;
          else
              pred.next = newNode;
          size++;
          modCount++;
      }
  
  // 返回从0开始的index 对应的元素
  Node<E> node(int index) {
     		//分半查
          if (index < (size >> 1)) {// 右移 除2
              Node<E> x = first;
              for (int i = 0; i < index; i++)
                  x = x.next;
              return x;
          } else {
              Node<E> x = last;
              for (int i = size - 1; i > index; i--)
                  x = x.prev;
              return x;
          }
      }
  ```

- 删除方法remove()

  ```java
  public E remove(int index) {
      	// index [0,size)
          checkElementIndex(index);
          return unlink(node(index));
  }
  
  public boolean remove(Object o) {
          if (o == null) {
              for (Node<E> x = first; x != null; x = x.next) {
                  if (x.item == null) {
                      unlink(x);
                      return true;
                  }
              }
          } else {
              for (Node<E> x = first; x != null; x = x.next) {
                  if (o.equals(x.item)) {
                      unlink(x);
                      return true;
                  }
              }
          }
          return false;
      }
  ```

  
