计数操作，并发情况下好复杂。
JDK8ConcurrentHashMap在插入和删除的情况下都会调用addCount方法：

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
```

其中baseCount是volatile修饰的变量，通过CAS修改其值：

```
        /**
         * Base counter value, used mainly when there is no contention,
         * but also as a fallback during table initialization
         * races. Updated via CAS.
         * 基本计数器值，主要在没有争用时使用，也可作为表初始化竞争期间的回退。通过CAS更新。
         */
        private transient volatile long baseCount;

        private static final long BASECOUNT = sun.misc.Unsafe.getUnsafe().objectFieldOffset
                        (ConcurrentHashMap.class.getDeclaredField("baseCount"));
```

CounterCell是一个简单的内部静态类，每个CounterCell都是一个用于记录数量的单元：

```
        /**
         * Table of counter cells. When non-null, size is a power of 2.
         * 计数器单元格表。当非空时，size是2的幂。
         */
        private transient volatile CounterCell[] counterCells;

         /**
         * A padded cell for distributing counts.  Adapted from LongAdder
         * and Striped64.  See their internal docs for explanation.
         * 用于分配计数的填充单元格。改编自LongAdder and Striped64
         * 请参阅他们的内部文档进行解释。
         * 
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
```

ThreadLocalRandom.getProbe():
ThreadLocalRandom是一个线程私有的伪随机数生成器，
每个线程的probe都是不同的
（这点基于ThreadLocalRandom的内部实现，
它在内部维护了一个probeGenerator，
这是一个类型为AtomicInteger的静态常量，
每当初始化一个ThreadLocalRandom时probeGenerator都会先自增一个常量然后返回的整数即为当前线程的probe，
probe变量被维护在Thread对象中），可以认为每个线程的probe就是它在CounterCell数组中的hash code。

fullAddCount()函数根据当前线程的probe寻找对应的CounterCell进行计数，如果CounterCell数组未被初始化，则初始化CounterCell数组和CounterCell。
这个方法太他妈复杂了，全程懵逼。

```
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
```

CounterCell数组的大小永远是一个2的n次方，初始容量为2，每次扩容的新容量都是之前容量乘以二，处于性能考虑，它的最大容量上限是机器的CPU数量。
所以说CounterCell数组的碰撞冲突是很严重的，因为它的bucket基数太小了。
而发生碰撞就代表着一个CounterCell会被多个线程竞争，
为了解决这个问题，Doug Lea使用无限循环加上CAS来模拟出一个自旋锁来保证线程安全，
自旋锁的实现基于一个被volatile修饰的整数变量，该变量只会有两种状态：0和1，当它被设置为0时表示没有加锁，当它被设置为1时表示已被其他线程加锁。
这个自旋锁用于保护初始化CounterCell、初始化CounterCell数组以及对CounterCell数组进行扩容时的安全。

```
      /**
         * Spinlock (locked via CAS) used when resizing and/or creating CounterCells.
         * 自旋锁(通过CAS锁定)，用于调整大小and/or 创建CounterCells。
         * cellsBusy是一个只有0和1两个状态的volatile整数
         * 它被当做一个自旋锁，0代表无锁，1代表加锁
         */
        private transient volatile int cellsBusy;
```

统计总和就比较简单了：

```
        /**
         * 计算ConcurrentHashMap中元素的个数
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
```