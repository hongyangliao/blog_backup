注意：基于JDK1.8
### 位置
```
java.util.ArrayList
```

### 介绍
```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```
1. 说到Java的集合，估计大部分人第一个想到的就是ArrayList，从名字可以推测ArrayList底层是使用数组实现的，查看源码可以验证这个想法。
2. 那么我们为啥不直接数组呢，我们都知道数组的长度固定的，确定了就无法更改，而ArrayList使用的也是数组，但是实现了动态扩容功能（即创建一个新的数组，这个数组的容量比原来大一些，然后复制原来的数组）
3.  但是ArrayList使用Object[]数组实现，因此可以存放不同类型的对象，不过这在强转时容易出现问题，泛型可以解决这个问题，但是这样就只能存储一种类型的数据(泛型是在编译期做的校验，实际上会有一个泛型擦除的过程，可以反编译一下class文件可以看到泛型已经不存在)
4.  ArrayList不是线程安全的集合，因此如果在多线程的情况下使用它需要自己同步，或者使用集合工具类Collections.synchronizedList方法将其转换为线程安全的List，当然Vector和CopyOnWriteArrayList也是线程安全的选择
5.  如下图所示为ArrayList的继承体系,，其中实线代表extend，虚线代表implement
 ![这里写图片描述](https://img-blog.csdn.net/20180706102723227?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JMVUU1OTQ1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

可以看到ArrayList继承了AbstractList,AbstractList实现了 List 的一些位置相关操作(比如 get,set,add,remove)，是第一个实现随机访问方法的集合类，但不支持添加和替换；RandomAccess、Cloneable、Serializable都是一些标记接口，RandomAccess标识其有随机访问的能力，因为底层使用数组实现，使用下标访问时间复杂度为O(1)，相对的LinkedList由于使用链表实现的，因此并没有RandomAccess接口

### 源码分析
#### 属性
```
	/**
     * 这个是ArrayList中的数组的默认容量
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 共享空常量数组
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * 这个也是共享的空常量数组，与上面的那个区别DEFAULTCAPACITY_EMPTY_ELEMENTDATA 知道如何扩容
     * DEFAULTCAPACITY_EMPTY_ELEMENTDATA用于无参初始化；EMPTY_ELEMENTDATA用于指定容量为0时的初始化。
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * 代表了实际存储的数据
     */
    transient Object[] elementData;

    /**
	 * 存储数据有一个容量和一个size，容量代表着数组的长度，size代表着实际存储数据的大小
     */
    private int size;
	
	 /**
     * 这里定义了ArrayList内部数组的最大大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
	
	/**
     * 记录了ArrayList的修改次数,不知道你有没有遇到ConcurrentModificationException（并发修改异常），一般出现在
     * 迭代的时候,边迭代边修改就会出现这个异常，我们后面许多方法用到了这个属性，别急用到的时候分析
     */
	protected transient int modCount = 0;
```

#### 构造方法
```
	/**
     * 指定初始容量的构造方法，当initialCapacity == 0时，将EMPTY_ELEMENTDATA赋给elementData 
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     * 无参的构造方法，将DEFAULTCAPACITY_EMPTY_ELEMENTDATA赋给elementData，构造一个初始容量为十的空列表
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
	 * 构造一个包含Collection对象的列表，其原理就是调用Collection对象的toArray()方法，然后赋给实际存储数据elementData 
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

#### 常见方法
下面我们从常见的添加、删除、修改、查找以及一些常用方法来分析一下ArrayList的实现原理
##### 添加
```
	/**
	 * 添加一个元素，首先判断是否需要扩容，然后将元素赋值给下一个数组位置 其中size上面有讲，我们等会来看一下
	 * ensureCapacityInternal这个方法
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    /**
     * 添加一个元素到指定下标位置
     * 1. 先判断是否超过size
     * 2.判断是否需要扩容，还有一些操作需要使用ensureCapacityInternal这个方法
     * 3.给数组对应位置位置赋值
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
	
	/**
	 * 将Collection对象的元素添加到列表尾部中，先判断是否需要扩容，然后使用了System.arraycopy方法，
	 * 是一个native方法，将a复制到elementData 中，接着将size增加
     */
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 这个跟上面类似，只不过指定添加的位置
     */
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

	// 接下来我们看看在添加操作中都使用到的ensureCapacityInternal方法
	
	/**
     * 可以看到首先判断是不是使用无参构造创建的ArrayList(无参构造直接使用的是DEFAULTCAPACITY_EMPTY_ELEMENTDATA)，如
     * 果是则比较当前需要的大小和默认容量谁大，取大的，然后调用ensureExplicitCapacity方法
     */
	private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        ensureExplicitCapacity(minCapacity);
    }
	
   /**
	* 看到这里首先将modCount自增，由于每个add方法都调用了ensureCapacityInternal，而ensureCapacityInternal也调用
	* ensureExplicitCapacity，所有每个添加操作都modCount都会自增，
	* 然后判断需要的容量大小大于lementData数组当前容量大小，则调用grow方法
	*/
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * 扩容方法
     * 可以看到这里如果扩容到原来大小1.5倍还不够的情况下则直接将容量大小设置为需要的大小
     * 当容量大于MAX_ARRAY_SIZE时（如果你还记得它的值是多少的话，哈哈）调用hugeCapacity，返回最大容量大小
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
	
   /**
	* 判断当前新的容量是否大于MAX_ARRAY_SIZE，如果大于则返回int的最大值
	*/
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

	/**
     * 当调用指定位置的添加方法时都会调用这个方法检查是否在数组实际存储大小的值的范围内
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

##### 删除
添加讲完了，我们就来说一说删除方法
```
	/**
	 * 删除指定位置的元素并返回改元素的值
	 * 首先检查位置是否合法跟rangeCheckForAdd类似，我们等会来看，然后这里也修改了modCount，然后将index+1后的元素，复制
	 * 到从index开始的位置，即覆盖了index的值，然后将最后一个位置置为null，并将size减1
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

	/**
     * 从列表中删除特定值
     * 如果是null，就删除null值元素
     * 如果不为null，则调用equals方法判断是否相同，相同则删除，我们接下来来看看fastRemove方法
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
	
	/*
     * 这个方法跟上面的remove(int index)类似，那么为啥叫fastRemove，对比一下就知道少了范围校验和返回值
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
    
	/**
     * 清空里列表
     * 即将数组的值都置为null，并将size置为0
     */
    public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
	
   /**
	* 删除一定范围内列表的值，protected 注意下
	*/
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
	
	/**
     * 删除本列表中包含指定Collection对象值的元素，调用batchRemove方法
     */
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, false);
    }
	
	/**
     * 只保留本列表中包含指定Collection对象值的元素，调用batchRemove方法
     */
	public boolean retainAll(Collection<?> c) {
	        Objects.requireNonNull(c);
	        return batchRemove(c, true);
	    }

	/**
     * 批量删除
     * 可以看到removeAll和retainAll都用了这个方法，不同的是complement不同
     * removeAll complement为false 其实就是求差集
     * retainAll complement为true 求并集
     * 当抛出异常时r肯定不等于size
     */
	private boolean batchRemove(Collection<?> c, boolean complement) {
	        final Object[] elementData = this.elementData;
	        int r = 0, w = 0;
	        boolean modified = false;
	        try {
	            for (; r < size; r++)
	                if (c.contains(elementData[r]) == complement)
	                    elementData[w++] = elementData[r];
	        } finally {
	            // Preserve behavioral compatibility with AbstractCollection,
	            // even if c.contains() throws.
	            if (r != size) {
	                System.arraycopy(elementData, r,
	                                 elementData, w,
	                                 size - r);
	                w += size - r;
	            }
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
```

##### 查找
```
	/**
     * 获取列表指定位置的值
     */
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```

##### 修改
```
	/**
     * 修改指定位置的值
     * 可以看到修改并没有修改modCount
     */
    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```

##### 其他重要的方法
ArrayList中还有一些比较重要的方法
```
	 /**
     * 1.修改次数加1
     * 2.将elementData中空余的空间（包括null值）去除，例如：数组长度为10，其中只有前三个元素有值，其他为空，那么调用该方
     * 法之后，数组的长度变为3
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }

	/**
     * 判断列表是否为空
     */
    public boolean isEmpty() {
        return size == 0;
    }

    /**
     * 判断是否包含某个元素，实际上调用indexOf方法
     */
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }

    /**
     * 返回elementData中与该对象相等的值第一次出现的位置，同样分为null和不为null的情况
     */
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回elementData中与该对象相等的值最后一次出现的位置，跟indexOf方法类似
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * clone方法
     */
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
    }
	
	/**
	 * 将ArrayList转换为数组对象实际上，就是copy elementData出来
	 */
	public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * 将ArrayList的值复制到a数组中
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
    
	/**
    * 给ArrayList排序，传入Comparator对象，实际上调用的是Arrays.sort方法
    */
	@Override
    @SuppressWarnings("unchecked")
    public void sort(Comparator<? super E> c) {
        final int expectedModCount = modCount;
        Arrays.sort((E[]) elementData, 0, size, c);
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }
```

##### 迭代器
ArrayList中的内部类真是看得挺头疼的，占了一半的源码
```
	/**
	 * 返回指定位置的List迭代器
	 */
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    /**
     * 返回List迭代器
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }
    
    /**
     * 返回迭代器
     */
	public Iterator<E> iterator() {
        return new Itr();
    }
	
	//我们来看看ListItr和Itr是啥玩意
	/**
	 * Itr是ArrayList的内部类，我们主要看cursor、lastRet 属性，和next()方法，
	 * cursor记录当前迭代到哪个位置，lastRet 记录下一个需要返回的元素的下标，next（）主要是返回下一个元素的值，其中有个
	 * checkForComodification方法，里面modCount终于有了用场，我们来看看
	 */
	private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }
		
	    /**
	    * 不错你很有耐心，就是这里modCount 有了用场，当创建迭代器时将modCount赋给expectedModCount，
	    * 每次迭代时对比modCount 和expectedModCount的值，如果不同就说明了在迭代期间做了修改，这时候抛出
	    * ConcurrentModificationException，这种策略称为fail-fast，即快速失败，等会在小结的时候我们来看一下概念
	    */
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    /**
     * 这个和Itr 差不多但是多了向上迭代的功能
     */
    private class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {
            super();
            cursor = index;
        }

        public boolean hasPrevious() {
            return cursor != 0;
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor - 1;
        }

        @SuppressWarnings("unchecked")
        public E previous() {
            checkForComodification();
            int i = cursor - 1;
            if (i < 0)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i;
            return (E) elementData[lastRet = i];
        }

        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.set(lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                ArrayList.this.add(i, e);
                cursor = i + 1;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }
```

###### SubList
 接下来介绍最后一个啃老族，为啥说是啃老族族
```
	/**
	* 其作用是返回一个以fromIndex为起始索引（包含），以toIndex为终止索引（不包含）的子列表（List）
	* 其中SubList又是一个内部类值得注意的是，返回的这个子列表的幕后其实还是原列表；
	* 也就是说，修改这个子列表，将导致原列表也发生改变
	*/
	public List<E> subList(int fromIndex, int toIndex) {
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, 0, fromIndex, toIndex);
    }

    static void subListRangeCheck(int fromIndex, int toIndex, int size) {
        if (fromIndex < 0)
            throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
        if (toIndex > size)
            throw new IndexOutOfBoundsException("toIndex = " + toIndex);
        if (fromIndex > toIndex)
            throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                               ") > toIndex(" + toIndex + ")");
    }

    private class SubList extends AbstractList<E> implements RandomAccess {
        private final AbstractList<E> parent;
        private final int parentOffset;
        private final int offset;
        int size;

        SubList(AbstractList<E> parent,
                int offset, int fromIndex, int toIndex) {
            this.parent = parent;
            this.parentOffset = fromIndex;
            this.offset = offset + fromIndex;
            this.size = toIndex - fromIndex;
            this.modCount = ArrayList.this.modCount;
        }

        public E set(int index, E e) {
            rangeCheck(index);
            checkForComodification();
            E oldValue = ArrayList.this.elementData(offset + index);
            ArrayList.this.elementData[offset + index] = e;
            return oldValue;
        }

        public E get(int index) {
            rangeCheck(index);
            checkForComodification();
            return ArrayList.this.elementData(offset + index);
        }

        public int size() {
            checkForComodification();
            return this.size;
        }

        public void add(int index, E e) {
            rangeCheckForAdd(index);
            checkForComodification();
            parent.add(parentOffset + index, e);
            this.modCount = parent.modCount;
            this.size++;
        }

        public E remove(int index) {
            rangeCheck(index);
            checkForComodification();
            E result = parent.remove(parentOffset + index);
            this.modCount = parent.modCount;
            this.size--;
            return result;
        }

        protected void removeRange(int fromIndex, int toIndex) {
            checkForComodification();
            parent.removeRange(parentOffset + fromIndex,
                               parentOffset + toIndex);
            this.modCount = parent.modCount;
            this.size -= toIndex - fromIndex;
        }

        public boolean addAll(Collection<? extends E> c) {
            return addAll(this.size, c);
        }

        public boolean addAll(int index, Collection<? extends E> c) {
            rangeCheckForAdd(index);
            int cSize = c.size();
            if (cSize==0)
                return false;

            checkForComodification();
            parent.addAll(parentOffset + index, c);
            this.modCount = parent.modCount;
            this.size += cSize;
            return true;
        }

        public Iterator<E> iterator() {
            return listIterator();
        }

        public ListIterator<E> listIterator(final int index) {
            checkForComodification();
            rangeCheckForAdd(index);
            final int offset = this.offset;

            return new ListIterator<E>() {
                int cursor = index;
                int lastRet = -1;
                int expectedModCount = ArrayList.this.modCount;

                public boolean hasNext() {
                    return cursor != SubList.this.size;
                }

                @SuppressWarnings("unchecked")
                public E next() {
                    checkForComodification();
                    int i = cursor;
                    if (i >= SubList.this.size)
                        throw new NoSuchElementException();
                    Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length)
                        throw new ConcurrentModificationException();
                    cursor = i + 1;
                    return (E) elementData[offset + (lastRet = i)];
                }

                public boolean hasPrevious() {
                    return cursor != 0;
                }

                @SuppressWarnings("unchecked")
                public E previous() {
                    checkForComodification();
                    int i = cursor - 1;
                    if (i < 0)
                        throw new NoSuchElementException();
                    Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length)
                        throw new ConcurrentModificationException();
                    cursor = i;
                    return (E) elementData[offset + (lastRet = i)];
                }

                @SuppressWarnings("unchecked")
                public void forEachRemaining(Consumer<? super E> consumer) {
                    Objects.requireNonNull(consumer);
                    final int size = SubList.this.size;
                    int i = cursor;
                    if (i >= size) {
                        return;
                    }
                    final Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length) {
                        throw new ConcurrentModificationException();
                    }
                    while (i != size && modCount == expectedModCount) {
                        consumer.accept((E) elementData[offset + (i++)]);
                    }
                    // update once at end of iteration to reduce heap write traffic
                    lastRet = cursor = i;
                    checkForComodification();
                }

                public int nextIndex() {
                    return cursor;
                }

                public int previousIndex() {
                    return cursor - 1;
                }

                public void remove() {
                    if (lastRet < 0)
                        throw new IllegalStateException();
                    checkForComodification();

                    try {
                        SubList.this.remove(lastRet);
                        cursor = lastRet;
                        lastRet = -1;
                        expectedModCount = ArrayList.this.modCount;
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                public void set(E e) {
                    if (lastRet < 0)
                        throw new IllegalStateException();
                    checkForComodification();

                    try {
                        ArrayList.this.set(offset + lastRet, e);
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                public void add(E e) {
                    checkForComodification();

                    try {
                        int i = cursor;
                        SubList.this.add(i, e);
                        cursor = i + 1;
                        lastRet = -1;
                        expectedModCount = ArrayList.this.modCount;
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                final void checkForComodification() {
                    if (expectedModCount != ArrayList.this.modCount)
                        throw new ConcurrentModificationException();
                }
            };
        }

        public List<E> subList(int fromIndex, int toIndex) {
            subListRangeCheck(fromIndex, toIndex, size);
            return new SubList(this, offset, fromIndex, toIndex);
        }

        private void rangeCheck(int index) {
            if (index < 0 || index >= this.size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }

        private void rangeCheckForAdd(int index) {
            if (index < 0 || index > this.size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }

        private String outOfBoundsMsg(int index) {
            return "Index: "+index+", Size: "+this.size;
        }

        private void checkForComodification() {
            if (ArrayList.this.modCount != this.modCount)
                throw new ConcurrentModificationException();
        }

        public Spliterator<E> spliterator() {
            checkForComodification();
            return new ArrayListSpliterator<E>(ArrayList.this, offset,
                                               offset + this.size, this.modCount);
        }
    }
```

### 小结
ArrayList源码到这里就分析完了，花了一天时间，接下来做个小结
1. 由于ArrayList内部使用数组实现，那么其随机查找的速度非常快
2. 由于ArrayList每次扩容都需要复制数组，效率比较低，所以在已知大概需要多大容量的情况下应该明确指定创建ArrayList时容量值
3. modConut问题，每次修改时modCount都会增加，这样做是为了在迭代时防止并发修改
4.  使用modCount时使用的fail-fast几乎在在java.util的集合中都有使用

### fail-fast
1. 概念：在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出Concurrent Modification Exception。
2. 原理：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。集合在被遍历期间如果内容发生变化，就会改变modCount的值。每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历。
3. 注意：这里异常的抛出条件是检测到 modCount！=expectedmodCount 这个条件。如果集合发生变化时修改modCount值刚好又设置为了expectedmodCount值，则异常不会抛出。因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的bug。
4. 场景：java.util包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改）
