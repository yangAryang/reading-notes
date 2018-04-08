# ArrayList
### 1.ArryList被引入得原因
答：数组功能得弱小化和不方便化。

### 2.ArrayList集合特征：
答：List接口下的一个实现类,底层采用动态数组。可实现基本的增删改查功能。

### 3.关注特征：
#### 3.1是否允许为空?
答：允许NULL值得存在。
#### 3.2是否允许存储重复元素?
答：允许存储重复元素。
#### 3.3是否有序?
答：有序。
#### 3.4是否允许线程安全?
答：非线程安全。

### 4.关注源码(JDK:1.8.0.161)

#### 4.1 初始化大小：
```
private static final int DEFAULT_CAPACITY = 10;
```
答：从源码内可看到初始化得大小为10.
#### 4.2 扩容机制：

```
    //确保容量在范围内
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
    //什么时候扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    //扩容具体方案。
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //
        int newCapacity = oldCapacity + (oldCapacity >> 1); 
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        //扩容核心代码
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
答：扩容机制,最后看到得还是grow方法。
    JDK1.6得时候扩容方式是乘3除2再加1.
    目前是原始容量+原始容量得一半构成新容量。如果不清除无符号右移是什么意思先自行补充基础。新的容量拿到后,然后范围得判断。确保容量是可用得。然后调用数组进行Copy,这也是为什么ArrayList集合底层是动态数组得原因。想当初老师说动态数组,还没反应过来。

#### 4.3 增

```
    transient Object[] elementData;
    
    //将元素添加至尾部
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    
    //将元素添加至指定索引处
    public void add(int index, E element) {
        rangeCheckForAdd(index);
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
    
    //将一个集合中得所有元素添加至此集合得末尾
    public boolean addAll(Collection<? extends E> c){
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
    
    //同上,但是指定了所插入得位置。
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
```
答：总体来说想进行确认,增加之后容量大小是合法得,然后将元素放置在末尾。体现了增加元素得顺序。

#### 4.3 删

```
    //移除指定具体元素
    public boolean remove(Object o)
    
    //移除指定索引位置得元素
    public E remove(int index) 
    
    //快速移除指定索引处得元素。
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
    
    //移除规定范围索引得元素,左含右不含
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // clear to let GC do its work
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }
    
    
```

#### 4.4 改

```
    //修改指定索引处元素得值。
    //选进行范围确认,然后修改底层数组内容。
    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
    
    
```

#### 4.5 查

```
    //得到指定索引处得元素得值
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```

### 5.扩容机制思考:为什么是1.5？
答：第一次有人问我得时候我第一反应就是效率问题。太大不好,太小也不好。可具体还是不直到为什么。谷歌了之后：[The expensive part at increasing the capacity of an ArrayList is copying the content of the backing array a new (larger)one.](https://stackoverflow.com/questions/5040753/why-arraylist-grows-at-a-rate-of-1-5-but-for-hashmap-its-2) 基于数组Copy得思想,开发者认为1.5倍是最合适得。不会太浪费也不会造成频繁扩容。

### 6.ArryList得优缺点？

##### 优点：
1.底层采用动态数组,随机访问模式,并且实现了标记接口RandomAccess，表明get得速度非常快。</br>
2.元素顺序添加方便,仅仅是数组得添加。
##### 缺点：
删除和插入元素得时候，涉及到元素复制，如果要复制得元素很多得话，耗费性能。
###### 综上可得使用场景：顺序添加,随机访问。
