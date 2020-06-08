基于JDK8，LinkedBlockingQueue源码翻译

```
    /**
     * An optionally-bounded {@linkplain BlockingQueue blocking queue} based on
     * linked nodes.
     * <p>
     * 基于链表节点的有操作边界的阻塞队列
     * <p>
     * This queue orders elements FIFO (first-in-first-out).
     * The <em>head</em> of the queue is that element that has been on the
     * queue the longest time.
     * The <em>tail</em> of the queue is that element that has been on the
     * queue the shortest time. New elements
     * are inserted at the tail of the queue, and the queue retrieval
     * operations obtain elements at the head of the queue.
     * Linked queues typically have higher throughput than array-based queues but
     * less predictable performance in most concurrent applications.
     * <p>
     * 是一个先入先出队列，队列的头是队列中存在时间最长的元素。队列的尾是队列中存在时间最短的元素。
     * 新的元素插入到队列的尾部，队列检索操作获取队列头部的元素。
     * 链接队列通常比基于数组的队列具有更高的吞吐量，但在大多数并发应用程序中，预测性能较差。
     *
     * <p>The optional capacity bound constructor argument serves as a
     * way to prevent excessive queue expansion. The capacity, if unspecified,
     * is equal to {@link Integer#MAX_VALUE}.  Linked nodes are
     * dynamically created upon each insertion unless this would bring the
     * queue above capacity.
     * <p>
     * 可选的容量绑定构造函数参数用作防止过度队列扩展的一种方法。
     * 如果没有指定容量，则为{@link Integer#MAX_VALUE}。
     * 每次插入时都会动态创建链接节点，除非这会使队列超出容量。
     *
     *
     * <p>This class and its iterator implement all of the
     * <em>optional</em> methods of the {@link Collection} and {@link
     * Iterator} interfaces.
     * <p>
     * 这个类和他的iterator实现了集合与迭代器接口的所有操作。
     *
     * <p>This class is a member of the
     * <a href="{@docRoot}/../technotes/guides/collections/index.html">
     * Java Collections Framework</a>.
     *
     * @param <E> the type of elements held in this collection
     * @author Doug Lea
     * @since 1.5
     */
    public class LinkedBlockingQueue<E> extends AbstractQueue<E>
            implements BlockingQueue<E>, java.io.Serializable {
        private static final long serialVersionUID = -6903933977591709194L;

        /*
         * A variant of the "two lock queue" algorithm.  The putLock gates
         * entry to put (and offer), and has an associated condition for
         * waiting puts.  Similarly for the takeLock.  The "count" field
         * that they both rely on is maintained as an atomic to avoid
         * needing to get both locks in most cases. Also, to minimize need
         * for puts to get takeLock and vice-versa, cascading notifies are
         * used. When a put notices that it has enabled at least one take,
         * it signals taker. That taker in turn signals others if more
         * items have been entered since the signal. And symmetrically for
         * takes signalling puts. Operations such as remove(Object) and
         * iterators acquire both locks.
         * “双锁队列”算法的变体。putLock控制着put和offer,并且与之关联一个condision来等待插入。
         * takeLock和上面差不多。它们都依赖的“count”字段作为原子进行维护，以避免在大多数情况下需要同时获得两个锁。
         * 此外，为了最小化获取takeLock的put需求，还使用了级联通知。
         * 当put注意到它至少启用了一次take时，它向taker发出信号。
         * 如果在信号发出后输入了更多的items，那么该taker就会依次向其他人发出信号。
         * puts和这个对称。remove(Object)和iterator操作需要两种锁。
         *
         * Visibility between writers and readers is provided as follows:
         * 作者和读者之间的可见性如下:
         *
         * Whenever an element is enqueued, the putLock is acquired and
         * count updated.  A subsequent reader guarantees visibility to the
         * enqueued Node by either acquiring the putLock (via fullyLock)
         * or by acquiring the takeLock, and then reading n = count.get();
         * this gives visibility to the first n items.
         * 每当一个元素进入队列时，都会获取putLock并更新计数。
         * 随后的读取器通过获取putLock(通过fullyLock)或获取takeLock，
         * 然后读取n = count.get()，从而保证队列节点的可见性;这使得前n项可见。
         *
         * To implement weakly consistent iterators, it appears we need to
         * keep all Nodes GC-reachable from a predecessor dequeued Node.
         * 为了实现弱一致的迭代器，我们似乎需要保持所有的节点GC-reachable 可从一个已退出队列的节点访问。
         * That would cause two problems:
         * 这将导致两个问题:
         * - allow a rogue Iterator to cause unbounded memory retention
         * 允许异常迭代器导致无限制的内存保留
         * - cause cross-generational linking of old Nodes to new Nodes if
         *   a Node was tenured while live, which generational GCs have a
         *   hard time dealing with, causing repeated major collections.
         * 如果一个节点是终身存在的，则会导致旧节点与新节点的跨代连接，而跨代GCs很难处理这一问题，从而导致重复的主要集合。
         * However, only non-deleted Nodes need to be reachable from
         * dequeued Nodes, and reachability does not necessarily have to
         * be of the kind understood by the GC.  We use the trick of
         * linking a Node that has just been dequeued to itself.  Such a
         * self-link implicitly means to advance to head.next.
         * 然而，只有未删除的节点才需要从排队删除的节点中访问，并且可达性不必是GC所理解的那种。
         * 我们使用的技巧是将刚刚从队列中删除的节点链接到它自己。
         * 这样的自链接隐含的意思是前进到head.next。
         */

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

    //     /**
    //      * Tells whether both locks are held by current thread.
    //      */
    //     boolean isFullyLocked() {
    //         return (putLock.isHeldByCurrentThread() &&
    //                 takeLock.isHeldByCurrentThread());
    //     }

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

        // this doc comment is overridden to remove the reference to collections
        // greater in size than Integer.MAX_VALUE

        /**
         * Returns the number of elements in this queue.
         * 返回此队列中的元素数量。
         *
         * @return the number of elements in this queue
         */
        public int size() {
            return count.get();
        }

        // this doc comment is a modified copy of the inherited doc comment,
        // without the reference to unlimited queues.

        /**
         * Returns the number of additional elements that this queue can ideally
         * (in the absence of memory or resource constraints) accept without
         * blocking. This is always equal to the initial capacity of this queue
         * less the current {@code size} of this queue.
         * <p>
         * 返回此队列在理想情况下(在没有内存或资源约束的情况下)可以不阻塞地接受的附加元素的数量
         * 这总是等于这个队列的初始容量减去这个队列的当前{@code size}。
         *
         * <p>Note that you <em>cannot</em> always tell if an attempt to insert
         * an element will succeed by inspecting {@code remainingCapacity}
         * because it may be the case that another thread is about to
         * insert or remove an element.
         * <p>
         * 注意，您不能总是通过检查{@code残留量}来判断插入元素的尝试是否会成功
         * ，因为可能是另一个线程将要插入或删除一个元素。
         */
        public int remainingCapacity() {
            return capacity - count.get();
        }

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
         * Returns {@code true} if this queue contains the specified element.
         * More formally, returns {@code true} if and only if this queue contains
         * at least one element {@code e} such that {@code o.equals(e)}.
         * 如果此队列包含指定的元素，则返回{@code true}。
         * @param o object to be checked for containment in this queue
         * @return {@code true} if this queue contains the specified element
         */
        public boolean contains(Object o) {
            if (o == null) return false;
            fullyLock();
            try {
                for (Node<E> p = head.next; p != null; p = p.next)
                    if (o.equals(p.item))
                        return true;
                return false;
            } finally {
                fullyUnlock();
            }
        }

        /**
         * Returns an array containing all of the elements in this queue, in
         * proper sequence.
         * 返回包含此队列中所有元素的数组，按适当的顺序。
         * <p>The returned array will be "safe" in that no references to it are
         * maintained by this queue.  (In other words, this method must allocate
         * a new array).  The caller is thus free to modify the returned array.
         * 返回的数组将是“安全的”，因为这个队列不维护对它的引用。(换句话说，这个方法必须分配一个新的数组)。
         * <p>This method acts as bridge between array-based and collection-based
         * APIs.
         *
         * @return an array containing all of the elements in this queue
         */
        public Object[] toArray() {
            //全部锁定
            fullyLock();
            try {
                int size = count.get();
                //新建数组
                Object[] a = new Object[size];
                int k = 0;
                //迭代链表组装数组
                for (Node<E> p = head.next; p != null; p = p.next)
                    a[k++] = p.item;
                return a;
            } finally {
                //解锁
                fullyUnlock();
            }
        }

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

        /**
         * 将队列中值，全部移除，并发设置到给定的集合中
         * @throws UnsupportedOperationException {@inheritDoc}
         * @throws ClassCastException            {@inheritDoc}
         * @throws NullPointerException          {@inheritDoc}
         * @throws IllegalArgumentException      {@inheritDoc}
         */
        public int drainTo(Collection<? super E> c) {
            return drainTo(c, Integer.MAX_VALUE);
        }

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

    }
```