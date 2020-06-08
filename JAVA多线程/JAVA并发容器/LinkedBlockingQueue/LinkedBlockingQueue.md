## LinkedBlockingQueue源码解析

[LinkedBlockingQueue](LinkedBlockingQueue源码.md)

### 特点

- 是先进先出队列FIFO。
- 采用ReentrantLock保证线程安全

### 功能

#### 增加

增加有三种方式，前提：队列满

| 方式 |   put    |  add   |   offer   |
| :--: | :------: | :----: | :-------: |
| 特点 | 一直阻塞 | 抛异常 | 返回false |

#### 删除

删除有三种方式，前提：队列为空

| 方式 |         remove         |   poll    | take |
| :--: | :--------------------: | :-------: | :--: |
| 特点 | NoSuchElementException | 返回false | 阻塞 |

#### 定义与性质

- LinkedBlockingQueue是一个阻塞队列，内部由两个ReentrantLock来实现出入队列的线程安全，由各自的Condition对象的await和signal来实现等待和唤醒功能。

- 基于单向链表的、范围任意的（其实是有界的）、FIFO 阻塞队列。

- 头结点和尾结点一开始总是指向一个哨兵的结点，它不持有实际数据，当队列中有数据时，头结点仍然指向这个哨兵，尾结点指向有效数据的最后一个结点。这样做的好处在于，与计数器 count 结合后，对队头、队尾的访问可以独立进行，而不需要判断头结点与尾结点的关系。

  [![2019103010016\_1.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/2019103010016_1.png)](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/javaCore/javaConcurrent/2019103010016_1.png)

  继承图谱

#### 节点与属性

```
        /**
         * Linked list node class
         * 链表节点内部类
         */
        static class Node<E> {
            //节点元素
            E item;

            /**
             * One of:
             * 之一：
             * - the real successor Node
             * 真正的继承节点
             * - this Node, meaning the successor is head.next
             * 这个节点表示继承节点是head.next
             * - null, meaning there is no successor (this is the last node)
             * null,表示没有继承节点，它是尾节点
             */
            Node<E> next;

            Node(E x) {
                item = x;
            }
        }

        /**
         * The capacity bound, or Integer.MAX_VALUE if none
         * 容量界限，如果未设定，则为Integer最大值
         */
        private final int capacity;

        /**
         * Current number of elements
         * 当前元素个数
         */
        private final AtomicInteger count = new AtomicInteger();

        /**
         * Head of linked list.
         * 链表的头
         * Invariant: head.item == null
         * 不变量：head.item == null
         */
        transient Node<E> head;

        /**
         * Tail of linked list.
         * 链表的尾
         * Invariant: last.next == null
         * 不变量：last.next == null
         */
        private transient Node<E> last;

        /**
         * Lock held by take, poll, etc
         * take,poll等获取锁
         */
        private final ReentrantLock takeLock = new ReentrantLock();

        /**
         * Wait queue for waiting takes
         * 等待任务的等待队列
         */
        private final Condition notEmpty = takeLock.newCondition();

        /**
         * Lock held by put, offer, etc
         * put，offer等插入锁
         */
        private final ReentrantLock putLock = new ReentrantLock();

        /**
         * Wait queue for waiting puts
         * 等待插入的等待队列
         */
        private final Condition notFull = putLock.newCondition();
```

#### 插入线程与获取线程的相互通知

其中signalNotEmpty()方法，在插入线程发现队列为空时调用，告知获取线程需要等待。
signalNotFull()方法在获取线程发现队列已满时调用，告知插入线程需要等待。

```
     /**
         * Signals a waiting take. Called only from put/offer (which do not
         * otherwise ordinarily lock takeLock.)
         * 表示等待take。put/offer调用，否则通常不会锁定takeLock。
         */
        private void signalNotEmpty() {
            //获取takeLock
            final ReentrantLock takeLock = this.takeLock;
            //锁定takeLock
            takeLock.lock();
            try {
                //唤醒take线程等待队列
                notEmpty.signal();
            } finally {
                //释放锁
                takeLock.unlock();
            }
        }

        /**
         * Signals a waiting put. Called only from take/poll.
         * 表示等待put,take/poll 调用
         */
        private void signalNotFull() {
            //获取putLock
            final ReentrantLock putLock = this.putLock;
            //锁定putLock
            putLock.lock();
            try {
                //唤醒插入线程等待队列
                notFull.signal();
            } finally {
                //释放锁
                putLock.unlock();
            }
        }
```

#### 入队与出对操作

enqueue()方法只能在持有 putLock 锁下执行，dequeue()在持有 takeLock 锁下执行。

```
     /**
         * Links node at end of queue.
         * 在队列尾部插入
         *
         * @param node the node
         */
        private void enqueue(Node<E> node) {
            // assert putLock.isHeldByCurrentThread();
            // assert last.next == null;
            //last.next指向当前node
            //尾指针后移
            last = last.next = node;
        }

        /**
         * Removes a node from head of queue.
         * 移除队列头
         *
         * @return the node
         */
        private E dequeue() {
            // assert takeLock.isHeldByCurrentThread();
            // assert head.item == null;
            //保存头指针
            Node<E> h = head;
            //获取当前链表第一个元素
            Node<E> first = h.next;
            //头指针的next指向自己
            h.next = h; // help GC
            //头指针指向第一个元素
            head = first;
            //获取第一个元素的值
            E x = first.item;
            //将第一个元素的值置空
            first.item = null;
            //返回第一个元素的值
            return x;
        }
```

#### 对两把锁的加锁与释放

在需要对两把锁同时加锁时，把加锁的顺序与释放的顺序封装成方法，确保所有地方都是一致的。
而且获取锁时都是不响应中断的，一直获取直到加锁成功，这就避免了第一把锁加锁成功，而第二把锁加锁失败导致锁不释放的风险。

```
       /**
         * Locks to prevent both puts and takes.
         * 锁定putLock和takeLock
         */
        void fullyLock() {
            putLock.lock();
            takeLock.lock();
        }

        /**
         * Unlocks to allow both puts and takes.
         * 与fullyLock的加锁顺序相反，先解锁takeLock，再解锁putLock
         */
        void fullyUnlock() {
            takeLock.unlock();
            putLock.unlock();
        }
```





#### 简单介绍一下LinkedBlockingQueue中API的源码

如构造方法，新增，获取，删除，drainTo。

#### 构造函数

LinkedBlockingQueue有三个构造方法，其中无参构造尽量少用，因为容量为Integer的最大值，操作不当会出现内存溢出。

```
     /**
         * Creates a {@code LinkedBlockingQueue} with a capacity of
         * {@link Integer#MAX_VALUE}.
         * 创建一个容量为Integer最大值的LinkedBlockingQueue
         */
        public LinkedBlockingQueue() {
            this(Integer.MAX_VALUE);
        }

        /**
         * Creates a {@code LinkedBlockingQueue} with the given (fixed) capacity.
         * 创建一个指定容量的LinkedBlockingQueue
         *
         * @param capacity the capacity of this queue
         * @throws IllegalArgumentException if {@code capacity} is not greater
         *                                  than zero
         */
        public LinkedBlockingQueue(int capacity) {
            //参数校验
            if (capacity <= 0) throw new IllegalArgumentException();
            //设置容量
            this.capacity = capacity;
            //首尾节点指向一个空节点
            last = head = new Node<E>(null);
        }

        /**
         * Creates a {@code LinkedBlockingQueue} with a capacity of
         * {@link Integer#MAX_VALUE}, initially containing the elements of the
         * given collection,
         * added in traversal order of the collection's iterator.
         * 创建一个{@code LinkedBlockingQueue}，其容量为{@link Integer#MAX_VALUE}，
         * 最初包含给定集合的元素，按集合迭代器的遍历顺序添加。
         *
         * @param c the collection of elements to initially contain
         * @throws NullPointerException if the specified collection or any
         *                              of its elements are null
         */
        public LinkedBlockingQueue(Collection<? extends E> c) {
            this(Integer.MAX_VALUE);
            //获取putLock
            final ReentrantLock putLock = this.putLock;
            //锁定
            putLock.lock(); // Never contended, but necessary for visibility
            try {
                int n = 0;
                for (E e : c) {
                    if (e == null)
                        throw new NullPointerException();
                    if (n == capacity)
                        throw new IllegalStateException("Queue full");
                    enqueue(new Node<E>(e));
                    ++n;
                }
                count.set(n);
            } finally {
                putLock.unlock();
            }
        }
```

#### offer(E e)

将给定的元素设置到队列中，如果设置成功返回true, 否则返回false. e的值不能为空，否则抛出空指针异常。

```
     /**
         * Inserts the specified element at the tail of this queue if it is
         * possible to do so immediately without exceeding the queue's capacity,
         * returning {@code true} upon success and {@code false} if this queue
         * is full.
         * <p>
         * 如果可以在不超过队列容量的情况下立即插入指定的元素到队列的尾部，
         * 成功后返回{@code true}，如果队列已满，返回{@code false}。
         * <p>
         * When using a capacity-restricted queue, this method is generally
         * preferable to method {@link BlockingQueue#add add}, which can fail to
         * insert an element only by throwing an exception.
         * <p>
         * 当使用容量受限的队列时，此方法通常比方法{@link BlockingQueue#add add}更可取，后者只能通过抛出异常才能插入元素。
         * 非阻塞
         *
         * @throws NullPointerException if the specified element is null
         */
        public boolean offer(E e) {
            //非空判断
            if (e == null) throw new NullPointerException();
            //计数器
            final AtomicInteger count = this.count;
            //如果队列已满，直接返回插入失败
            if (count.get() == capacity)
                return false;
            int c = -1;
            //新建节点
            Node<E> node = new Node<E>(e);
            //获取插入锁
            final ReentrantLock putLock = this.putLock;
            //锁定
            putLock.lock();
            try {
                //如果队列未满
                if (count.get() < capacity) {
                    //插入队列
                    enqueue(node);
                    ///计数
                    c = count.getAndIncrement();
                    //还有空余空间
                    if (c + 1 < capacity)
                        //唤醒插入线程
                        notFull.signal();
                }
            } finally {
                //解锁
                putLock.unlock();
            }
            //如果队列为空
            if (c == 0)
                //通知获取线程阻塞
                signalNotEmpty();
            //返回成功或者插入失败
            return c >= 0;
        }
```

#### offer(E e, long timeout, TimeUnit unit)

将给定元素在给定的时间内设置到队列中，如果设置成功返回true, 否则返回false.

```
     /**
         * Inserts the specified element at the tail of this queue, waiting if
         * necessary up to the specified wait time for space to become available.
         * <p>
         * 将指定的元素插入到此队列的末尾，如果需要，将等待到指定的等待时间，直到空间可用为止。
         *
         * @return {@code true} if successful, or {@code false} if
         * the specified waiting time elapses before space is available
         * @throws InterruptedException {@inheritDoc}
         * @throws NullPointerException {@inheritDoc}
         */
        public boolean offer(E e, long timeout, TimeUnit unit)
                throws InterruptedException {

            //不允许空元素
            if (e == null) throw new NullPointerException();
            //等待时间
            long nanos = unit.toNanos(timeout);
            //本地变量
            int c = -1;
            //获取putLock
            final ReentrantLock putLock = this.putLock;
            //获取计数器
            final AtomicInteger count = this.count;
            //可中断加锁，即在锁获取过程中不处理中断状态，而是直接抛出中断异常，由上层调用者处理中断。
            putLock.lockInterruptibly();
            try {
                //只要队列已满
                while (count.get() == capacity) {
                    //如果超时，则返回插入失败
                    if (nanos <= 0)
                        return false;
                    //导致当前线程等待，直到发出信号或中断，或指定的等待时间过期。
                    nanos = notFull.awaitNanos(nanos);
                }
                //入队列
                enqueue(new Node<E>(e));
                //计数
                c = count.getAndIncrement();
                //如果队列增加1个元素还未满
                if (c + 1 < capacity)
                    //唤醒插入进程
                    notFull.signal();
            } finally {
                //解锁
                putLock.unlock();
            }

            //如果队列中没有元素了
            if (c == 0)
                //通知获取线程等待
                signalNotEmpty();
            //返回插入成功
            return true;
        }
```

#### put(E e)

将元素设置到队列中，如果队列中没有多余的空间，该方法会一直阻塞，直到队列中有多余的空间。

```
     /**
         * Inserts the specified element at the tail of this queue, waiting if
         * necessary for space to become available.
         * <p>
         * 将指定的元素插入到此队列的末尾，如果需要，则等待空间可用。
         * 阻塞
         *
         * @throws InterruptedException {@inheritDoc}
         * @throws NullPointerException {@inheritDoc}
         */
        public void put(E e) throws InterruptedException {
            //不可以插入空元素
            if (e == null) throw new NullPointerException();
            // Note: convention in all put/take/etc is to preset local var
            //所有put/take/etc中的约定都是预先设置本地var
            // holding count negative to indicate failure unless set.
            //除非设置，否则保持计数为负数表示失败。
            int c = -1;
            //新建节点
            Node<E> node = new Node<E>(e);
            //获取putLock
            final ReentrantLock putLock = this.putLock;
            //获取计数器
            final AtomicInteger count = this.count;

            //可中断加锁，即在锁获取过程中不处理中断状态，而是直接抛出中断异常，由上层调用者处理中断。
            putLock.lockInterruptibly();
            try {
                /*
                 * Note that count is used in wait guard even though it is
                 * not protected by lock. This works because count can
                 * only decrease at this point (all other puts are shut
                 * out by lock), and we (or some other waiting put) are
                 * signalled if it ever changes from capacity. Similarly
                 * for all other uses of count in other wait guards.
                 * 注意count在wait守卫线程中使用，即使它没有被锁保护。
                 * 这是因为count只能在此时减少(所有其他put都被锁定关闭)，
                 * 如果它从容量更改，我们(或其他一些等待put)将收到信号。
                 * 类似地，count在其他等待守卫线程中的所有其他用途也是如此。
                 */
                //只要当前队列已满
                while (count.get() == capacity) {
                    //通知插入线程等待
                    notFull.await();
                }
                //插入队列
                enqueue(node);
                //数量加1
                c = count.getAndIncrement();
                //如果队列增加1个元素还未满
                if (c + 1 < capacity)
                    //唤醒插入进程
                    notFull.signal();
            } finally {
                //解锁
                putLock.unlock();
            }
            //如果队列中没有元素了
            if (c == 0)
                //通知获取线程等待
                signalNotEmpty();
        }
```

#### take()

从队列中获取值，如果队列中没有值，线程会一直阻塞，直到队列中有值，并且该方法取得了该值。

```
    /**
         * 阻塞方式获取队列中数据
         *
         * @return
         * @throws InterruptedException
         */
        public E take() throws InterruptedException {
            E x;
            int c = -1;
            final AtomicInteger count = this.count;
            final ReentrantLock takeLock = this.takeLock;
            //会抛出中断异常的获取锁的方法
            takeLock.lockInterruptibly();
            try {
                //阻塞
                while (count.get() == 0) {
                    notEmpty.await();
                }
                //出队列
                x = dequeue();
                //数量减一
                c = count.getAndDecrement();
                if (c > 1)
                    //通知下一个获取线程
                    notEmpty.signal();
            } finally {
                takeLock.unlock();
            }
            //队列已满
            if (c == capacity)
                //通过插入线程阻塞
                signalNotFull();
            return x;
        }
```

#### peek()

非阻塞的获取队列中的第一个元素，不出队列。

```
      /**
         * 非阻塞，不出队列即不删除第一个元素
         *
         * @return
         */
        public E peek() {
            //队列为空，直接返回
            if (count.get() == 0)
                return null;
            final ReentrantLock takeLock = this.takeLock;
            takeLock.lock();
            try {
                //获取第一个元素，非哨兵
                Node<E> first = head.next;
                //元素为空，返回null
                if (first == null)
                    return null;
                else
                    //返回第一个元素值
                    return first.item;
            } finally {
                takeLock.unlock();
            }
        }
```

#### poll()

非阻塞的获取队列中的值，未获取到返回null。

```
      /**
         * 非阻塞获取元素
         *
         * @return
         */
        public E poll() {
            final AtomicInteger count = this.count;
            //队列为空，直接返回
            if (count.get() == 0)
                return null;
            E x = null;
            int c = -1;
            final ReentrantLock takeLock = this.takeLock;
            takeLock.lock();
            try {
                //队列非空，获取队列中元素
                if (count.get() > 0) {
                    x = dequeue();
                    c = count.getAndDecrement();
                    if (c > 1)
                        notEmpty.signal();
                }
            } finally {
                takeLock.unlock();
            }
            if (c == capacity)
                signalNotFull();
            return x;
        }
```

#### poll(long timeout, TimeUnit unit)

在给定的时间里，从队列中获取值，如果没有取到会抛出异常。

```
     /**
         * 等待一定时间后，未获取到队列元素时，返回null
         *
         * @param timeout
         * @param unit
         * @return
         * @throws InterruptedException
         */
        public E poll(long timeout, TimeUnit unit) throws InterruptedException {
            E x = null;
            int c = -1;
            //时间转换
            long nanos = unit.toNanos(timeout);
            final AtomicInteger count = this.count;
            final ReentrantLock takeLock = this.takeLock;
            //可中断
            takeLock.lockInterruptibly();
            try {
                //等待一定时间
                while (count.get() == 0) {
                    if (nanos <= 0)
                        return null;
                    nanos = notEmpty.awaitNanos(nanos);
                }
                //出队列
                x = dequeue();
                //减一
                c = count.getAndDecrement();
                if (c > 1)
                    //唤醒读线程
                    notEmpty.signal();
            } finally {
                takeLock.unlock();
            }
            if (c == capacity)
                //队列已满，通知插入线程阻塞
                signalNotFull();
            return x;
        }
```

#### remove(Object o)

从队列中移除指定的值。将两把锁都锁定。

```
      /**
         * Removes a single instance of the specified element from this queue,
         * if it is present.  More formally, removes an element {@code e} such
         * that {@code o.equals(e)}, if this queue contains one or more such
         * elements.
         * 从此队列中删除指定元素的单个实例(如果存在)。
         * Returns {@code true} if this queue contained the specified element
         * (or equivalently, if this queue changed as a result of the call).
         * 如果此队列包含指定的元素，则返回{@code true}(如果此队列由于调用而更改，则返回相同的值)。
         * @param o element to be removed from this queue, if present
         * @return {@code true} if this queue changed as a result of the call
         */
        public boolean remove(Object o) {
            //不支持null
            if (o == null) return false;
            //锁定两个锁
            fullyLock();
            try {
                //迭代队列
                for (Node<E> trail = head, p = trail.next;
                     p != null;
                     trail = p, p = p.next) {
                    //通过equals方法匹配待删除元素
                    if (o.equals(p.item)) {
                        //移除p节点
                        unlink(p, trail);
                        //成功
                        return true;
                    }
                }
                //失败
                return false;
            } finally {
                //解锁
                fullyUnlock();
            }
        }
         /**
         * Unlinks interior Node p with predecessor trail.
         * 将内部节点p与前一个跟踪断开连接。
         */
        void unlink(Node<E> p, Node<E> trail) {
            // assert isFullyLocked();
            // p.next is not changed, to allow iterators that are
            // traversing p to maintain their weak-consistency guarantee.
            //p节点内容置空
            p.item = null;
            //trail节点的next指向p的next
            trail.next = p.next;
            //如果p是队尾
            if (last == p)
                //trail变为队尾
                last = trail;
            //如果队列已满
            if (count.getAndDecrement() == capacity)
                //通知插入线程阻塞
                notFull.signal();
        }
```

#### clear()

清空队列。

```
     /**
         * Atomically removes all of the elements from this queue.
         * The queue will be empty after this call returns.
         * 原子性地从队列中删除所有元素。此调用返回后，队列将为空。
         */
        public void clear() {
            //锁定
            fullyLock();
            try {
                //清空数据，帮助垃圾回收
                for (Node<E> p, h = head; (p = h.next) != null; h = p) {
                    h.next = h;
                    p.item = null;
                }
                head = last;
                // assert head.item == null && head.next == null;
                //如果容量为0
                if (count.getAndSet(0) == capacity)
                    //唤醒插入线程
                    notFull.signal();
            } finally {
                //解锁
                fullyUnlock();
            }
        }
```

#### drainTo(Collection c)

将队列中值，全部移除，并发设置到给定的集合中。

```
     /**
         * 将队列中值，全部移除，并发设置到给定的集合中
         * @throws UnsupportedOperationException {@inheritDoc}
         * @throws ClassCastException            {@inheritDoc}
         * @throws NullPointerException          {@inheritDoc}
         * @throws IllegalArgumentException      {@inheritDoc}
         */
        public int drainTo(Collection<? super E> c, int maxElements) {
            //各种判断
            if (c == null)
                throw new NullPointerException();
            if (c == this)
                throw new IllegalArgumentException();
            if (maxElements <= 0)
                return 0;
            boolean signalNotFull = false;
            //锁
            final ReentrantLock takeLock = this.takeLock;
            takeLock.lock();
            try {
                //获取要转移的数量
                int n = Math.min(maxElements, count.get());
                // count.get provides visibility to first n Nodes
                Node<E> h = head;
                int i = 0;
                try {
                    //组装集合
                    while (i < n) {
                        Node<E> p = h.next;
                        c.add(p.item);
                        p.item = null;
                        h.next = h;
                        h = p;
                        ++i;
                    }
                    return n;
                } finally {
                    // Restore invariants even if c.add() threw
                    if (i > 0) {
                        // assert h.item == null;
                        head = h;
                        signalNotFull = (count.getAndAdd(-i) == capacity);
                    }
                }
            } finally {
                takeLock.unlock();
                if (signalNotFull)
                    signalNotFull();
            }
        }
```