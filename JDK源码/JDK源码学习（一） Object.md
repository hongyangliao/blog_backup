注意：基于JDK1.8
### 位置
```
java.lang.Object
```
### 介绍
```
public class Object
```
Object为所有类的根类，所有的类都将Object类作为父类，每个对象（包括数组）都实现了Object类的方法


### 方法详细信息

###### private static native void registerNatives()
native方法，表明不是用Java代码实现，而是由C/C++去完成，并被编译成了.dll，由Java去调用，方法的具体实现根据平台的不同应该也是不同的。使用native修饰，即表示操作系统，需要提供此方法，Java本身需要使用。具体到registerNatives()方法本身，其主要作用是将C/C++中的方法映射到Java中的native方法，实现方法命名的解耦。在源码中还有一个静态代码块，会在加载Object时调用，对几个本地方法进行注册(也就是初始化java方法映射到C的方法)
```
private static native void registerNatives();
static {
	registerNatives();
}
```

###### public final native Class< ? > getClass()
返回此Object的运行时类。 返回的类对象是被表示类的static synchronized方法锁定的对象，这里返回的是真正的class对象，而不是父类的class对象，如下
```
Object obj = new Object();
Class clazz = obj.getClass();
System.out.println(clazz);
// result: class java.lang.Object
Number num = 1;
Class<? extends Number> number_clazz = num.getClass();
System.out.println(number_clazz);
// result: class java.lang.Integer
```
Number对象num实际类型为java.lang.Integer

###### public native int hashCode()
OpenJDK8 默认hashCode的计算方法是通过和当前线程有关的一个随机数+三个确定值，运用随机数算法得到的一个随机数。和对象内存地址无关。
 ----
支持这种方法一般是为了散列表，如HashMap。hashCode一般需要符合三种特性
1.只要在执行Java应用程序时多次在同一个对象上调用该方法， hashCode方法必须始终返回相同的整数， 该整数不需要在不同的应用程序中保持一致
2.如果根据equals(Object)方法两个对象相等，则在两个对象中的每个对象上调用hashCode方法必须产生相同的整数结果
3.如果两个对象根据equals(java.lang.Object)方法不相等，那么在两个对象中的每个对象上调用hashCode方法可以产生不同的整数结果。 但是，为不等对象生成不同的整数结果可能会提高哈希表的性能

###### public boolean equals(Object obj)
```
public boolean equals(Object obj) {
	return (this == obj);
}
```
默认equals方法使用的是 ==， 也就是比较两个对象地址是否相同，一般都需要重写equals方法,如String类中，就是比较String中的实际存储数据的char数组是否相同
```
 public boolean equals(Object anObject) {
		// 判断是相同对象直接返回真
        if (this == anObject) {
            return true;
        }
        // 判断是否是String类或者其子类，如果不是返回假
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            // 比较长度是否相等，如果不等返回假
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                // 判断char数组对应位置的值是否相同
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```
equals方法一般也需要满足5个特性
1.自反性 ：对于任何非空的参考值x ， x.equals(x)应该返回true 。 
2.对称性 ：对于任何非空引用值x和y ， x.equals(y)应该返回true当且仅当y.equals(x)回报true 。 
3.传递性 ：对于任何非空引用值x ， y和z ，如果x.equals(y)回报true个y.equals(z)回报true ，然后x.equals(z)应该返回true 。 
4.一致的 ：对于任何非空引用值x和y ，多次调用x.equals(y)始终返回true或始终返回false ，没有设置中使用的信息equals比较上的对象被修改。 
5.对于任何非空的参考值x ， x.equals(null)应该返回false 

###### protected native Object clone() throws CloneNotSupportedException
创建一个新的内存空间，复制一个对象，这里是浅拷贝，下面举个栗子：
```
// 创建一个Student对象，这里必须实现Cloneable方法才可以调用clon()方法，否则会报异常
public class Student implements Cloneable{
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

// clone Student 对象可以发现克隆对象的hashCode变了(默认的toString()方法为返回值为class对象名加hashCode)，可以判断不是同一个对象
Student stu = new Student();
Student stu1 = (Student) stu.clone();
System.out.println(stu);
//result: com.liao.Student@4554617c
System.out.println(stu1);
//result: com.liao.Student@74a14482

// 可以看到成员变量name为引用类型，改变stu的name值sut1也改变，这个说明为clone方法为浅拷贝
 Student stu = new Student();
 stu.setName("同学");
 stu.setAge(10);
 Student stu1 = (Student) stu.clone();
 stu.setName("你好");
 stu.setAge(2);
 System.out.println(stu.getName() + ":" + stu.getAge());
 //result: 你好:2

// 数组也有clone()方法,所有数组都被认为是实现接口Cloneable ，并且数组类型T[]的clone方法的返回类型是T[] 
int[] arr = new int[] {1, 2, 4};
int[] arr1 = (int[])arr.clone();
System.out.println(Arrays.toString(arr));
System.out.println(Arrays.toString(arr1));
```
结论:
1. object的clone()方法会创建一个新的对象
2. object的clone()方法为浅拷贝
3. 调用clone()方法的类需要实现Cloneable标识接口，否则会则抛出CloneNotSupportedException异常
4. 数组也有clone()方法,所有数组都被认为是实现接口Cloneable ，并且数组类型T[]的clone方法的返回类型是T[]

###### public String toString()
> 返回对象的字符串表示形式，默认返回class对象的名字@hashCode的16进制
```
public String toString() {
	return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

###### public final native void notify()
唤醒正在等待对象监视器的单个线程

###### public final native void notifyAll()
唤醒正在等待对象监视器的所有线程

###### public final native void wait(long timeout) throws InterruptedException
导致当前线程等待，直到另一个线程调用该对象的 notify()方法或 notifyAll()方法

###### public final void wait(long timeout, int nanos) throws InterruptedException
导致当前线程等待，直到另一个线程调用 notify()方法或该对象的 notifyAll()方法，或者指定的时间已过，这里实际调用的是上面的wait(long timeout)方法,我们可以看到传递的纳秒值实际上只做了判断并没有实际使用，也就是实际上只有毫秒起了作用
```
 public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
		// 只判断了nanos即纳秒值，并没有使用
        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeout++;
        }
		// 调用 wait(timeout)方法
        wait(timeout);
    }

```

###### public final void wait() throws InterruptedException
导致当前线程等待，直到另一个线程调用该对象的 notify()方法或 notifyAll()方法，这个方法调用的实际是本地方法 wait(long timeout)
```
// 生产者
public final void wait() throws InterruptedException {
        wait(0);
    }
```
下面对notify/notifyAll和wait的用法举一个栗子,就用比较经典的生产者消费者问题
```
public class Process implements Runnable {
    //传递数据
    private List<String> queue;
    private int count = 0;
    //对象,作为锁
    private Object lock;

    public Process(List queue, Object lock) {
        this.queue = queue;
        this.lock = lock;
    }

    @Override
    public void run() {
        while (true) {
            synchronized (lock) {
                // 当queue中没有数据时，生产数据，否则等待释放锁
                if (queue != null && queue.size() == 0) {
                    System.out.println("生产了" + count);
                    queue.add("面包" + count++ + "号");
                    // 休眠一段时间
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                } else {
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                // 唤醒其他线程
                lock.notify();
            }
        }
    }
}

//消费者
public class Consumer implements Runnable {
    // 传递数据
    private List<String> queue;
    //作为对象锁
    private Object lock;

    public Consumer(List queue, Object lock) {
        this.queue = queue;
        this.lock = lock;
    }


    @Override
    public void run() {
        while (true) {
            synchronized (lock) {
                // 当queue中有数据时，消费数据，否则等待，释放锁
                if (queue != null && queue.size() > 0) {
                    String pro = this.queue.remove(0);
                    System.out.println("消费了" + pro);
                } else {
                    try {
                        this.lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                // 唤醒其他线程
                lock.notify();
            }
        }
    }
}
```

测试
```
//测试
 List<String> queue = new ArrayList<>();
 Object lock = new Object();
 Process process = new Process(queue, lock);
 Consumer consumer = new Consumer(queue, lock);

Thread processThread = new Thread(process);
Thread consumerThread = new Thread(consumer);
processThread.start();
consumerThread.start();
```
运行结果
```
生产了0
消费了面包0号
生产了1
消费了面包1号
生产了2
消费了面包2号
生产了3
消费了面包3号
生产了4
消费了面包4号
...
```
有以下几点想进行说明
1. 同步时需要使用同一把对象锁
2.  wait时会释放锁,然后调用notify方法可以唤醒一个线程

##### protected void finalize() throws Throwable
当垃圾收集确定不再有对该对象的引用时，垃圾收集器在对象上调用该方法。 一个子类覆盖了处理系统资源或执行其他清理的finalize方法,下面举个栗子
```
class Test {
    @Override
    protected void finalize() throws Throwable {
//        super.finalize();
        System.out.println("我被调用了");
    }
}


```
测试
```
Test test = new Test();
test = null;
// 一般是不使用gc方法的，因为这个方法并不一定会导致JVM发生垃圾回收，存在不确定性，这里是为了测试
System.gc();
Thread.sleep(5000);
```
运行结果
```
我被调用了
```

需要注意的是finalize（）方法的调用，可能会导致对象生命周期被延长，导致内存溢出
