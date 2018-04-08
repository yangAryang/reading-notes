### 1.概述
答：LinkedList与ArrayList同样实现于List接口,ArrayList底层基于数组,而LinkedList则是基于双向链表。

### 2.集合特征
答：双向链表特征使得与ArrayList相比来说,普通情况下的插入和删除操作优于ArrayList,但是随机访问的性能则逊色于ArrayList。
### 3.关注特征
#### 3.1是否允许为空?
答：允许NULL值得存在。
#### 3.2是否允许存储重复元素?
答：允许存储重复元素。
#### 3.3是否有序?
答：有序。
#### 3.4是否允许线程安全?
答：非线程安全。

### 4.关注源码
#### 4.1内部结点的构成

```
//静态内部类
    private static class Node<E> {
        //实际存储数据的部分。
        E item;
        //向前和向后所指向的指针
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```
答：Node类是其内部的静态内部类,用于存储每个结点对应的前置指针、数据和后置指针。item是真正用于存储实际数据的容器。

#### 4.2 关注构造器

```
    //提供了一个空参构造器
    public LinkedList() {
    }
    //提供了一个用于初始化集合的构造器
    public LinkedList(Collection<? extends E> c) {
        this();
    //调用方法将集合c中的元素全部添加到LinkedList内
        addAll(c);
    }
```
答：基本的无参构造器和一个用于导入其他集合的有参构造器。
#### 4.3 增- 普通add

```
    //普通的add方法内部调用的就是将元素添加至末尾的方法。也就是链表中的尾插法。
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```
答：将原本最后一个结点的引用给了l,然后创建一个结点并将元素初始化。再见新结点的引用给了最后一个结点的引用。接着继续判断之前的最后一个结点是否为null,将引用确定话。因为仍要保证双向链表的特征,头结点的前置指针指向最后一个结点,最后一个结点的后置指针仍旧指向头节点的。


#### 4.4 增- 变形add
```
    //头插法
    public void addFirst(E e) {
        linkFirst(e);
    }
    
    //尾插法
    public void addLast(E e) {
        linkLast(e);
    }
    
    //两种addAll方法,区别在于是否指定位置
    //未指定位置则插入到末尾
    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);
    //先将集合转换
        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;
        //
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
答：做法仍旧是先将集合转换为Object数组,然后创建结点。注意创建结点的位置,并不是在循环之内创建,而是在循环之外创建,复用性高。最后通过循环将其插入。

### 4.5 删 所有的移除算法都是基于以下关键算法变形

```
//remove的时候分开判断,null值与非null值
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
    
    //关键算法:将引用和data赋为null。
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
    
    //移除首元素
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
    
    //移除尾元素
    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }
```
**remove方法为什么要分开判断null值与非null值？unlink算法不赋为null可以吗？**</br>
答：LinkedList集合或其父类中的equals方法肯定改写了Object的equals方法。改写了判断两个值是否相等的方法内容。其实在unlinkl方法中将data与引用的值不赋为null也是可以的,此时已经没有任何引用指向他们了,所以在HotSpot中采用的根节点搜素算法是可以的,但是为什么还要赋值为null,ImportNew站点给出的想法是为了兼容其他虚拟机中另外一种垃圾回收算法-引用计数法。不太懂的可以移步到JVM那一章节。总之我猜都是为了效率和复用。

#### 4.6 改

```
//设置指定索引处元素的值,返回旧值
    public E set(int index, E element) {
        checkElementIndex(index);
        //得到指定索引处的结点的node元素
        Node<E> x = node(index);
        //先拿旧值再赋新值
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }   
```
答：算法体现的思想,先拿旧值再赋新值。

#### 4.7 查

```
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
    
    //重点关注查的方式与ArrayList的区别
    Node<E> node(int index) {
        // assert isElementIndex(index);
        //判断范围,然后决定遍历的顺序
        if (index < (size >> 1)) {
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
答：二分思想,不再穷举暴力查找,先看范围后决定遍历的方式。

### 5.LinkedList的优缺点:
优点:一般情况下插入和删除速率较高,但只是一定意义上。
</br>
缺点:维护双向链表增加数据的同时仍要维护相应的引用。
</br>
具体的比较需要源码来分析,篇幅过长,下一节。
</br>
