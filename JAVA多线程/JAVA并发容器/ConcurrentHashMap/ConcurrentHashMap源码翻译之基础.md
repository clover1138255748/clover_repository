基于JDK8

```
        /* ---------------- Constants -------------- */

        /**
         * The largest possible table capacity.  This value must be
         * exactly 1<<30 to stay within Java array allocation and indexing
         * bounds for power of two table sizes, and is further required
         * because the top two bits of 32bit hash fields are used for
         * control purposes.
         * <p>
         * 最大的表容量。这个值必须恰好是1<<30，才能保持在Java数组分配和索引范围内，
         * 以获得两种表大小的幂，而且这个值是进一步需要的，因为32位哈希字段的前两位用于控制目的。
         */
        private static final int MAXIMUM_CAPACITY = 1 << 30;

        /**
         * The default initial table capacity.  Must be a power of 2
         * (i.e., at least 1) and at most MAXIMUM_CAPACITY.
         * 默认的表初始容量，必须是2的幂次方，最小是1，最大为MAXIMUM_CAPACITY
         */
        private static final int DEFAULT_CAPACITY = 16;

        /**
         * The largest possible (non-power of two) array size.
         * Needed by toArray and related methods.
         * 最大的数组大小(非2次幂)。toArray and related方法使用。
         */
        static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

        /**
         * The default concurrency level for this table. Unused but
         * defined for compatibility with previous versions of this class.
         * 表默认的并发级别，未使用，但为与该类的以前版本兼容而定义。
         */
        private static final int DEFAULT_CONCURRENCY_LEVEL = 16;

        /**
         * The load factor for this table. Overrides of this value in
         * constructors affect only the initial table capacity.  The
         * actual floating point value isn't normally used -- it is
         * simpler to use expressions such as {@code n - (n >>> 2)} for
         * the associated resizing threshold.
         * 表的默认加载因子，在构造函数中重写此值只影响初始表容量。
         * 通常不使用实际的浮点值 —— 对于相关的调整阈值，使用{@code n - (n >>> 2)}等表达式更简单。
         */
        private static final float LOAD_FACTOR = 0.75f;

        /**
         * The bin count threshold for using a tree rather than list for a
         * bin.  Bins are converted to trees when adding an element to a
         * bin with at least this many nodes. The value must be greater
         * than 2, and should be at least 8 to mesh with assumptions in
         * tree removal about conversion back to plain bins upon
         * shrinkage.
         * 使用树(而不是列表)来设置bin计数阈值。当向至少具有这么多节点的bin添加元素时，bin将转换为树。
         * 该值必须大于2，并且应该至少为8，以便与树移除中关于收缩后转换回普通Bin的假设相吻合。
         */
        static final int TREEIFY_THRESHOLD = 8;

        /**
         * The bin count threshold for untreeifying a (split) bin during a
         * resize operation. Should be less than TREEIFY_THRESHOLD, and at
         * most 6 to mesh with shrinkage detection under removal.
         * 用于在调整大小操作期间反树化(拆分)bin的bin计数阈值，应该小于TREEIFY_THRESHOLD，
         * 最多6个，去吻合去缩检测。
         */
        static final int UNTREEIFY_THRESHOLD = 6;

        /**
         * The smallest table capacity for which bins may be treeified.
         * (Otherwise the table is resized if too many nodes in a bin.)
         * The value should be at least 4 * TREEIFY_THRESHOLD to avoid
         * conflicts between resizing and treeification thresholds.
         * 最小的表容量，其中的箱子可以treeified。(否则，如果一个bin中有太多节点，则会调整表的大小。)
         * 该值应该至少为4 * TREEIFY_THRESHOLD，以避免调整大小和treeification阈值之间的冲突。
         */
        static final int MIN_TREEIFY_CAPACITY = 64;

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
         * For serialization compatibility.
         * 序列化的兼容性
         */
        private static final ObjectStreamField[] serialPersistentFields = {
                new ObjectStreamField("segments", Segment[].class),
                new ObjectStreamField("segmentMask", Integer.TYPE),
                new ObjectStreamField("segmentShift", Integer.TYPE)
        };

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
        static class Node<K, V> implements Entry<K, V> {
            //哈希码
            final int hash;
            //键
            final K key;
            //值
            volatile V val;
            //下一个节点
            volatile Node<K, V> next;

            Node(int hash, K key, V val, Node<K, V> next) {
                this.hash = hash;
                this.key = key;
                this.val = val;
                this.next = next;
            }

            public final K getKey() {
                return key;
            }

            public final V getValue() {
                return val;
            }

            public final int hashCode() {
                return key.hashCode() ^ val.hashCode();
            }

            public final String toString() {
                return key + "=" + val;
            }

            public final V setValue(V value) {
                throw new UnsupportedOperationException();
            }

            public final boolean equals(Object o) {
                Object k, v, u;
                Entry<?, ?> e;
                return ((o instanceof Map.Entry) &&
                        (k = (e = (Entry<?, ?>) o).getKey()) != null &&
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
            Node<K, V> find(int h, Object k) {
                //当前节点
                Node<K, V> e = this;
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

        /* ---------------- Static utilities 静态工具 -------------- */

        /**
         * Spreads (XORs) higher bits of hash to lower and also forces top
         * bit to 0. Because the table uses power-of-two masking, sets of
         * hashes that vary only in bits above the current mask will
         * always collide. (Among known examples are sets of Float keys
         * holding consecutive whole numbers in small tables.)  So we
         * apply a transform that spreads the impact of higher bits
         * downward. There is a tradeoff between speed, utility, and
         * quality of bit-spreading. Because many common sets of hashes
         * are already reasonably distributed (so don't benefit from
         * spreading), and because we use trees to handle large sets of
         * collisions in bins, we just XOR some shifted bits in the
         * cheapest possible way to reduce systematic lossage, as well as
         * to incorporate impact of the highest bits that would otherwise
         * never be used in index calculations because of table bounds.
         * <p>
         * 将较高的哈希值扩展(XORs)为较低的哈希值，并将最高位强制为0。
         * 由于该表使用了2的幂掩码，因此仅在当前掩码之上以位为单位变化的散列集总是会发生冲突。
         * (已知的例子包括在小表中保存连续整数的浮点键集。)因此，我们应用一个转换，将更高位的影响向下传播。
         * 位扩展的速度、实用性和质量之间存在权衡。
         * 因为许多常见的散列集已经合理分布(所以不要受益于传播),在桶中我们用树来处理大型的碰撞,
         * 我们只是XOR一些改变以最划算的方式来减少系统lossage,以及最高位的影响,否则就不会因为表的边界而在索引计算中使用。
         */
        static final int spread(int h) {
            return (h ^ (h >>> 16)) & HASH_BITS;
        }

        /**
         * Returns a power of two table size for the given desired capacity.
         * See Hackers Delight, sec 3.2
         * 返回给定目标容量的2次幂
         */
        private static final int tableSizeFor(int c) {
            int n = c - 1;
            n |= n >>> 1;
            n |= n >>> 2;
            n |= n >>> 4;
            n |= n >>> 8;
            n |= n >>> 16;
            return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
        }

        /**
         * Returns x's Class if it is of the form "class C implements
         * Comparable<C>", else null.
         * 如果x实现了Comparable<C>接口，则返回此Class,否则返回null
         */
        static Class<?> comparableClassFor(Object x) {
            if (x instanceof Comparable) {
                Class<?> c;
                Type[] ts, as;
                Type t;
                ParameterizedType p;
                if ((c = x.getClass()) == String.class) // bypass checks
                    return c;
                if ((ts = c.getGenericInterfaces()) != null) {
                    for (int i = 0; i < ts.length; ++i) {
                        if (((t = ts[i]) instanceof ParameterizedType) &&
                                ((p = (ParameterizedType) t).getRawType() ==
                                        Comparable.class) &&
                                (as = p.getActualTypeArguments()) != null &&
                                as.length == 1 && as[0] == c) // type arg is c
                            return c;
                    }
                }
            }
            return null;
        }

        /**
         * Returns k.compareTo(x) if x matches kc (k's screened comparable
         * class), else 0.
         * 比较大小
         */
        @SuppressWarnings({"rawtypes", "unchecked"}) // for cast to Comparable
        static int compareComparables(Class<?> kc, Object k, Object x) {
            return (x == null || x.getClass() != kc ? 0 :
                    ((Comparable) k).compareTo(x));
        }

        /* ---------------- Table element access 表元素访问，核心了-------------- */

        /*
         * Volatile access methods are used for table elements as well as
         * elements of in-progress next table while resizing.  All uses of
         * the tab arguments must be null checked by callers.  All callers
         * also paranoically precheck that tab's length is not zero (or an
         * equivalent check), thus ensuring that any index argument taking
         * the form of a hash value anded with (length - 1) is a valid
         * index.  Note that, to be correct wrt arbitrary concurrency
         * errors by users, these checks must operate on local variables,
         * which accounts for some odd-looking inline assignments below.
         * Note that calls to setTabAt always occur within locked regions,
         * and so in principle require only release ordering, not
         * full volatile semantics, but are currently coded as volatile
         * writes to be conservative.
         * 在调整大小时，易失性访问方法用于表元素以及正在进行中的下一个表的元素。
         * 调用者必须检查选项卡参数的所有使用是否为空。
         * 所有调用者还偏执地预先检查制表符的长度是否为零(或等效的检查)，从而确保任何以散列值形式出现的索引参数(长度- 1)都是有效的索引。
         * 注意，要纠正用户的wrt任意并发性错误，这些检查必须对本地变量进行操作，这就解释了下面一些奇怪的内联分配。
         * 注意，对setTabAt的调用总是发生在锁定区域内，
         * 因此原则上只需要释放顺序，而不需要完全的volatile语义，但是目前将其编码为volatile写，以保持保守性。
         */

        @SuppressWarnings("unchecked")
        static final <K, V> Node<K, V> tabAt(Node<K, V>[] tab, int i) {
            //native 方法 获取当前节点
            return (Node<K, V>) U.getObjectVolatile(tab, ((long) i << ASHIFT) + ABASE);
        }

        static final <K, V> boolean casTabAt(Node<K, V>[] tab, int i,
                                             Node<K, V> c, Node<K, V> v) {
            //native 方法 cas 获取tab数组中索引为i的节点，如果是和c相同，则更新为节点v
            return U.compareAndSwapObject(tab, ((long) i << ASHIFT) + ABASE, c, v);
        }

        static final <K, V> void setTabAt(Node<K, V>[] tab, int i, Node<K, V> v) {
            // native 方法 更新节点
            U.putObjectVolatile(tab, ((long) i << ASHIFT) + ABASE, v);
        }

        /* ---------------- Fields 字段-------------- */

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
         * Base counter value, used mainly when there is no contention,
         * but also as a fallback during table initialization
         * races. Updated via CAS.
         * 基本计数器值，主要在没有争用时使用，也可作为表初始化竞争期间的回退。通过CAS更新。
         */
        private transient volatile long baseCount;

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
         * Spinlock (locked via CAS) used when resizing and/or creating CounterCells.
         * 自旋锁(通过CAS锁定)，用于调整大小and/or 创建CounterCells。
         * cellsBusy是一个只有0和1两个状态的volatile整数
         * 它被当做一个自旋锁，0代表无锁，1代表加锁
         */
        private transient volatile int cellsBusy;

        /**
         * Table of counter cells. When non-null, size is a power of 2.
         * 计数器单元格表。当非空时，size是2的幂。
         */
        private transient volatile CounterCell[] counterCells;

        /* ---------------- Public operations -------------- */

        /**
         * Creates a new, empty map with the default initial table size (16).
         * 创建一个默认初始化容量为16的新的空的表
         */
        public ConcurrentHashMap() {
        }

        /**
         * Creates a new, empty map with an initial table size
         * accommodating the specified number of elements without the need
         * to dynamically resize.
         * 创建一个新的空映射，初始表大小可容纳指定数量的元素，而不需要动态调整大小。
         *
         * @param initialCapacity The implementation performs internal
         *                        sizing to accommodate this many elements. 初始容量
         * @throws IllegalArgumentException if the initial capacity of
         *                                  elements is negative
         */
        public ConcurrentHashMap(int initialCapacity) {
            if (initialCapacity < 0)
                throw new IllegalArgumentException();
            //如果初试容量大于最大初始容量无符号右移1位，则cap取值为最大初始容量，否则计算给定目标容量的最小2次幂的值
            int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
                    MAXIMUM_CAPACITY :
                    tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
            //将表初始和扩容控制值 赋值
            this.sizeCtl = cap;
        }

        /**
         * Creates a new map with the same mappings as the given map.
         * 创建与给定映射具有相同映射的新映射。
         *
         * @param m the map
         */
        public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
            //默认表容量为16
            this.sizeCtl = DEFAULT_CAPACITY;
            //批量插入
            putAll(m);
        }

        /**
         * Creates a new, empty map with an initial table size based on
         * the given number of elements ({@code initialCapacity}) and
         * initial table density ({@code loadFactor}).
         * 用给定的初始容量和加载因子创建一个新的空的map，一个并发更新线程。
         *
         * @param initialCapacity the initial capacity. The implementation
         *                        performs internal sizing to accommodate this many elements,
         *                        given the specified load factor.  初始容量
         * @param loadFactor      the load factor (table density) for
         *                        establishing the initial table size 加载因子
         * @throws IllegalArgumentException if the initial capacity of
         *                                  elements is negative or the load factor is nonpositive
         * @since 1.6
         */
        public ConcurrentHashMap(int initialCapacity, float loadFactor) {
            this(initialCapacity, loadFactor, 1);
        }

        /**
         * Creates a new, empty map with an initial table size based on
         * the given number of elements ({@code initialCapacity}), table
         * density ({@code loadFactor}), and number of concurrently
         * updating threads ({@code concurrencyLevel}).
         * 根据给定的元素数量({@code initialCapacity})、表密度({@code loadFactor})和并发更新线程数量({@code concurrencyLevel})创建一个新的空映射，初始表大小。
         *
         * @param initialCapacity  the initial capacity. The implementation
         *                         performs internal sizing to accommodate this many elements,
         *                         given the specified load factor. 初始容量
         * @param loadFactor       the load factor (table density) for
         *                         establishing the initial table size 加载因子
         * @param concurrencyLevel the estimated number of concurrently
         *                         updating threads. The implementation may use this value as
         *                         a sizing hint. 并发等级
         * @throws IllegalArgumentException if the initial capacity is
         *                                  negative or the load factor or concurrencyLevel are
         *                                  nonpositive
         */
        public ConcurrentHashMap(int initialCapacity,
                                 float loadFactor, int concurrencyLevel) {
            //入参校验
            if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
                throw new IllegalArgumentException();

            //如果初始容量小于并发等级，则初始容量变为并发等级大小
            if (initialCapacity < concurrencyLevel)   // Use at least as many bins
                initialCapacity = concurrencyLevel;   // as estimated threads
            //计算表容量
            long size = (long) (1.0 + (long) initialCapacity / loadFactor);
            //如果表容量大于等于最大容量，则取最大容量值，否则取size值的最小2次幂的值
            int cap = (size >= (long) MAXIMUM_CAPACITY) ?
                    MAXIMUM_CAPACITY : tableSizeFor((int) size);
            //赋值 sizeCtl
            this.sizeCtl = cap;
        }

        // Original (since JDK1.2) Map methods
        // 原始的map方法

        /**
         * 计算容量
         * {@inheritDoc}
         */
        public int size() {
            long n = sumCount();
            return ((n < 0L) ? 0 :
                    (n > (long) Integer.MAX_VALUE) ? Integer.MAX_VALUE :
                            (int) n);
        }

        /**
         * 判断map是否为空
         * {@inheritDoc}
         */
        public boolean isEmpty() {
            return sumCount() <= 0L; // ignore transient negative values 忽略瞬时负值
        }

        /**
         * Returns the value to which the specified key is mapped,
         * or {@code null} if this map contains no mapping for the key.
         * <p>
         * 返回指定键映射到的值，如果此映射不包含键的映射，则返回{@code null}。
         *
         * <p>More formally, if this map contains a mapping from a key
         * {@code k} to a value {@code v} such that {@code key.equals(k)},
         * then this method returns {@code v}; otherwise it returns
         * {@code null}.  (There can be at most one such mapping.)
         * <p>
         * 更正式地说，如果这个映射包含一个键{@code k}到一个值{@code v}的映射，
         * 使得{@code key.equals(k)}，那么这个方法返回{@code v};否则返回{@code null}。
         * (最多可以有一个这样的映射。)
         *
         * @throws NullPointerException if the specified key is null 如果指定的键为空
         */
        public V get(Object key) {
            Node<K, V>[] tab;
            Node<K, V> e, p;
            int n, eh;
            K ek;
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

        /**
         * Tests if the specified object is a key in this table.
         * 测试指定的对象是否是该表中的键。
         *
         * @param key possible key 指定的键
         * @return {@code true} if and only if the specified object
         * is a key in this table, as determined by the
         * {@code equals} method; {@code false} otherwise
         * @throws NullPointerException if the specified key is null
         */
        public boolean containsKey(Object key) {
            return get(key) != null;
        }

        /**
         * Returns {@code true} if this map maps one or more keys to the
         * specified value. Note: This method may require a full traversal
         * of the map, and is much slower than method {@code containsKey}.
         * <p>
         * 如果此映射将一个或多个键映射到指定值，则返回{@code true}。
         * 注意:这个方法可能需要遍历整个map，并且比方法{@code containsKey}慢得多。
         *
         * @param value value whose presence in this map is to be tested
         * @return {@code true} if this map maps one or more keys to the
         * specified value
         * @throws NullPointerException if the specified value is null
         */
        public boolean containsValue(Object value) {
            if (value == null)
                throw new NullPointerException();
            Node<K, V>[] t;
            if ((t = table) != null) {

                //表遍历者,这个好消耗性能
                Traverser<K, V> it = new Traverser<K, V>(t, t.length, 0, t.length);
                for (Node<K, V> p; (p = it.advance()) != null; ) {
                    V v;
                    //存在此值，返回true
                    if ((v = p.val) == value || (v != null && value.equals(v)))
                        return true;
                }
            }
            return false;
        }

        /**
         * Maps the specified key to the specified value in this table.
         * Neither the key nor the value can be null.
         * <p>
         * 将指定的键映射到该表中的指定值。键和值都不能为空。
         *
         * <p>The value can be retrieved by calling the {@code get} method
         * with a key that is equal to the original key.
         * <p>
         * 可以通过调用{@code get}方法来检索该值，方法的键值等于原始键值。
         *
         * @param key   key with which the specified value is to be associated 要与指定值关联的键
         * @param value value to be associated with the specified key 值与指定的键关联
         * @return the previous value associated with {@code key}, or
         * {@code null} if there was no mapping for {@code key}
         * @throws NullPointerException if the specified key or value is null
         */
        public V put(K key, V value) {
            return putVal(key, value, false);
        }

        /**
         * Implementation for put and putIfAbsent
         * 实现put和putIfAbsent
         * onlyIfAbsent含义：如果我们传入的key已存在我们是否去替换，true:不替换，false：替换。
         */
        final V putVal(K key, V value, boolean onlyIfAbsent) {
            //键值都不为空
            if (key == null || value == null) throw new NullPointerException();
            //发散键的hash值
            int hash = spread(key.hashCode());
            //桶数量 0
            int binCount = 0;
            for (Node<K, V>[] tab = table; ; ) {
                Node<K, V> f;
                int n, i, fh;
                //检查表是否初始化
                if (tab == null || (n = tab.length) == 0)
                    //初始化表
                    tab = initTable();
                    //检查指定键所在节点是否为空
                else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                    //通过CAS方法添加键值对
                    if (casTabAt(tab, i, null,
                            new Node<K, V>(hash, key, value, null)))
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
                                for (Node<K, V> e = f; ; ++binCount) {
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
                                    Node<K, V> pred = e;
                                    //下一个节点为空
                                    if ((e = e.next) == null) {
                                        //新建节点
                                        pred.next = new Node<K, V>(hash, key,
                                                value, null);
                                        break;
                                    }
                                }
                            }
                            //如果时树节点
                            else if (f instanceof TreeBin) {
                                Node<K, V> p;
                                //桶数量为2
                                binCount = 2;
                                //找到插入的位置，插入节点，平衡插入
                                if ((p = ((TreeBin<K, V>) f).putTreeVal(hash, key,
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

        /**
         * Copies all of the mappings from the specified map to this one.
         * These mappings replace any mappings that this map had for any of the
         * keys currently in the specified map.
         * <p>
         * 将指定映射的所有映射复制到此映射，这些映射替换了当前指定映射中任意键的映射。
         *
         * @param m mappings to be stored in this map
         */
        public void putAll(Map<? extends K, ? extends V> m) {
            //尝试调整表的大小以适应给定的元素数量。
            tryPresize(m.size());
            //迭代插入
            for (Entry<? extends K, ? extends V> e : m.entrySet())
                putVal(e.getKey(), e.getValue(), false);
        }

        /**
         * Removes the key (and its corresponding value) from this map.
         * This method does nothing if the key is not in the map.
         * <p>
         * 从映射中删除键(及其对应值)。如果键不在映射中，则此方法不执行任何操作。
         *
         * @param key the key that needs to be removed 指定移除的键
         * @return the previous value associated with {@code key}, or
         * {@code null} if there was no mapping for {@code key} 如果没有针对{@code key}的映射，则与{@code key}关联的前一个值或{@code null}
         * @throws NullPointerException if the specified key is null 如果指定的键为空
         */
        public V remove(Object key) {
            return replaceNode(key, null, null);
        }

        /**
         * Implementation for the four public remove/replace methods:
         * Replaces node value with v, conditional upon match of cv if
         * non-null.  If resulting value is null, delete.
         * 删除/替换的操作
         */
        final V replaceNode(Object key, V value, Object cv) {
            //发散hash
            int hash = spread(key.hashCode());
            //迭代表
            for (Node<K, V>[] tab = table; ; ) {
                Node<K, V> f;
                int n, i, fh;
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
                                for (Node<K, V> e = f, pred = null; ; ) {
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
                                TreeBin<K, V> t = (TreeBin<K, V>) f;
                                TreeNode<K, V> r, p;
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

        /**
         * Removes all of the mappings from this map.
         * 清空map
         */
        public void clear() {
            long delta = 0L; // negative number of deletions 删除次数为负数
            int i = 0;
            Node<K, V>[] tab = table;
            while (tab != null && i < tab.length) {
                int fh;
                //获取节点
                Node<K, V> f = tabAt(tab, i);
                if (f == null)
                    ++i;
                    //转发节点的哈希
                else if ((fh = f.hash) == MOVED) {
                    //如果还在进行扩容操作就先进行扩容
                    tab = helpTransfer(tab, f);
                    i = 0; // restart
                } else {
                    //锁定当前节点
                    synchronized (f) {
                        //确认节点
                        if (tabAt(tab, i) == f) {
                            //获取节点，包括树节点
                            Node<K, V> p = (fh >= 0 ? f :
                                    (f instanceof TreeBin) ?
                                            ((TreeBin<K, V>) f).first : null);
                            while (p != null) {
                                //数量减一
                                --delta;
                                //下一个节点
                                p = p.next;
                            }
                            //置空
                            setTabAt(tab, i++, null);
                        }
                    }
                }
            }
            if (delta != 0L)
                //计数
                addCount(delta, -1);
        }

        /**
         * Returns the hash code value for this {@link Map}, i.e.,
         * the sum of, for each key-value pair in the map,
         * {@code key.hashCode() ^ value.hashCode()}.
         * <p>
         * 返回这个{@link Map}的哈希码值，即，对于映射中的每个键值对，{@code key.hashCode() ^ value.hashCode()}的和。
         *
         * @return the hash code value for this map
         */
        public int hashCode() {
            int h = 0;
            Node<K, V>[] t;
            if ((t = table) != null) {
                Traverser<K, V> it = new Traverser<K, V>(t, t.length, 0, t.length);
                for (Node<K, V> p; (p = it.advance()) != null; )
                    //取键的Hash值与值的Hash值的异或 的 和
                    h += p.key.hashCode() ^ p.val.hashCode();
            }
            return h;
        }

        /**
         * Compares the specified object with this map for equality.
         * Returns {@code true} if the given object is a map with the same
         * mappings as this map.  This operation may return misleading
         * results if either map is concurrently modified during execution
         * of this method.
         * <p>
         * 重写的equals方法
         *
         * @param o object to be compared for equality with this map
         * @return {@code true} if the specified object is equal to this map
         */
        public boolean equals(Object o) {
            //如果栈中值不相等
            if (o != this) {
                //如果o未实现Map接口，则不相等
                if (!(o instanceof Map))
                    return false;
                Map<?, ?> m = (Map<?, ?>) o;
                Node<K, V>[] t;
                int f = (t = table) == null ? 0 : t.length;
                Traverser<K, V> it = new Traverser<K, V>(t, f, 0, f);
                //迭代，只要有一个值不相等，则返回false
                for (Node<K, V> p; (p = it.advance()) != null; ) {
                    V val = p.val;
                    Object v = m.get(p.key);
                    if (v == null || (v != val && !v.equals(val)))
                        return false;
                }
                //迭代视图
                for (Entry<?, ?> e : m.entrySet()) {
                    Object mk, mv, v;
                    if ((mk = e.getKey()) == null ||
                            (mv = e.getValue()) == null ||
                            (v = get(mk)) == null ||
                            (mv != v && !mv.equals(v)))
                        return false;
                }
            }
            return true;
        }

        /**
         * Stripped-down version of helper class used in previous version,
         * declared for the sake of serialization compatibility
         * <p>
         * 以前版本中使用的helper类的简化版本，为了序列化兼容性而声明
         * 继承自重入锁
         */
        static class Segment<K, V> extends ReentrantLock implements Serializable {
            private static final long serialVersionUID = 2249069246763182397L;
            final float loadFactor;

            Segment(float lf) {
                this.loadFactor = lf;
            }
        }

        // Overrides of JDK8+ Map extension method defaults

        /**
         * Returns the value to which the specified key is mapped, or the
         * given default value if this map contains no mapping for the
         * key.
         * 返回指定键映射到的值，如果此映射不包含键的映射，则返回给定的默认值。
         *
         * @param key          the key whose associated value is to be returned
         * @param defaultValue the value to return if this map contains
         *                     no mapping for the given key
         * @return the mapping for the key, if present; else the default value
         * @throws NullPointerException if the specified key is null
         */
        public V getOrDefault(Object key, V defaultValue) {
            V v;
            return (v = get(key)) == null ? defaultValue : v;
        }
```