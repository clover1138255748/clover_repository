基于JDK8

```
        /* ---------------- Special Nodes -------------- */

        /**
         * A node inserted at head of bins during transfer operations.
         * 在迭代操作期间插入到桶头的节点。
         */
        static final class ForwardingNode<K, V> extends Node<K, V> {
            final Node<K, V>[] nextTable;

            ForwardingNode(Node<K, V>[] tab) {
                super(MOVED, null, null, null);
                this.nextTable = tab;
            }

            Node<K, V> find(int h, Object k) {
                // loop to avoid arbitrarily deep recursion on forwarding nodes
                outer:
                for (Node<K, V>[] tab = nextTable; ; ) {
                    Node<K, V> e;
                    int n;
                    if (k == null || tab == null || (n = tab.length) == 0 ||
                            (e = tabAt(tab, (n - 1) & h)) == null)
                        return null;
                    for (; ; ) {
                        int eh;
                        K ek;
                        if ((eh = e.hash) == h &&
                                ((ek = e.key) == k || (ek != null && k.equals(ek))))
                            return e;
                        if (eh < 0) {
                            if (e instanceof ForwardingNode) {
                                tab = ((ForwardingNode<K, V>) e).nextTable;
                                continue outer;
                            } else
                                return e.find(h, k);
                        }
                        if ((e = e.next) == null)
                            return null;
                    }
                }
            }
        }

        /* ---------------- Table Initialization and Resizing -------------- */

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

        /**
         * Initializes table, using the size recorded in sizeCtl.
         * 使用sizeCtl中记录的大小初始化表。
         */
        private final Node<K, V>[] initTable() {
            Node<K, V>[] tab;
            int sc;
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
                            Node<K, V>[] nt = (Node<K, V>[]) new Node<?, ?>[n];
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
            //如果当前计数表格中不为空 或者 设置BASECOUNT=baseCount+x 失败(存在其他线程再操作这个baseCount),
            // 即，尝试使用CAS更新baseCount失败就转用CounterCells进行更新
            if ((as = counterCells) != null ||
                    !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
                CounterCell a;
                long v;
                int m;
                boolean uncontended = true;
                //如果表格为空 或者 表格中探测值为空 或者 设置CELLVALUE为 探测值的a.value+x 失败（很倒霉，这个值也设置失败了）
                //即，尝试使用CAS更新当前线程的counterCell失败就转用fullAddCount进行更新
                if (as == null || (m = as.length - 1) < 0 ||
                        (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                        !(uncontended =
                                U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                    //全部尝试一遍 没有人争的
                    fullAddCount(x, uncontended);
                    return;
                }
                //if <= 1 only check if uncontended
                if (check <= 1)
                    return;
                //计数
                s = sumCount();
            }
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

        /* ---------------- Counter support -------------- */

        /**
         * A padded cell for distributing counts.  Adapted from LongAdder
         * and Striped64.  See their internal docs for explanation.
         * 用于分配计数的填充单元格。改编自LongAdder and Striped64
         * 请参阅他们的内部文档进行解释。
         * <p>
         * 注解@sun.misc.Contended用于解决伪共享问题。
         * 所谓伪共享，即是在同一缓存行（CPU缓存的基本单位）中存储了多个变量，
         * 当其中一个变量被修改时，就会影响到同一缓存行内的其他变量，
         * 导致它们也要跟着被标记为失效，其他变量的缓存命中率将会受到影响。
         * 解决伪共享问题的方法一般是对该变量填充一些无意义的占位数据，
         * 从而使它独享一个缓存行。
         */
        @sun.misc.Contended
        static final class CounterCell {
            //volatile修饰的值，内存可见
            volatile long value;

            CounterCell(long x) {
                value = x;
            }
        }

        /**
         * 计算ConcurrentHashMap中元素的个数
         *
         * @return the long
         */
        final long sumCount() {
            CounterCell[] as = counterCells;
            CounterCell a;
            //基本计数值
            long sum = baseCount;
            //计数单元格
            if (as != null) {
                //迭代技术单元格，计算总数量
                for (int i = 0; i < as.length; ++i) {
                    if ((a = as[i]) != null)
                        //累加
                        sum += a.value;
                }
            }
            return sum;
        }

        // See LongAdder version for explanation 有关说明，请参阅LongAdder版本
        private final void fullAddCount(long x, boolean wasUncontended) {
            int h;
            // 当前线程的probe等于0，证明该线程的ThreadLocalRandom还未被初始化
            if ((h = ThreadLocalRandom.getProbe()) == 0) {
                // 初始化ThreadLocalRandom，当前线程会被设置一个probe
                ThreadLocalRandom.localInit();      // force initialization 强行初始化
                // probe用于在CounterCell数组中寻址
                h = ThreadLocalRandom.getProbe();
                // 未竞争标志
                wasUncontended = true;
            }
            // 冲突标志
            boolean collide = false;                // True if last slot nonempty
            //死循环
            for (; ; ) {
                CounterCell[] as;
                CounterCell a;
                int n;
                long v;
                //如果当前表格中存在数据，即已经初始化过
                if ((as = counterCells) != null && (n = as.length) > 0) {
                    // 如果寻址到的Cell为空，那么创建一个新的Cell
                    if ((a = as[(n - 1) & h]) == null) {
                        // cellsBusy是一个只有0和1两个状态的volatile整数
                        // 它被当做一个自旋锁，0代表无锁，1代表加锁
                        if (cellsBusy == 0) {            // Try to attach new Cell 尝试连接新的单元格
                            // 将传入的x作为初始值创建一个新的CounterCell
                            CounterCell r = new CounterCell(x); // Optimistic create
                            // 通过CAS尝试对自旋锁加锁
                            if (cellsBusy == 0 &&
                                    U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                                // 加锁成功，声明Cell是否创建成功的标志
                                boolean created = false;
                                //再核对下锁
                                try {               // Recheck under lock
                                    CounterCell[] rs;
                                    int m, j;
                                    // 再次检查CounterCell数组是否不为空
                                    // 并且寻址到的Cell为空
                                    if ((rs = counterCells) != null &&
                                            (m = rs.length) > 0 &&
                                            rs[j = (m - 1) & h] == null) {
                                        // 将之前创建的新Cell放入数组
                                        rs[j] = r;
                                        created = true;
                                    }
                                } finally {
                                    // 释放锁
                                    cellsBusy = 0;
                                }
                                // 如果已经创建成功，中断循环
                                // 因为新Cell的初始值就是传入的增量，所以计数已经完毕了
                                if (created)
                                    break;
                                // 如果未成功
                                // 代表as[(n - 1) & h]这个位置的Cell已经被其他线程设置
                                // 那么就从循环头重新开始
                                continue;           // Slot is now non-empty
                            }
                        }
                        collide = false;

                        // as[(n - 1) & h]非空
                        // 在addCount()函数中通过CAS更新当前线程的Cell进行计数失败
                        // 会传入wasUncontended = false，代表已经有其他线程进行竞争
                    } else if (!wasUncontended)       // CAS already known to fail
                        // 设置未竞争标志，之后会重新计算probe，然后重新执行循环
                        wasUncontended = true;      // Continue after rehash
                        // 尝试进行计数，如果成功，那么就退出循环
                    else if (U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))
                        break;
                        // 尝试更新失败，检查counterCell数组是否已经扩容
                        // 或者容量达到最大值（CPU的数量）
                    else if (counterCells != as || n >= NCPU)
                        // 设置冲突标志，防止跳入下面的扩容分支
                        // 之后会重新计算probe
                        collide = false;            // At max size or stale
                        // 设置冲突标志，重新执行循环
                        // 如果下次循环执行到该分支，并且冲突标志仍然为true
                        // 那么会跳过该分支，到下一个分支进行扩容
                    else if (!collide)
                        collide = true;
                        // 尝试加锁，然后对counterCells数组进行扩容
                    else if (cellsBusy == 0 &&
                            U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                        try {
                            // 检查是否已被扩容
                            if (counterCells == as) {// Expand table unless stale
                                // 新数组容量为之前的1倍
                                CounterCell[] rs = new CounterCell[n << 1];
                                // 迁移数据到新数组
                                for (int i = 0; i < n; ++i)
                                    rs[i] = as[i];
                                counterCells = rs;
                            }
                        } finally {
                            // 释放锁
                            cellsBusy = 0;
                        }
                        collide = false;
                        // 重新执行循环
                        continue;                   // Retry with expanded table
                    }
                    // 为当前线程重新计算probe
                    h = ThreadLocalRandom.advanceProbe(h);
                    // CounterCell数组未初始化，尝试获取自旋锁，然后进行初始化
                } else if (cellsBusy == 0 && counterCells == as &&
                        U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                    boolean init = false;
                    try {                           // Initialize table
                        if (counterCells == as) {
                            // 初始化CounterCell数组，初始容量为2
                            CounterCell[] rs = new CounterCell[2];
                            // 初始化CounterCell
                            rs[h & 1] = new CounterCell(x);
                            counterCells = rs;
                            init = true;
                        }
                    } finally {
                        cellsBusy = 0;
                    }
                    // 初始化CounterCell数组成功，退出循环
                    if (init)
                        break;
                    // 如果自旋锁被占用，则只好尝试更新baseCount
                } else if (U.compareAndSwapLong(this, BASECOUNT, v = baseCount, v + x))
                    break;                          // Fall back on using base
            }
        }

        /* ---------------- Conversion from/to TreeBins -------------- */

        /**
         * Replaces all linked nodes in bin at given index unless table is
         * too small, in which case resizes instead.
         */
        private final void treeifyBin(Node<K, V>[] tab, int index) {
            Node<K, V> b;
            int n, sc;
            if (tab != null) {
                if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
                    tryPresize(n << 1);
                else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
                    synchronized (b) {
                        if (tabAt(tab, index) == b) {
                            TreeNode<K, V> hd = null, tl = null;
                            for (Node<K, V> e = b; e != null; e = e.next) {
                                TreeNode<K, V> p =
                                        new TreeNode<K, V>(e.hash, e.key, e.val,
                                                null, null);
                                if ((p.prev = tl) == null)
                                    hd = p;
                                else
                                    tl.next = p;
                                tl = p;
                            }
                            setTabAt(tab, index, new TreeBin<K, V>(hd));
                        }
                    }
                }
            }
        }

        /**
         * Returns a list on non-TreeNodes replacing those in given list.
         * 返回非树节点上的列表，替换给定列表中的列表。
         */
        static <K, V> Node<K, V> untreeify(Node<K, V> b) {
            Node<K, V> hd = null, tl = null;
            for (Node<K, V> q = b; q != null; q = q.next) {
                Node<K, V> p = new Node<K, V>(q.hash, q.key, q.val, null);
                if (tl == null)
                    hd = p;
                else
                    tl.next = p;
                tl = p;
            }
            return hd;
        }

        /* ----------------Table Traversal -------------- */

        /**
         * Records the table, its length, and current traversal index for a
         * traverser that must process a region of a forwarded table before
         * proceeding with current table.
         */
        static final class TableStack<K, V> {
            int length;
            int index;
            Node<K, V>[] tab;
            TableStack<K, V> next;
        }

        /**
         * Encapsulates traversal for methods such as containsValue; also
         * serves as a base class for other iterators and spliterators.
         * <p>
         * 封装方法的遍历，如containsValue;还用作其他迭代器和spliterator的基类。
         * <p>
         * Method advance visits once each still-valid node that was
         * reachable upon iterator construction. It might miss some that
         * were added to a bin after the bin was visited, which is OK wrt
         * consistency guarantees. Maintaining this property in the face
         * of possible ongoing resizes requires a fair amount of
         * bookkeeping state that is difficult to optimize away amidst
         * volatile accesses.  Even so, traversal maintains reasonable
         * throughput.
         * <p>
         * 方法预先访问在迭代器构造时可访问的每个仍然有效的节点。
         * 它可能会遗漏一些在访问了bin之后添加到bin中的内容，这是wrt一致性的保证。
         * 面对可能正在进行的大小调整，维护此属性需要相当数量的簿记状态，而在易变访问中很难优化这些状态。
         * 即便如此，遍历仍然保持了合理的吞吐量。
         * <p>
         * Normally, iteration proceeds bin-by-bin traversing lists.
         * However, if the table has been resized, then all future steps
         * must traverse both the bin at the current index as well as at
         * (index + baseSize); and so on for further resizings. To
         * paranoically cope with potential sharing by users of iterators
         * across threads, iteration terminates if a bounds checks fails
         * for a table read.
         * <p>
         * 通常情况下，迭代按bin遍历列表。但是，如果表已经调整了大小，
         * 但是，如果表的大小已经调整，那么以后的所有步骤都必须遍历当前索引处的bin和at (index + baseSize);
         * 来进一步调整大小。进一步调整。为了偏执地处理线程间迭代器用户的潜在共享，如果读取的表的边界检查失败，迭代将终止。
         */
        static class Traverser<K, V> {
            Node<K, V>[] tab;        // current table; updated if resized 当前表;如果更新调整
            Node<K, V> next;         // the next entry to use 下一个要使用的entry
            TableStack<K, V> stack, spare; // to save/restore on Forwardin gNodes 转发节点时的存储与恢复
            int index;              // index of bin to use next  bin next 索引
            int baseIndex;          // current index of initial table 初始表的当前索引
            int baseLimit;          // index bound for initial table 初始表的索引边界
            final int baseSize;     // initial table size 初始表大小

            Traverser(Node<K, V>[] tab, int size, int index, int limit) {
                this.tab = tab;
                this.baseSize = size;
                this.baseIndex = this.index = index;
                this.baseLimit = limit;
                this.next = null;
            }

            /**
             * Advances if possible, returning next valid node, or null if none.
             * 如果可能，则前进，返回下一个有效节点，如果没有，则返回null。
             */
            final Node<K, V> advance() {
                Node<K, V> e;
                //如果下一节点存在，则走下一步
                if ((e = next) != null)
                    e = e.next;
                //死循环
                for (; ; ) {
                    Node<K, V>[] t;
                    int i, n;  // must use locals in checks
                    //如果e不为空
                    if (e != null)
                        return next = e;
                    //如果初始索引超过边界或者表为空，或者已经迭代到头
                    if (baseIndex >= baseLimit || (t = tab) == null ||
                            (n = t.length) <= (i = index) || i < 0)
                        return next = null;
                    //获取到节点且不为空
                    if ((e = tabAt(t, i)) != null && e.hash < 0) {
                        //如果是ForwardingNode节点
                        if (e instanceof ForwardingNode) {
                            tab = ((ForwardingNode<K, V>) e).nextTable;
                            e = null;
                            //保存状态
                            pushState(t, i, n);
                            continue;
                        }
                        //如果时树节点
                        else if (e instanceof TreeBin)
                            e = ((TreeBin<K, V>) e).first;
                        else
                            e = null;
                    }
                    //如果栈不为空
                    if (stack != null)
                        recoverState(n);
                    else if ((index = i + baseSize) >= n)
                        index = ++baseIndex; // visit upper slots if present
                }
            }

            /**
             * Saves traversal state upon encountering a forwarding node.
             * 保存遇到转发节点时的遍历状态。
             */
            private void pushState(Node<K, V>[] t, int i, int n) {
                TableStack<K, V> s = spare;  // reuse if possible
                if (s != null)
                    spare = s.next;
                else
                    s = new TableStack<K, V>();
                s.tab = t;
                s.length = n;
                s.index = i;
                s.next = stack;
                stack = s;
            }

            /**
             * Possibly pops traversal state.
             * pop遍历状态。
             *
             * @param n length of current table
             */
            private void recoverState(int n) {
                TableStack<K, V> s;
                int len;
                while ((s = stack) != null && (index += (len = s.length)) >= n) {
                    n = len;
                    index = s.index;
                    tab = s.tab;
                    s.tab = null;
                    TableStack<K, V> next = s.next;
                    s.next = spare; // save for reuse
                    stack = next;
                    spare = s;
                }
                if (s == null && (index += baseSize) >= n)
                    index = ++baseIndex;
            }
        }

        // Unsafe mechanics
        private static final sun.misc.Unsafe U;
        private static final long SIZECTL;
        private static final long TRANSFERINDEX;
        private static final long BASECOUNT;
        private static final long CELLSBUSY;
        private static final long CELLVALUE;
        private static final long ABASE;
        private static final int ASHIFT;

        static {
            try {
                U = sun.misc.Unsafe.getUnsafe();
                Class<?> k = ConcurrentHashMap.class;
                SIZECTL = U.objectFieldOffset
                        (k.getDeclaredField("sizeCtl"));
                TRANSFERINDEX = U.objectFieldOffset
                        (k.getDeclaredField("transferIndex"));
                BASECOUNT = U.objectFieldOffset
                        (k.getDeclaredField("baseCount"));
                CELLSBUSY = U.objectFieldOffset
                        (k.getDeclaredField("cellsBusy"));
                Class<?> ck = CounterCell.class;
                CELLVALUE = U.objectFieldOffset
                        (ck.getDeclaredField("value"));
                Class<?> ak = Node[].class;
                ABASE = U.arrayBaseOffset(ak);
                int scale = U.arrayIndexScale(ak);
                if ((scale & (scale - 1)) != 0)
                    throw new Error("data type scale not a power of two");
                ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);
            } catch (Exception e) {
                throw new Error(e);
            }
        }
    }
```