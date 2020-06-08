## ConcurrentHashMap 源码解析

[ConcurrentHashMap源码翻译之类注释与综述部分](ConcurrentHashMap源码翻译之类注释与综述部分.md)

[ConcurrentHashMap源码翻译之基础](ConcurrentHashMap源码翻译之基础.md)

[ConcurrentHashMap源码翻译之核心方法](ConcurrentHashMap源码翻译之核心方法.md)

## ConcurrentHashMap分析之CAS原理

### 锁的产生

锁，是为了解决并行运算时，数据并发读写的安全性问题。

### 锁的分类

在实现锁的技术中，分为乐观锁与悲观锁，悲观锁总是假设坏的情况，认为同一份数据在并发情况下一定会有修改，而乐观锁则相反，认为那份数据不会发生修改，在更新数据时会采用尝试不断重试的方式更新数据。

JAVA中的各种锁都是悲观锁，乐观锁是通过CAS技术实现。

### CAS

CAS(Compare And Swap，比较交换)：多个线程尝试使用CAS同时更新同一份数据时，只有其中一个线程能更新数据的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以重复尝试修改数据的值。CAS 操作中包含三个操作数 —— 需要读写的内存位置（V）、进行比较的预期原值（A）和拟写入的新值(B)。如果内存位置V的值与预期原值A相匹配，那么处理器会自动将该位置值更新为新值B。否则处理器不做任何操作。

### Java中的CAS

Java中CAS操作通过JNI本地方法实现：

```
    unsafe.compareAndSwapInt(this, valueOffset, expect, update)
```

### CAS的缺点

#### 1. ABA问题

> 你拿着一个装满钱的手提箱在飞机场，此时过来了一个火辣性感的美女，然后她很暖昧地挑逗着你，并趁你不注意的时候，把用一个一模一样的手提箱和你那装满钱的箱子调了个包，然后就离开了，你看到你的手提箱还在那，于是就提着手提箱去赶飞机去了。

看似没有问题，其实已经改变过了，解决办法：增加版本号。

#### 2. 循环时间长时开销大

自旋CAS如果更新不成功，就一直重新循环，直到成功，如果长时间不成功，会给CPU带来非常大的执行开销。解决办法：网上查说用处理器提供的pause指令，这块不懂了。

#### 3.只能保证一个共享变量的原子操作

java中的AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。这个类目前还没主动的使用过。

## ConcurrentHashMap分析之数据结构(JDK7与JDK8)

ConcurrentHashMap的数据结构奠定了其高并发编程中的作用。其基本的数据结构还是离不开其本源HashMap。比如JDK8中他们同样用了数组+链表+红黑树，只不过在ConcurrentHashMap中增加了一些无锁技术，来实现多线程操作容器。

#### JDK7

先贴两张盗的图

[![201910301005\_1.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/201910301005_2.png)](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/javaCore/javaConcurrent/201910301005_1.png)

1.7-1.png

[![201910301005\_2.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/201910301005_2.png)](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/javaCore/javaConcurrent/201910301005_2.png)

1.7-2.png

从图中可以看出，其数据结构主要由Segment数组+HashEntry数组+链表组成，也就是说定位一个数据时，要通过两次Hash计算。

JDK7的ConcurrentHashMap主要通过锁分段技术实现对资源的并发访问。Segment数组中的每个Segment，都是继承自ReentrantLock （可重入锁，后期再写一个锁的专题），即其锁的粒度为Segment。

在一个线成获取到这个Segment的锁的时候，其他线成可以访问别的Segment，以此增加了容器处理的吞吐量。

#### JDK8

JDK8的HashMap中比JDK7多了一个红黑树以用来增加数据分布的平衡性。JDK8的ConcurrentHashMap相对于JDK7来说，锁的粒度变小了，也就是说并发性更大了。用了synchronized+CAS代替了ReentrantLock ，数据结构也简单了一些。
贴一下盗的图：

[![201910301005\_3.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/201910301005_2.png)](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/javaCore/javaConcurrent/201910301005_3.png)

1.8-1.png

从图上可以看出，数据结构相对比较简单，主要由数组与链表+红黑树组成，锁的粒度就是各个链表首节点。

#### 总结

1. ##### 锁粒度：

JDK8的锁粒度相比于JDK7降低了,JDK1.7版本锁的粒度是基于Segment的，包含多个HashEntry，而JDK1.8锁的粒度就是HashEntry（首节点）.

1. ##### 代码复杂度

JDK8的虽然去掉了分段锁的概念，即数据结构变得简单了，但是相应的代码复杂度就上来了，比如红黑树

1. ##### 锁的方式

JDK8采用synchronized+CAS代替了ReentrantLock，这块的原因可能就是synchronized比较受重视。

- 因为粒度降低了，在相对而言的低粒度加锁方式，synchronized并不比ReentrantLock差，在粗粒度加锁中ReentrantLock可能通过Condition来控制各个低粒度的边界，更加的灵活，而在低粒度中，Condition的优势就没有了。
- 基于JVM的synchronized优化空间更大，使用内嵌的关键字比使用API更加自然。
- 在大量的数据操作下，对于JVM的内存压力，基于API的ReentrantLock会开销更多的内存。

## ConcurrentHashMap分析之tabAt-casTabAt-setTabAt

ConcurrentHashMap中有大量的CAS操作，tabAt、casTabAt、setTabAt是用来实现CAS的方法。
在这只是简单的学习了解，不做深层次的学习，比如为啥用sun.misc.Unsafe，绕过JVM优化数组操作之类的。

### tabAt

获取数组tab中索引为i的节点。

```
     static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
            //native 方法 获取当前节点
            return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
        }
```

### casTabAt

用CAS方法设置数组tab中索引为i的节点的值，如果当前节点与期望的节点c相同，则更新为节点v。

```
     static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                            Node<K,V> c, Node<K,V> v) {
            //native 方法 cas 获取tab数组中索引为i的节点，如果是和c相同，则更新为节点v
            return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
        }
```

### setTabAt

更新 数组tab中索引为i的节点 为 节点v。

```
    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
            // native 方法 更新节点
            U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
        }
```

上面这三个方法都是native方法，依赖于未开源的sun.misc.Unsafe类，这个类太强大了，而且好多都是绕开JVM的方法，以后要好好研究一下。

## ConcurrentHashMap分析之get

先贴一段代码，根据源码分析ConcurrentHashMap中的get方法：

```
        /**
         * Returns the value to which the specified key is mapped,
         * or {@code null} if this map contains no mapping for the key.
         *
         * 返回指定键映射到的值，如果此映射不包含键的映射，则返回{@code null}。
         *
         * <p>More formally, if this map contains a mapping from a key
         * {@code k} to a value {@code v} such that {@code key.equals(k)},
         * then this method returns {@code v}; otherwise it returns
         * {@code null}.  (There can be at most one such mapping.)
         *
         * 更正式地说，如果这个映射包含一个键{@code k}到一个值{@code v}的映射，
         * 使得{@code key.equals(k)}，那么这个方法返回{@code v};否则返回{@code null}。
         * (最多可以有一个这样的映射。)
         *
         * @throws NullPointerException if the specified key is null 如果指定的键为空
         */
        public V get(Object key) {
            Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
            //扩展key的Hash值
            int h = spread(key.hashCode());
            //如果表格不为空且表格长度大于0且所查key的所在节点不为空，(n - 1) & h相当于取模操作，即获取其索引位置。
            if ((tab = table) != null && (n = tab.length) > 0 &&
                (e = tabAt(tab, (n - 1) & h)) != null) {
                //如果获取到当前值
                if ((eh = e.hash) == h) {
                    if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                        //返回当前节点值
                        return e.val;
                }
                //如果当前节点hash值小于0，溢出的时候为负数
                else if (eh < 0)
                    //调用find方法，返回值，或者没找到则返回空
                    return (p = e.find(h, key)) != null ? p.val : null;
                //获取当前节点的下一个节点，如果非空，则判断是否相同，找到了即返回其值。
                while ((e = e.next) != null) {
                    if (e.hash == h &&
                        ((ek = e.key) == key || (ek != null && key.equals(ek))))
                        return e.val;
                }
            }
            return null;
        }
```

节点Node

```
      /* ---------------- Nodes -------------- */

        /**
         * Key-value entry.  This class is never exported out as a
         * user-mutable Map.Entry (i.e., one supporting setValue; see
         * MapEntry below), but can be used for read-only traversals used
         * in bulk tasks.  Subclasses of Node with a negative hash field
         * are special, and contain null keys and values (but are never
         * exported).  Otherwise, keys and vals are never null.
         * 键值项。这个类永远不会作为用户可变的Map.Entry导出，但是可以用于批量任务中使用的只读遍历。
         * 具有负哈希字段的Node的子类是特殊的，包含空键和值(但从不导出)。否则，键和val永远不会为空。
         */
        static class Node<K,V> implements Entry<K,V> {
            //哈希码
            final int hash;
            //键
            final K key;
            //值
            volatile V val;
            //下一个节点
            volatile Node<K,V> next;

            Node(int hash, K key, V val, Node<K,V> next) {
                this.hash = hash;
                this.key = key;
                this.val = val;
                this.next = next;
            }

            public final K getKey()       { return key; }
            public final V getValue()     { return val; }
            public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
            public final String toString(){ return key + "=" + val; }
            public final V setValue(V value) {
                throw new UnsupportedOperationException();
            }

            public final boolean equals(Object o) {
                Object k, v, u; Entry<?,?> e;
                return ((o instanceof Map.Entry) &&
                        (k = (e = (Entry<?,?>)o).getKey()) != null &&
                        (v = e.getValue()) != null &&
                        (k == key || k.equals(key)) &&
                        (v == (u = val) || v.equals(u)));
            }

            /**
             * Virtualized support for map.get(); overridden in subclasses.
             * 对map.get()的虚拟化支持;在子类覆盖。
             * h:待查找key的hash
             * k:待查找key
             */
            Node<K,V> find(int h, Object k) {
                //当前节点
                Node<K,V> e = this;
                //当如果当前节点不为空
                if (k != null) {
                    //迭代节点内的元素，获取制定的键的值
                    do {
                        K ek;
                        if (e.hash == h &&
                            ((ek = e.key) == k || (ek != null && k.equals(ek))))
                            return e;
                    } while ((e = e.next) != null);
                }
                return null;
            }
        }
```

从源码可以看出，ConcurrentHashMap中的get方法是没有锁的。
分析一下get方法是怎么实现的多线程下数据可见性。

1. 通过tabAt的volatile读
2. 通过结点Node的volatile属性next
3. 通过结点Node的volatile属性val

## ConcurrentHashMap分析之putVal

基于JDK8的ConcurrentHashMap中的putVal源码及注释：

```
     /** Implementation for put and putIfAbsent
         * 实现put和putIfAbsent
         * onlyIfAbsent含义：如果我们传入的key已存在我们是否去替换，true:不替换，false：替换。
         * */
        final V putVal(K key, V value, boolean onlyIfAbsent) {
            //键值都不为空
            if (key == null || value == null) throw new NullPointerException();
            //发散键的hash值
            int hash = spread(key.hashCode());
            //桶数量 0
            int binCount = 0;
            for (Node<K,V>[] tab = table;;) {
                Node<K,V> f; int n, i, fh;
                //检查表是否初始化
                if (tab == null || (n = tab.length) == 0)
                    //初始化表
                    tab = initTable();
                //检查指定键所在节点是否为空
                else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                    //通过CAS方法添加键值对
                    if (casTabAt(tab, i, null,
                                 new Node<K,V>(hash, key, value, null)))
                        break;                   // no lock when adding to empty bin 添加到空桶时没有锁
                }
                //如果当前节点的Hash值为MOVED
                else if ((fh = f.hash) == MOVED)
                    //如果还在进行扩容操作就先进行扩容
                    tab = helpTransfer(tab, f);
                else {
                    V oldVal = null;
                    //synchronized锁定此f节点
                    synchronized (f) {
                        //再次检测节点是否相同
                        if (tabAt(tab, i) == f) {
                            //如果此节点hash值大于0
                            if (fh >= 0) {
                                //桶数量为1
                                binCount = 1;
                                for (Node<K,V> e = f;; ++binCount) {
                                    K ek;
                                    //获取到指定的键
                                    if (e.hash == hash &&
                                        ((ek = e.key) == key ||
                                         (ek != null && key.equals(ek)))) {
                                        //保存老的值
                                        oldVal = e.val;
                                        //onlyIfAbsent：如果我们传入的key已存在我们是否去替换，true:不替换，false：替换。
                                        if (!onlyIfAbsent)
                                            //替换掉
                                            e.val = value;
                                        break;
                                    }
                                    Node<K,V> pred = e;
                                    //下一个节点为空
                                    if ((e = e.next) == null) {
                                        //新建节点
                                        pred.next = new Node<K,V>(hash, key,
                                                                  value, null);
                                        break;
                                    }
                                }
                            }
                            //如果时树节点
                            else if (f instanceof TreeBin) {
                                Node<K,V> p;
                                //桶数量为2
                                binCount = 2;
                                //找到插入的位置，插入节点，平衡插入
                                if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                               value)) != null) {
                                    oldVal = p.val;
                                    if (!onlyIfAbsent)
                                        p.val = value;
                                }
                            }
                        }
                    }
                    //如果桶数量不为0
                    if (binCount != 0) {
                        //如果桶数量大于树化阈值
                        if (binCount >= TREEIFY_THRESHOLD)
                            //将链表转为树
                            treeifyBin(tab, i);
                        //如果老的值存在，则返回
                        if (oldVal != null)
                            return oldVal;
                        break;
                    }
                }
            }
            //增加数量
            addCount(1L, binCount);
            return null;
        }
```

这个方法的大概思路就是：

1. 初始化操作
2. 如果待插入键的插入位置没有节点，则通过CAS方式创建一个节点。
3. 如果当前节点的Hash值为 static final int MOVED = -1; // hash for forwarding nodes 转发节点的哈希，则调用helpTransfer方法，如果还在进行扩容操作就先进行扩容，Helps transfer if a resize is in progress。
4. 再有就是发生Hash碰撞时通过synchronized关键字锁定当前节点，这里有两种情况，一种是链表形式：直接遍历到尾端插入，一种是红黑树：按照红黑树结构插入。
5. 计数

这里面有两个地方保证了putVal的并发实现，一个是没有待插入节点时的CAS技术，一个是发现有存在Hash碰撞时的synchronized关键字。

## ConcurrentHashMap分析之replaceNode

replaceNode方法

```
     /**
         * Implementation for the four public remove/replace methods:
         * Replaces node value with v, conditional upon match of cv if
         * non-null.  If resulting value is null, delete.
         * 删除/替换的操作
         *
         */
        final V replaceNode(Object key, V value, Object cv) {
            //发散hash
            int hash = spread(key.hashCode());
            //迭代表
            for (Node<K,V>[] tab = table;;) {
                Node<K,V> f; int n, i, fh;
                //如果表为空，且键所在节点为空，跳出循环
                if (tab == null || (n = tab.length) == 0 ||
                    (f = tabAt(tab, i = (n - 1) & hash)) == null)
                    break;
                //如果key所在节点的hash值为MOVED
                else if ((fh = f.hash) == MOVED)
                    //如果还在进行扩容操作就先进行扩容
                    tab = helpTransfer(tab, f);
                else {
                    V oldVal = null;
                    //是否检验
                    boolean validated = false;
                    //锁当前节点
                    synchronized (f) {
                        //确认节点
                        if (tabAt(tab, i) == f) {
                            //节点的hash值大于0
                            if (fh >= 0) {
                                //校验
                                validated = true;
                                for (Node<K,V> e = f, pred = null;;) {
                                    K ek;
                                    //找到键位置
                                    if (e.hash == hash &&
                                        ((ek = e.key) == key ||
                                         (ek != null && key.equals(ek)))) {
                                        //老的值
                                        V ev = e.val;
                                        //删除节点
                                        if (cv == null || cv == ev ||
                                            (ev != null && cv.equals(ev))) {
                                            oldVal = ev;
                                            if (value != null)
                                                e.val = value;
                                            else if (pred != null)
                                                pred.next = e.next;
                                            else
                                                setTabAt(tab, i, e.next);
                                        }
                                        break;
                                    }
                                    pred = e;
                                    if ((e = e.next) == null)
                                        break;
                                }
                            }
                            //是树形节点
                            else if (f instanceof TreeBin) {
                                validated = true;
                                TreeBin<K,V> t = (TreeBin<K,V>)f;
                                TreeNode<K,V> r, p;
                                //找到删除的key
                                if ((r = t.root) != null &&
                                    (p = r.findTreeNode(hash, key, null)) != null) {
                                    V pv = p.val;
                                    if (cv == null || cv == pv ||
                                        (pv != null && cv.equals(pv))) {
                                        oldVal = pv;
                                        if (value != null)
                                            p.val = value;
                                        //删除树节点
                                        else if (t.removeTreeNode(p))
                                            setTabAt(tab, i, untreeify(t.first));
                                    }
                                }
                            }
                        }
                    }
                    //检查
                    if (validated) {
                        if (oldVal != null) {
                            if (value == null)
                                //数量减一
                                addCount(-1L, -1);
                            return oldVal;
                        }
                        break;
                    }
                }
            }
            return null;
        }
```

此方法通过以下三处实现多线程的并发访问：

1. 头节点的synchronized锁。
2. Node节点的`volatile Node<K,V> next`属性，用了volatile读。
3. Node节点的`volatile V val`属性，用了volatile读。

再盗取一张图：

[![201910301009\_1.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/201910301009_1.png)](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/javaCore/javaConcurrent/201910301009_1.png)

remove.png

> 如图所示：删除的node节点的next依然指着下一个元素。此时若有一个遍历线程正在遍历这个已经删除的节点，这个遍历线程依然可以通过next属性访问下一个元素。从遍历线程的角度看，他并没有感知到此节点已经删除了，这说明了ConcurrentHashMap提供了弱一致性的迭代器。

## ConcurrentHashMap分析之计数

//单独开一篇讲解

[ConcurrentHashMap分析之计数](ConcurrentHashMap分析之计数.md)

## ConcurrentHashMap分析之初始化

JDK8的ConcurrentHashMap的初始化源码及注释：

```
        /**
         * Initializes table, using the size recorded in sizeCtl.
         * 使用sizeCtl中记录的大小初始化表。
         */
        private final Node<K,V>[] initTable() {
            Node<K,V>[] tab; int sc;
            //只要表为空，就一直循环
            while ((tab = table) == null || tab.length == 0) {
                //如果sizeCtl小于0，
                if ((sc = sizeCtl) < 0)
                    //用了yield方法后，该线程就会把CPU时间让掉，让其他或者自己的线程执行（也就是谁先抢到谁执行）
                    Thread.yield(); // lost initialization race; just spin 失去了初始化CPU竞争;只是自旋
                else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) { //如果SIZECTL与sc相同，则把SIZECTL设置为-1，即当前线程获取到了初始化的工作。
                    try {
                        //再次确认 表为空
                        if ((tab = table) == null || tab.length == 0) {
                            //如果初始化size大于0，则n为sc,否则为16.
                            int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                            //创建n个Node数组
                            @SuppressWarnings("unchecked")
                            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                            //table指向空数组
                            table = tab = nt;
                            // 如果 n 为 16 的话，那么这里 sc = 12
                            // 其实就是 0.75 * n
                            sc = n - (n >>> 2);
                        }
                    } finally {
                        //sizeCtl重新定义新的值，用于扩容。
                        sizeCtl = sc;
                    }
                    break;
                }
            }
            return tab;
        }
```

由源码可知：初始化的代码比较简单，通过初始化数组，之后将sizeCtl赋值。
这里涉及到的知识点：

1. Thread.yield()：使当前线程从执行状态（运行状态）变为可执行态（就绪状态）。
2. U.compareAndSwapInt(this, SIZECTL, sc, -1)和SIZECTL = U.objectFieldOffset(k.getDeclaredField("sizeCtl"))：通过反射获取sizeCtl，再通过CAS设置其值。
3. sc = n - (n >>> 2)：其实就是 0.75 * n，扩容阈值。

这个方法并发通过对SIZECTL+CAS实现。

## ConcurrentHashMap分析之扩容

[ConcurrentHashMap分析之扩容](ConcurrentHashMap分析之扩容.md)