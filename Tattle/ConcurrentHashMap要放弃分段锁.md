# ConcurrentHashMap要放弃分段锁

## 背景

我们都知道 HashMap 是一个线程不安全的类，多线程环境下，使用 HashMap 进行put操作会引起死循环，导致CPU利用率接近100%，所以如果你的并发量很高的话，所以是不推荐使用 HashMap 的。

而我们所知的，HashTable 是线程安全的，但是因为 HashTable 内部使用的 synchronized 来保证线程的安全，所以，在多线程情况下，HashTable 虽然线程安全，但是他的效率也同样的比较低下。

所以就出现了一个效率相对来说比 HashTable 高，但是还比 HashMap 安全的类，那就是 ConcurrentHashMap，而 ConcurrentHashMap 在 JDK8 中却放弃了使用分段锁，为什么呢？那他之后是使用什么来保证线程安全呢？我们今天来看看。

## 什么是分段锁

其实这个分段锁很容易理解，既然其他的锁都是锁全部，那分段锁是不是和其他的不太一样，是的，他就相当于把一个方法切割成了很多块，在单独的一块上锁的时候，其他的部分是不会上锁的，也就是说，这一段被锁住，并不影响其他模块的运行，分段锁如果这样理解是不是就好理解了，我们先来看看 JDK7 中的 ConcurrentHashMap 的分段锁的实现。

```java
//初始总容量，默认16
static final int DEFAULT_INITIAL_CAPACITY = 16;
//加载因子，默认0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f;
//并发级别，默认16
static final int DEFAULT_CONCURRENCY_LEVEL = 16;

static final class Segment<K,V> extends ReentrantLock implements Serializable {

transient volatile HashEntry<K,V>[] table;

}
```

`Segment<K,V>`是这个类真正的的主要内容，`ConcurrentHashMap`是由`Segment`数组结构和`HashEntry`数组结构组成.

我们看到了`Segment<K,V>`,而他的内部，又有`HashEntry`数组结构组成. `Segment`继承自 `RentrantLock`在这里充当的是一个锁，而在其内部的`HashEntry`则是用来存储键值对数据.

也就是说，一个`Segment`里包含一个`HashEntry`数组，每个`HashEntry`是一个链表结构的元素，当需要`put`元素的时候，并不是对整个`hashmap`进行加锁，而是先通过`hashcode`来知道他要放在哪一个分段中，然后对这个分段进行加锁。

最后也就出现了，如果不是在同一个分段中的 put 数据，那么`ConcurrentHashMap`就能够保证并行的 `put`，也就是说，在并发过程中，他就是一个线程安全的`Map`。

## 为什么JDK8舍弃掉了分段锁

- 每个锁控制的是一段，当分段很多，并且加锁的分段不连续的时候，内存空间的浪费比较严重。
- 如果某个分段特别的大，那么就会影响效率，耽误时间。

我们得出了`JDK7`中的`ConcurrentHashMap`使用的是`Segment`和`HashEntry`，而在`JDK8`中 `ConcurrentHashMap`就变了，`JDK8`中的 `ConcurrentHashMap`使用的是`synchronized`和 `CAS`和 `HashEntry` 和红黑树。


`ConcurrentHashMap`和`HashMap`一样，使用了红黑树，而在`ConcurrentHashMap`中则是取消了`Segment`分段锁的数据结构，取而代之的是Node数组+链表+红黑树的结构。

```java
/第一次put 初始化 Node 数组
private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }


    public V put(K key, V value) {
        return putVal(key, value, false);
    }

    /** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
                //如果相应位置的Node还未初始化，则通过CAS插入相应的数据
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            ...
           //如果该节点是TreeBin类型的节点，说明是红黑树结构，则通过putTreeVal方法往红黑树中插入节点
           else if (f instanceof TreeBin) {
           Node<K,V> p;
           binCount = 2;
           if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,value)) != null) {
               oldVal = p.val;
               if (!onlyIfAbsent)
                   p.val = value;
           }
           //如果binCount不为0，说明put操作对数据产生了影响，如果当前链表的个数达到8个，则通过treeifyBin方法转化为红黑树，如果oldVal不为空，说明是一次更新操作,返回旧值
           if (binCount != 0) {
               if (binCount >= TREEIFY_THRESHOLD)
                   treeifyBin(tab, i);
               if (oldVal != null)
                   return oldVal;
               break;
           }
       }
        addCount(1L, binCount);
        return null;
    }
```