
注意：基于JDK1.8

### 位置
```
 java.util.LinkedList
```

### 介绍
```
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```
1. 刚学Java那会，总有人说ArrayList是由数组实现的，LinkedList是由链表实现的，ArrayList和LinkedList的是有序的,ArrayList根据索引随机查找快，LinkedList添加删除快。 
2. LinkedList跟ArrayList一样不是线程安全的，当然也可使用Collections.synchronizedList方法包装一下。
3. LinkedList中也使用fail-fast机制，不知道的小伙伴可以参考ArrayList的源码分析[JDK源码学习（二） ArrayList](https://blog.csdn.net/blue5945/article/details/80966033)中的关于fail-fast的介绍
4.  下面我们来看看ArrayList的继承体系
![这里写图片描述](https://img-blog.csdn.net/2018072019265984?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JMVUU1OTQ1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
如上图所示
LinkedList继承了AbstractSequentialList （只支持按次序访问，而不像 AbstractList 那样支持随机访问）抽象类，实现了Deque（Deque是双端队列，具有栈和队列的性质），cloneable，serializable接口。

### 源码分析
#### 属性
```
	// 记录了LinkedList的节点数量
	transient int size = 0;

    /**
     * 指向第一个节点
     */
    transient Node<E> first;

    /**
     * 指向最后一个节点
     */
    transient Node<E> last;
	
	/**
	 * 看了第一个节点和最后一个节点都是Node类型，我们来看看None到底是啥
	 * None是LinkedList的一个私有的内部静态类,代表着一个节点，item代表着存储的数据，next、prev分别代表着前一
	 * 个节点和后一个节点，从这里可以看出LinkedList是一个双向链表
	 */
	private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

#### 构造方法
```
	/**
     * 构造一个空列表
     */
    public LinkedList() {
    }

    /**
	 * 构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序，做了两步事情
	 * 1.调用默认构造函数
	 * 2. 调用addAll(Collection c)方法，等会看看addAll方法干了啥
	 * 
     */
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
``` 

#### 常见方法
##### 添加
```
	/**
     * 在双端链表前面添加元素，调用linkFirst方法
     */
    public void addFirst(E e) {
        linkFirst(e);
    }

    /**
     * 在双端链表尾部添加元素，调用linkLast方法
     */
    public void addLast(E e) {
        linkLast(e);
    }
	
	/**
	 * 1.这里使用final变量，猜测是为了防止f变量在后面的操作中被重新赋值，后面许多方法中都使用了final变量
     * 2. 创建一个新节点有前面的Node内部类可知，将新建的节点的下一个节点指向的头结点
     * 3. 如果头结点为空，那么说明只有1个节点，将尾节点也指向新建的节点
     * 否则将头结点的前一节点指向新建的节点，建立双向链接
     * 4.将LinkedList的节点数量size自增
     * 5.将modCount自增
     * 这里又看到modCount了，你应该懂得（不懂看ArrayList的分析）
     * 不过这里的modCount是继承了其父类的父类AbstractList的
     */
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }

    /**
     * 链接尾节点，跟linkFirst差不多，不同的是需要将新建的节点与尾节点建立双向链接，之后将尾节点指向新建的节点
     */
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
	
	/**
	 * 还记得构造方法中的调用的addAll方法，其实调用的重载方法，实现添加到LinkedList尾部
	 */
	public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }
	
    /**
     * 添加集合到LinkedList中，index代表想要添加的位置
     * 1.首先判断添加位置是否合适（在>=0 && <= size范围内）
     * 2.将想要添加的集合转换为分数组，如果数组长度为0，，则直接返回false
     * 3.使用pred和succ来存要插入节点的前一个节点和前一个节点的前一个节点
     * 4.循环添加节点到前一个节点的位置，并移动尾节点的指向
     * 5.LinkedList节点数量加相应的大小，modCount自增
     */
    public boolean addAll(int index, Collection<? extends E> c) {
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
	
	/**
	 * 判断想要添加的位置是否符合要求，其实调用的是isPositionIndex方法
	 */
	private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

   
	/**
     * 当位置大于等于0并且位置大小小于等于LinkedList的节点的数量即为符合条件位置
     */
    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }
	
	/**
	 * 添加节点
	 * 实际上调用的linkLast方法，即在尾部添加节点
	 */
	public boolean add(E e) {
        linkLast(e);
        return true;
    }
```

##### 删除
```
	/**
     * 删除第一个节点
     * 
     * 我们待会看unlinkFirst方法
     */
    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }

    /**
     * 删除最后一个节点
     * 同样也是为null判断，然后调用unlinkLast方法
     */
    public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }
    
	/**
     * 我们来看unlinkFirst方法
     * 1.保存头结点的值用于返回，保存头结点的下一个节点
     * 2.将传过来的头结点的值和指向下一个节点的变量next的值置为null
     * 3.将头结点指向前面保存的头结点的下一个节点
     * 4.此时头结点指向next，如果next为空，则代表没有节点了，于是将尾节点也置为null，
     * 如果next不为空，则将next的指向前面节点的变量置为null，因为此时next为头结点
     * 5.将LinkedList的节点数量size自减,modCount++
     * 6.最后返回删除的节点的值
     * 
     */
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
    
    /**
     * 和unLindLast差不多，差别在于，这次删除的是尾节点
     * 
     */
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
	
	/**
	 * 删除值为o的节点
	 * 分两种情况
	 * 1.当o为null时，可以看到就是使用循环一直获取下一个节点使用==来判断
	 * 2.当o部位null时，使用equals来判断，那么如果自定义类，是不是要重写equals方法呢，你懂的，如果没有相等的
	 * 最后都调用unlink方法
	 */
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
	
	/**
     * 1.首先保存该节点的值，前一个节点和后一个节点的引用
     * 2.判断前一个节点是否为null，如果是则代表是头节点，则头结点指向下一个节点，否则则将该节点的上一个节点的
     * next引用，指向该节点的next，将该节点的前一节点引用置为null
     * （说白了即是将该节点的前一个节点与该节点断开链接，并重新链接）
     * 3.判断下一个节点是否为null，如果是则代表是尾节点，则将尾节点指向该节点的上一节点，否则则将该节点的下一个节
     * 点的prev引用，指向该节点的prev，将该节点的下一个都节点的应用置为null
     * 4.将节点的值置为null
     * 5.将LinkedList的节点数量size自减,modCount++
     * 6.返回该节点的值
     */
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
```

##### 获取
```
	/**
     * 获取第一个节点元素
     * 如果第一个节点元素为空则抛出NoSuchElementException异常
     */
    public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }

    /**
     * 获取最后一个节点
     * 如果该节点为空则抛出异常
     */
    public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }
```

##### 其他重要方法
```
	 /**
     * 判断是否含有值为o的节点 调用indexOf方法
     */
    public boolean contains(Object o) {
        return indexOf(o) != -1;
    }

    /**
     * 返回LinkedList节点数量，实际上就是返回size
     */
    public int size() {
        return size;
    }
	
	/**
	 * 返回值为o的节点在LinkedList第一次出现的位置
	 * 分两种情况
	 * 1.当o为null时，可以看到就是使用循环一直获取下一个节点使用==来判断
	 * 2.当o部位null时，使用equals来判断，那么如果自定义类，是不是要重写equals方法呢，你懂的，如果没有相等的
	 * 则返回-1
	 */
	 public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
	
	/**
	 * 返回值为o的节点在LinkedList最后一次出现的位置
	 * 跟indexOf类似，但是循环是从后往前的
	 */
	public int lastIndexOf(Object o) {
        int index = size;
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (x.item == null)
                    return index;
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (o.equals(x.item))
                    return index;
            }
        }
        return -1;
    }
	
	 /**
     * 清空LinkedList
     * 循环将节点的值，上一个节点的引用，下个节点的引用置为null，同时将头结点和尾节点的引用置为null
     * 修改size为0
     */
    public void clear() {
        // Clearing all of the links between nodes is "unnecessary", but:
        // - helps a generational GC if the discarded nodes inhabit
        //   more than one generation
        // - is sure to free memory even if there is a reachable Iterator
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }
	
	

    /**
     * clone方法，实际上还是调用了Object的Clone方法，属于浅克隆
     */
    public Object clone() {
        LinkedList<E> clone = superClone();

        // Put clone into "virgin" state
        clone.first = clone.last = null;
        clone.size = 0;
        clone.modCount = 0;

        // Initialize clone with our elements
        for (Node<E> x = first; x != null; x = x.next)
            clone.add(x.item);

        return clone;
    }
	
	@SuppressWarnings("unchecked")
    private LinkedList<E> superClone() {
        try {
            return (LinkedList<E>) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new InternalError(e);
        }
    }

    /**
     * 转换为数组方法，就是循环遍历，将节点的值放在数组中
     */
    public Object[] toArray() {
        Object[] result = new Object[size];
        int i = 0;
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.item;
        return result;
    }
```

##### 队列操作
由于LinkedList继承了Deque，所以自然有一些队列的操作，还有一些双端队列的操作
```
	/**
	 * 返回第一个节点的值
	 */
	public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

    /**
     * 返回第一个元素的值，调用getFirst方法，和peek的区别在与当没有节点时peek不会抛出异常返回null
     * 而element则会抛出异常
     */
    public E element() {
        return getFirst();
    }

    /**
     * 删除第一个节点，并返回该节点的值
     */
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

    /**
     * 删除第一个节点，调用的removeFirst方法
     */
    public E remove() {
        return removeFirst();
    }

    /**
     * 在尾部添加节点
     * 调用add方法
     */
    public boolean offer(E e) {
        return add(e);
    }

    // Deque operations
    /**
     * Inserts the specified element at the front of this list.
     *
     * @param e the element to insert
     * @return {@code true} (as specified by {@link Deque#offerFirst})
     * @since 1.6
     */
    public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }

    /**
     * 在尾部添加节点
     */
    public boolean offerLast(E e) {
        addLast(e);
        return true;
    }
	
    /**
     * 返回头部节点的值
     */
    public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
     }

    /**
     * 返回尾部节点的值
     */
    public E peekLast() {
        final Node<E> l = last;
        return (l == null) ? null : l.item;
    }

    /**
     * 删除头部节点
     */
    public E pollFirst() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

    /**
     * 删除尾部节点
     */
    public E pollLast() {
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }

    /**
     * 向头部添加节点
     */
    public void push(E e) {
        addFirst(e);
    }

    /**
     * 删除第一个节点
     */
    public E pop() {
        return removeFirst();
    }

    /**
     * 删除第一个值为o的节点
     */
    public boolean removeFirstOccurrence(Object o) {
        return remove(o);
    }

    /**
     * 删除最后一个值为o的节点
     */
    public boolean removeLastOccurrence(Object o) {
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

##### 迭代器
LinkedList也有ListIterator和ArrayList差不多，这里就不讲了
```
 public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
    }

    private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;
        private Node<E> next;
        private int nextIndex;
        private int expectedModCount = modCount;

        ListItr(int index) {
            // assert isPositionIndex(index);
            next = (index == size) ? null : node(index);
            nextIndex = index;
        }

        public boolean hasNext() {
            return nextIndex < size;
        }

        public E next() {
            checkForComodification();
            if (!hasNext())
                throw new NoSuchElementException();

            lastReturned = next;
            next = next.next;
            nextIndex++;
            return lastReturned.item;
        }

        public boolean hasPrevious() {
            return nextIndex > 0;
        }

        public E previous() {
            checkForComodification();
            if (!hasPrevious())
                throw new NoSuchElementException();

            lastReturned = next = (next == null) ? last : next.prev;
            nextIndex--;
            return lastReturned.item;
        }

        public int nextIndex() {
            return nextIndex;
        }

        public int previousIndex() {
            return nextIndex - 1;
        }

        public void remove() {
            checkForComodification();
            if (lastReturned == null)
                throw new IllegalStateException();

            Node<E> lastNext = lastReturned.next;
            unlink(lastReturned);
            if (next == lastReturned)
                next = lastNext;
            else
                nextIndex--;
            lastReturned = null;
            expectedModCount++;
        }

        public void set(E e) {
            if (lastReturned == null)
                throw new IllegalStateException();
            checkForComodification();
            lastReturned.item = e;
        }

        public void add(E e) {
            checkForComodification();
            lastReturned = null;
            if (next == null)
                linkLast(e);
            else
                linkBefore(e, next);
            nextIndex++;
            expectedModCount++;
        }

        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            while (modCount == expectedModCount && nextIndex < size) {
                action.accept(next.item);
                lastReturned = next;
                next = next.next;
                nextIndex++;
            }
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

##### 倒序迭代器
由于LinkedList是双端队列，使用descendingIterator可以倒序迭代，实际上利用了ListItr方法生成的List迭代器
```
	 /**
     * @since 1.6
     */
    public Iterator<E> descendingIterator() {
        return new DescendingIterator();
    }

    /**
     * Adapter to provide descending iterators via ListItr.previous
     */
    private class DescendingIterator implements Iterator<E> {
        private final ListItr itr = new ListItr(size());
        public boolean hasNext() {
            return itr.hasPrevious();
        }
        public E next() {
            return itr.previous();
        }
        public void remove() {
            itr.remove();
        }
    }
```

##### 并行迭代器
JDK1.8才出现有利于提高效率，可以利用多核优势，代码先放这里，有时间学习一下相关知识
```
  @Override
    public Spliterator<E> spliterator() {
        return new LLSpliterator<E>(this, -1, 0);
    }

    /** A customized variant of Spliterators.IteratorSpliterator */
    static final class LLSpliterator<E> implements Spliterator<E> {
        static final int BATCH_UNIT = 1 << 10;  // batch array size increment
        static final int MAX_BATCH = 1 << 25;  // max batch array size;
        final LinkedList<E> list; // null OK unless traversed
        Node<E> current;      // current node; null until initialized
        int est;              // size estimate; -1 until first needed
        int expectedModCount; // initialized when est set
        int batch;            // batch size for splits

        LLSpliterator(LinkedList<E> list, int est, int expectedModCount) {
            this.list = list;
            this.est = est;
            this.expectedModCount = expectedModCount;
        }

        final int getEst() {
            int s; // force initialization
            final LinkedList<E> lst;
            if ((s = est) < 0) {
                if ((lst = list) == null)
                    s = est = 0;
                else {
                    expectedModCount = lst.modCount;
                    current = lst.first;
                    s = est = lst.size;
                }
            }
            return s;
        }

        public long estimateSize() { return (long) getEst(); }

        public Spliterator<E> trySplit() {
            Node<E> p;
            int s = getEst();
            if (s > 1 && (p = current) != null) {
                int n = batch + BATCH_UNIT;
                if (n > s)
                    n = s;
                if (n > MAX_BATCH)
                    n = MAX_BATCH;
                Object[] a = new Object[n];
                int j = 0;
                do { a[j++] = p.item; } while ((p = p.next) != null && j < n);
                current = p;
                batch = j;
                est = s - j;
                return Spliterators.spliterator(a, 0, j, Spliterator.ORDERED);
            }
            return null;
        }

        public void forEachRemaining(Consumer<? super E> action) {
            Node<E> p; int n;
            if (action == null) throw new NullPointerException();
            if ((n = getEst()) > 0 && (p = current) != null) {
                current = null;
                est = 0;
                do {
                    E e = p.item;
                    p = p.next;
                    action.accept(e);
                } while (p != null && --n > 0);
            }
            if (list.modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }

        public boolean tryAdvance(Consumer<? super E> action) {
            Node<E> p;
            if (action == null) throw new NullPointerException();
            if (getEst() > 0 && (p = current) != null) {
                --est;
                E e = p.item;
                current = p.next;
                action.accept(e);
                if (list.modCount != expectedModCount)
                    throw new ConcurrentModificationException();
                return true;
            }
            return false;
        }

        public int characteristics() {
            return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
        }
    }
```

### 小结
凌晨两点，好困啊，嘿嘿，继续努力。接下来我们来给LinkedList做一个小结
1.LinkedList是双端链表，使用内部静态类Node实现，并且记录了头结点和尾节点
2.LinkedList同样具有fast-fail机制
3.双端队列的特性使LinkedList可以从前往后也可以从后往前，可以用来模拟栈和队列
