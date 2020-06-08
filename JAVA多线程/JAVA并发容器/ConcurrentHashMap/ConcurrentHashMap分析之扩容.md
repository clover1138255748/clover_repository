### 扩容触发条件：

1. treeifyBin方法中将链表转换成红黑树时判断table数组的长度是否小于阈值（64），如果小于就进行扩容而不是树化。
2. putAll方法中，会根据size大小扩容。
3. putVal在计数的时候，判断当前Entry数量是否超过阈值，如果超过就进行扩容

### 与扩容相关的常量、变量与函数

```
        /**
         * Minimum number of rebinnings per transfer step. Ranges are
         * subdivided to allow multiple resizer threads.  This value
         * serves as a lower bound to avoid resizers encountering
         * excessive memory contention.  The value should be at least
         * DEFAULT_CAPACITY.
         * 每个转移步骤的最少复归数。范围被细分以允许多个调整大小的线程。
         * 此值用作下限，以避免调整大小器遇到过多的内存争用。
         * 该值至少应该是DEFAULT_CAPACITY。
         */
        private static final int MIN_TRANSFER_STRIDE = 16;

        /**
         * The number of bits used for generation stamp in sizeCtl.
         * Must be at least 6 for 32bit arrays.
         * 用于生成戳记的位的数目，单位为sizeCtl。
         * 32位数组必须至少为6。
         */
        private static int RESIZE_STAMP_BITS = 16;

        /**
         * The maximum number of threads that can help resize.
         * Must fit in 32 - RESIZE_STAMP_BITS bits.
         * 可以帮助调整大小的最大线程数。
         * 必须符合32-RESIZE_STAMP_BITS bits
         */
        private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

        /**
         * The bit shift for recording size stamp in sizeCtl.
         * 用sizeCtl记录尺寸戳的位偏移。
         */
        private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

        /*
         * Encodings for Node hash fields. See above for explanation.
         * 节点哈希字段的编码。见解释。
         */
        static final int MOVED = -1; // hash for forwarding nodes 转发节点的哈希
        static final int TREEBIN = -2; // hash for roots of trees 树根的哈希
        static final int RESERVED = -3; // hash for transient reservations 临时保留的哈希
        static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash 普通节点哈希的可用位

        /**
         * Number of CPUS, to place bounds on some sizings
         * cpu的数量，以限制某些大小
         */
        static final int NCPU = Runtime.getRuntime().availableProcessors();
           /**
         * The array of bins. Lazily initialized upon first insertion.
         * Size is always a power of two. Accessed directly by iterators.
         * 桶的数组。在第一次插入时惰性初始化。大小总是2的幂。由迭代器直接访问。
         */
        transient volatile Node<K, V>[] table;

        /**
         * The next table to use; non-null only while resizing.
         * 要使用的下一个表;仅在调整大小时非空。
         */
        private transient volatile Node<K, V>[] nextTable;
          /**
         * Table initialization and resizing control.  When negative, the
         * table is being initialized or resized: -1 for initialization,
         * else -(1 + the number of active resizing threads).  Otherwise,
         * when table is null, holds the initial table size to use upon
         * creation, or 0 for default. After initialization, holds the
         * next element count value upon which to resize the table.
         * 表初始化和调整大小控制。当为负数时，表被初始化或调整大小:初始化为-1（1 +活动调整大小的线程的数量）
         * 否则，当表为空时，保留创建时使用的初始表大小，默认情况下为0。
         * 初始化之后，保存下一个元素count值，根据该值调整表的大小。
         * <p>
         * -1 :代表table正在初始化,其他线程应该交出CPU时间片
         * <p>
         * -N: 表示正有N-1个线程执行扩容操作（高 16 位是 length 生成的标识符，低 16 位是扩容的线程数）
         * <p>
         * 大于 0: 如果table已经初始化,代表table容量,默认为table大小的0.75,如果还未初始化,代表需要初始化的大小
         */
        private transient volatile int sizeCtl;

        /**
         * The next table index (plus one) to split while resizing.
         * 调整大小时要分割的下一个表索引(加上一个)。
         */
        private transient volatile int transferIndex;

            /**
         * Returns the stamp bits for resizing a table of size n.
         * Must be negative when shifted left by RESIZE_STAMP_SHIFT.
         * 返回用于调整大小为n的表的戳记位，
         * 当通过RESIZE_STAMP_SHIFT左移时，必须为负。
         * 该函数返回一个用于数据校验的标志位，意思是对长度为n的table进行扩容。
         * 它将n的前导零（最高有效位之前的零的数量）和1 << 15做或运算，这时低16位的最高位为1，其他都为n的前导零。
         */
        static final int resizeStamp(int n) {
            //numberOfLeadingZeros方法的作用是返回无符号整型i的最高非零位前面的0的个数，包括符号位在内；
            //如果i为负数，这个方法将会返回0，符号位为1.
            return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));
        }
```

### 协调多个线程如何调用transfer方法进行hash桶的迁移

#### tryPresize

协调多个线程如何调用transfer方法进行hash桶的迁移，tryPresize在putAll以及treeifyBin中被调用。

```
      /**
         * 协调多个线程如何调用transfer方法进行hash桶的迁移
         *
         * @param size the size
         */
        private final void tryPresize(int size) {
            //计算扩容的目标size
            // 给定的容量若>=MAXIMUM_CAPACITY的一半，直接扩容到允许的最大值，否则调用函数扩容
            int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
                    tableSizeFor(size + (size >>> 1) + 1);
            int sc;
            //没有正在初始化或扩容，或者说表还没有被初始化
            while ((sc = sizeCtl) >= 0) {
                Node<K, V>[] tab = table;
                int n;
                //tab没有初始化
                if (tab == null || (n = tab.length) == 0) {
                    // 扩容阀值取较大者
                    n = (sc > c) ? sc : c;
                    // 期间没有其他线程对表操作，则CAS将SIZECTL状态置为-1，表示正在进行初始化
                    //初始化之前，CAS设置sizeCtl=-1
                    if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                        try {
                            if (table == tab) {
                                @SuppressWarnings("unchecked")
                                Node<K, V>[] nt = (Node<K, V>[]) new Node<?, ?>[n];
                                table = nt;
                                //sc=0.75n,相当于扩容阈值,无符号右移2位，此即0.75*n
                                sc = n - (n >>> 2);
                            }
                        } finally {
                            // 此时并没有通过CAS赋值，因为其他想要执行初始化的线程，
                            // 发现sizeCtl=-1，就直接返回，从而确保任何情况，
                            // 只会有一个线程执行初始化操作。
                            sizeCtl = sc;
                        }
                    }
                }
                // 若欲扩容值不大于原阀值，或现有容量>=最值，什么都不用做了
                //目标扩容size小于扩容阈值，或者容量超过最大限制时，不需要扩容
                else if (c <= sc || n >= MAXIMUM_CAPACITY)
                    break;
                    //扩容
                else if (tab == table) {
                    int rs = resizeStamp(n);
                    // sc<0表示，已经有其他线程正在扩容
                    if (sc < 0) {
                        //RESIZE_STAMP_SHIFT=16,MAX_RESIZERS=2^15-1
                        Node<K, V>[] nt;
                        // 1. (sc >>> RESIZE_STAMP_SHIFT) != rs ：扩容线程数 > MAX_RESIZERS-1
                        // 2. sc == rs + 1 和 sc == rs + MAX_RESIZERS
                        // 3. (nt = nextTable) == null ：表示nextTable正在初始化
                        // transferIndex <= 0 ：表示所有hash桶均分配出去
                        if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                                sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                                transferIndex <= 0)
                            //如果不需要帮其扩容，直接返回
                            break;
                        //CAS设置sizeCtl=sizeCtl+1
                        if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                            //帮其扩容
                            transfer(tab, nt);
                    }
                    // 第一个执行扩容操作的线程，将sizeCtl设置为：
                    // (resizeStamp(n) << RESIZE_STAMP_SHIFT) + 2)
                    else if (U.compareAndSwapInt(this, SIZECTL, sc,
                            (rs << RESIZE_STAMP_SHIFT) + 2))
                        transfer(tab, null);
                }
            }
        }
```

#### addCount

这个方法一共做了两件事：更新baseCount的值，检测是否进行扩容。

```
     /**
         * Adds to count, and if table is too small and not already
         * resizing, initiates transfer. If already resizing, helps
         * perform transfer if work is available.  Rechecks occupancy
         * after a transfer to see if another resize is already needed
         * because resizings are lagging additions.
         * <p>
         * 添加到count，如果表太小且尚未调整大小，则启动传输。
         * 如果已经调整大小，则在工作可用时帮助执行传输。
         * 在转移后重新检查占用情况，看看是否已经需要另一个大小调整，因为大小调整是滞后的添加。
         *
         * @param x     the count to add 添加的数量
         * @param check if <0, don't check resize, if <= 1 only check if uncontended 如果<0，不检查调整大小，如果<= 1，只检查是否无竞争
         */
        private final void addCount(long x, int check) {
            CounterCell[] as;
            long b, s;
            ......
            //检查，判断是否需要扩容
            if (check >= 0) {
                Node<K, V>[] tab, nt;
                int n, sc;
                //当sizeCtl小于总数量 且 table不为空 且 tabled的长度不为最大容量，进入循环
                while (s >= (long) (sc = sizeCtl) && (tab = table) != null &&
                        (n = tab.length) < MAXIMUM_CAPACITY) {
                    //计算扩容标志
                    int rs = resizeStamp(n);
                    //表未初始化
                    if (sc < 0) {
                        //如果sc右移16位不等于rs 或者 sc等于rs+1 或者 sc等于rs+最大扩容线程数 或者 nextTable未空 或者  transferIndex<=0
                        if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                                sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                                transferIndex <= 0)
                            break;
                        //如果SIZECTL与sc相同，则将sc的值+1 设置成功的话，
                        if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                            //转移数据
                            transfer(tab, nt);

                        //如果设置SIZECTL为(rs << RESIZE_STAMP_SHIFT) + 2成功
                    } else if (U.compareAndSwapInt(this, SIZECTL, sc,
                            (rs << RESIZE_STAMP_SHIFT) + 2))
                        //转移数据
                        transfer(tab, null);
                    //计数
                    s = sumCount();
                }
            }
        }
```

#### helpTransfer

协助扩容方法,多线程下，当前线程检测到其他线程正进行扩容操作，则协助其一起扩容。
（只有这种情况会被调用）从某种程度上说，其“优先级”很高，只要检测到扩容，就会放下其他工作，先扩容。
调用之前，nextTable一定已存在。

```
     /**
         * Helps transfer if a resize is in progress.
         * 如果还在进行扩容操作就先进行扩容
         */
        final Node<K, V>[] helpTransfer(Node<K, V>[] tab, Node<K, V> f) {
            Node<K, V>[] nextTab;
            int sc;
            //如果表非空 且 当前节点是ForwardingNode节点 且 nextTable不为空
            if (tab != null && (f instanceof ForwardingNode) &&
                    (nextTab = ((ForwardingNode<K, V>) f).nextTable) != null) {
                //返回的是对 tab.length 的一个数据校验标识，占 16 位。而 RESIZE_STAMP_SHIFT 的值为 16，那么位运算后，整个表达式必然在右边空出 16 个零。
                //也正如我们所说的，sizeCtl 的高 16 位为数据校验标识，低 16 为表示正在进行扩容的线程数量。
                int rs = resizeStamp(tab.length);
                // 如果 nextTab 没有被并发修改 且 tab 也没有被并发修改
                // 且 sizeCtl  < 0 （说明还在扩容）
                while (nextTab == nextTable && table == tab &&
                        (sc = sizeCtl) < 0) {
                    // 如果 sizeCtl 无符号右移  16 不等于 rs （ sc前 16 位如果不等于标识符，则标识符变化了）
                    // 或者 sizeCtl == rs + 1  （扩容结束了，不再有线程进行扩容）（默认第一个线程设置 sc ==rs 左移 16 位 + 2，当第一个线程结束扩容了，就会将 sc 减一。这个时候，sc 就等于 rs + 1）
                    // 或者 sizeCtl == rs + 65535  （如果达到最大帮助线程的数量，即 65535）
                    // 或者 转移下标正在调整 （扩容结束）
                    // 结束循环，返回 table
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                            sc == rs + MAX_RESIZERS || transferIndex <= 0)
                        break;
                    // 如果以上都不是, 将 sizeCtl + 1, （表示增加了一个线程帮助其扩容）
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                        // 进行转移扩容
                        transfer(tab, nextTab);
                        //循环结束
                        break;
                    }
                }
                return nextTab;
            }
            return table;
        }
```

### 扩容操作

> 扩容操作的核心在于数据的转移，多线程操作时，把整个table数组当做多个线程之间共享的任务队列，然后只需维护一个指针，
> 当有一个线程开始进行数据转移，就会先移动指针，表示指针划过的这片bucket区域由该线程负责。
> 这个指针被声明为一个volatile整型变量，它的初始位置位于table的尾部，即它等于table.length，很明显这个任务队列是逆向遍历的。
>
> 一个已经迁移完毕的bucket会被替换成ForwardingNode节点，用来标记此bucket已经被其他线程迁移完毕了。
> ForwardingNode是一个特殊节点，可以通过hash域的虚拟值来识别它，它同样重写了find()函数，用来在新数组中查找目标。
>
> 数据迁移的操作位于transfer()函数，多个线程之间依靠sizeCtl与transferIndex指针来协同工作，每个线程都有自己负责的区域，
> 一个完成迁移的bucket会被设置为ForwardingNode，其他线程遇见这个特殊节点就跳过该bucket，处理下一个bucket。

transfer源码：

```
      /**
         * Moves and/or copies the nodes in each bin to new table. See
         * above for explanation.
         * 将每个bin中的节点移动和/或复制到新表中。
         */
        private final void transfer(Node<K, V>[] tab, Node<K, V>[] nextTab) {
            int n = tab.length, stride;
            // 将 length / 8 然后除以 CPU核心数。如果得到的结果小于 16，那么就使用 16。
            // 这里的目的是让每个 CPU 处理的桶一样多，避免出现转移任务不均匀的现象，如果桶较少的话，默认一个 CPU（一个线程）处理 16 个桶
            if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
                stride = MIN_TRANSFER_STRIDE; // subdivide range
            // 初始化nextTab，容量为旧数组的一倍
            if (nextTab == null) {            // initiating
                try {
                    @SuppressWarnings("unchecked")
                    // 扩容  2 倍
                            Node<K, V>[] nt = (Node<K, V>[]) new Node<?, ?>[n << 1];
                    // 赋值
                    nextTab = nt;
                } catch (Throwable ex) {      // try to cope with OOME
                    // 扩容失败， sizeCtl 使用 int 最大值。
                    sizeCtl = Integer.MAX_VALUE;
                    return;
                }
                //赋值
                nextTable = nextTab;
                // 更新转移下标，就是 老的 tab 的 length
                transferIndex = n;
            }
            // 新 tab 的 length
            int nextn = nextTab.length;
            // 创建一个 ForwardingNode 节点，用于占位。
            // 当别的线程发现这个槽位中是 ForwardingNode 类型的节点，则跳过这个节点。
            ForwardingNode<K, V> fwd = new ForwardingNode<K, V>(nextTab);
            // 首次推进为 true，如果等于 true，说明需要再次推进一个下标（i--），
            // 反之，如果是 false，那么就不能推进下标，需要将当前的下标处理完毕才能继续推进
            boolean advance = true;
            // 完成状态，如果是 true，就结束此方法。
            boolean finishing = false; // to ensure sweep before committing nextTab
            // 死循环,i 表示下标，bound 表示当前线程可以处理的当前桶区间最小下标
            for (int i = 0, bound = 0; ; ) {
                Node<K, V> f;
                int fh;
                // 如果当前线程可以向后推进；这个循环就是控制 i 递减。
                // 同时，每个线程都会进入这里取得自己需要转移的桶的区间
                while (advance) {
                    int nextIndex, nextBound;
                    // 对 i 减一，判断是否大于等于 bound （正常情况下，如果大于 bound 不成立，
                    // 说明该线程上次领取的任务已经完成了。那么，需要在下面继续领取任务）
                    // 如果对 i 减一大于等于 bound（还需要继续做任务），
                    // 或者完成了，修改推进状态为 false，不能推进了。
                    // 任务成功后修改推进状态为 true。
                    // 通常，第一次进入循环，i-- 这个判断会无法通过，
                    // 从而走下面的 nextIndex 赋值操作（获取最新的转移下标）。
                    // 其余情况都是：如果可以推进，将 i 减一，然后修改成不可推进。
                    // 如果 i 对应的桶处理成功了，改成可以推进。
                    if (--i >= bound || finishing)
                        // 这里设置 false，是为了防止在没有成功处理一个桶的情况下却进行了推进
                        advance = false;
                        // 这里的目的是：
                        // 1. 当一个线程进入时，会选取最新的转移下标。
                        // 2. 当一个线程处理完自己的区间时，如果还有剩余区间的没有别的线程处理。再次获取区间。
                    else if ((nextIndex = transferIndex) <= 0) {
                        // 如果小于等于0，说明没有区间了 ，i 改成 -1，推进状态变成 false，
                        // 不再推进，表示，扩容结束了，当前线程可以退出了
                        // 这个 -1 会在下面的 if 块里判断，从而进入完成状态判断
                        i = -1;
                        // 这里设置 false，是为了防止在没有成功处理一个桶的情况下却进行了推进
                        advance = false;
                        // CAS 修改 transferIndex，即 length - 区间值，留下剩余的区间值供后面的线程使用
                    } else if (U.compareAndSwapInt
                            (this, TRANSFERINDEX, nextIndex,
                                    nextBound = (nextIndex > stride ?
                                            nextIndex - stride : 0))) {
                        // 这个值就是当前线程可以处理的最小当前区间最小下标
                        bound = nextBound;
                        // 初次对i 赋值，这个就是当前线程可以处理的当前区间的最大下标
                        i = nextIndex - 1;
                        // 这里设置 false，是为了防止在没有成功处理一个桶的情况下却进行了推进，这样对导致漏掉某个桶。
                        // 下面的 if (tabAt(tab, i) == f) 判断会出现这样的情况。
                        advance = false;
                    }
                }
                // 如果 i 小于0 （不在 tab 下标内，按照上面的判断，领取最后一段区间的线程扩容结束）
                //  如果 i >= tab.length
                //  如果 i + tab.length >= nextTable.length
                if (i < 0 || i >= n || i + n >= nextn) {
                    int sc;
                    // 如果完成了扩容
                    if (finishing) {
                        // 删除成员变量
                        nextTable = null;
                        // 更新 table
                        table = nextTab;
                        // 更新阈值
                        sizeCtl = (n << 1) - (n >>> 1);
                        return;
                    }
                    // 如果没完成
                    // 尝试将 sc -1. 表示这个线程结束帮助扩容了，将 sc 的低 16 位减一。
                    if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                        // 如果 sc - 2 不等于标识符左移 16 位。
                        // 如果他们相等了，说明没有线程在帮助他们扩容了。也就是说，扩容结束了。
                        if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                            // 不相等，说明没结束，当前线程结束方法。
                            return;
                        // 如果相等，扩容结束了，更新 finising 变量
                        finishing = advance = true;
                        // 再次循环检查一下整张表
                        i = n; // recheck before commit
                    }
                    // 获取老 tab i 下标位置的变量，如果是 null，就使用 fwd 占位。
                } else if ((f = tabAt(tab, i)) == null)
                    // 如果成功写入 fwd 占位，再次推进一个下标
                    advance = casTabAt(tab, i, null, fwd);
                    // 如果不是 null 且 hash 值是 MOVED。
                else if ((fh = f.hash) == MOVED)
                    // 说明别的线程已经处理过了，再次推进一个下标
                    advance = true; // already processed
                    // 到这里，说明这个位置有实际值了，且不是占位符。对这个节点上锁。为什么上锁，防止 putVal 的时候向链表插入数据
                else {
                    synchronized (f) {
                        // 判断 i 下标处的桶节点是否和 f 相同
                        if (tabAt(tab, i) == f) {
                            Node<K, V> ln, hn;
                            // 如果 f 的 hash 值大于 0 。TreeBin 的 hash 是 -2
                            if (fh >= 0) {
                                // 对老长度进行与运算（第一个操作数的的第n位于第二个操作数的第n位如果都是1，那么结果的第n为也为1，否则为0）
                                // 由于 Map 的长度都是 2 的次方（000001000 这类的数字），那么取于 length 只有 2 种结果，一种是 0，一种是1
                                //  如果是结果是0 ，Doug Lea 将其放在低位，反之放在高位，目的是将链表重新 hash，放到对应的位置上，让新的取于算法能够击中他。
                                int runBit = fh & n;
                                // 尾节点，且和头节点的 hash 值取于不相等
                                Node<K, V> lastRun = f;
                                // 遍历这个桶
                                for (Node<K, V> p = f.next; p != null; p = p.next) {
                                    // 取于桶中每个节点的 hash 值
                                    int b = p.hash & n;
                                    // 如果节点的 hash 值和首节点的 hash 值取于结果不同
                                    if (b != runBit) {
                                        // 更新 runBit，用于下面判断 lastRun 该赋值给 ln 还是 hn。
                                        runBit = b;
                                        // 这个 lastRun 保证后面的节点与自己的取于值相同，避免后面没有必要的循环
                                        lastRun = p;
                                    }
                                }
                                // 如果最后更新的 runBit 是 0 ，设置低位节点
                                if (runBit == 0) {
                                    ln = lastRun;
                                    hn = null;
                                } else {
                                    // 如果最后更新的 runBit 是 1， 设置高位节点
                                    hn = lastRun;
                                    ln = null;
                                }
                                // 再次循环，生成两个链表，lastRun 作为停止条件，这样就是避免无谓的循环（lastRun 后面都是相同的取于结果）
                                for (Node<K, V> p = f; p != lastRun; p = p.next) {
                                    int ph = p.hash;
                                    K pk = p.key;
                                    V pv = p.val;
                                    // 如果与运算结果是 0，那么就还在低位
                                    // 如果是0 ，那么创建低位节点
                                    if ((ph & n) == 0)
                                        ln = new Node<K, V>(ph, pk, pv, ln);
                                    else
                                        // 1 则创建高位
                                        hn = new Node<K, V>(ph, pk, pv, hn);
                                }
                                // 其实这里类似 hashMap
                                // 设置低位链表放在新链表的 i
                                setTabAt(nextTab, i, ln);
                                // 设置高位链表，在原有长度上加 n
                                setTabAt(nextTab, i + n, hn);
                                // 将旧的链表设置成占位符
                                setTabAt(tab, i, fwd);
                                // 继续向后推进
                                advance = true;
                                // 如果是红黑树
                            } else if (f instanceof TreeBin) {
                                TreeBin<K, V> t = (TreeBin<K, V>) f;
                                TreeNode<K, V> lo = null, loTail = null;
                                TreeNode<K, V> hi = null, hiTail = null;
                                int lc = 0, hc = 0;
                                // 遍历
                                for (Node<K, V> e = t.first; e != null; e = e.next) {
                                    int h = e.hash;
                                    TreeNode<K, V> p = new TreeNode<K, V>
                                            (h, e.key, e.val, null, null);
                                    // 和链表相同的判断，与运算 == 0 的放在低位
                                    if ((h & n) == 0) {
                                        if ((p.prev = loTail) == null)
                                            lo = p;
                                        else
                                            loTail.next = p;
                                        loTail = p;
                                        ++lc;
                                        // 不是 0 的放在高位
                                    } else {
                                        if ((p.prev = hiTail) == null)
                                            hi = p;
                                        else
                                            hiTail.next = p;
                                        hiTail = p;
                                        ++hc;
                                    }
                                }
                                // 如果树的节点数小于等于 6，那么转成链表，反之，创建一个新的树
                                ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                        (hc != 0) ? new TreeBin<K, V>(lo) : t;
                                hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                        (lc != 0) ? new TreeBin<K, V>(hi) : t;
                                // 低位树
                                setTabAt(nextTab, i, ln);
                                // 高位树
                                setTabAt(nextTab, i + n, hn);
                                // 旧的设置成占位符
                                setTabAt(tab, i, fwd);
                                // 继续向后推进
                                advance = true;
                            }
                        }
                    }
                }
            }
        }
```

从源码可知，transfer方法分为4个步骤：

1. 对后续要使用的变量做初始化，如果新建临时表，容量为老表的2倍。
2. 为当前线程分配任务和控制当前线程的任务进度，描述了如何与其他线程协同工作。
3. 最后是具体的迁移过程（对当前指向的bucket），这部分的逻辑与HashMap类似，
   拿旧数组的容量当做一个掩码，然后与节点的hash进行与操作，可以得出该节点的新增有效位，
   如果新增有效位为0就放入一个链表A，如果为1就放入另一个链表B，
   链表A在新数组中的位置不变（跟在旧数组的索引一致），链表B在新数组中的位置为原索引加上旧数组容量。

### 图形分析

1、ConcurrentHashMap支持并发扩容，实现方式是，将表拆分，让每个线程处理自己的区间,如图：

[![2019103010012\_1.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/2019103010012_2.png)](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/javaCore/javaConcurrent/2019103010012_1.png)

1.png

2、每个线程在处理自己桶中的数据
扩容前的状态：

[![2019103010012\_2.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/2019103010012_2.png)](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/javaCore/javaConcurrent/2019103010012_2.png)

2.png

当对 4 号桶或者 10 号桶进行转移的时候，会将链表拆成两份，规则是根据节点的 hash 值取于 length，如果结果是 0，放在低位，否则放在高位。
因此，10 号桶的数据，黑色节点会放在新表的 10 号位置，白色节点会放在新桶的 26 号位置。
下图是循环处理桶中数据的逻辑：

[![2019103010012\_3.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/2019103010012_2.png)](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/javaCore/javaConcurrent/2019103010012_3.png)

3.png

处理完之后，新桶的数据是这样的：

[![2019103010012\_4.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/2019103010012_2.png)](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/javaCore/javaConcurrent/2019103010012_4.png)

4.png