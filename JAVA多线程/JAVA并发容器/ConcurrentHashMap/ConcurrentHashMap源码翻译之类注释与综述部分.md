### 类注释

```
    /**
     * A hash table supporting full concurrency of retrievals and
     * high expected concurrency for updates. This class obeys the
     * same functional specification as {@link Hashtable}, and
     * includes versions of methods corresponding to each method of
     * {@code Hashtable}. However, even though all operations are
     * thread-safe, retrieval operations do <em>not</em> entail locking,
     * and there is <em>not</em> any support for locking the entire table
     * in a way that prevents all access.  This class is fully
     * interoperable with {@code Hashtable} in programs that rely on its
     * thread safety but not on its synchronization details.
     *
     * 支持检索的完全并发性和更新的高期望并发性的哈希表。该类遵循与{@link Hashtable}相同的功能规范，
     * 并包含与{@code Hashtable}的每个方法对应的方法版本。
     * 然而，即使所有操作都是线程安全的，检索操作也不需要锁定，而且不支持以阻止所有访问的方式锁定整个表。
     * 在依赖线程安全而不依赖同步细节的程序中，该类完全可以与{@code Hashtable}互操作。
     *
     *
     * <p>Retrieval operations (including {@code get}) generally do not
     * block, so may overlap with update operations (including {@code put}
     * and {@code remove}). Retrievals reflect the results of the most
     * recently <em>completed</em> update operations holding upon their
     * onset. (More formally, an update operation for a given key bears a
     * <em>happens-before</em> relation with any (non-null) retrieval for
     * that key reporting the updated value.)  For aggregate operations
     * such as {@code putAll} and {@code clear}, concurrent retrievals may
     * reflect insertion or removal of only some entries.  Similarly,
     * Iterators, Spliterators and Enumerations return elements reflecting the
     * state of the hash table at some point at or since the creation of the
     * iterator/enumeration.  They do <em>not</em> throw {@link
     * java.util.ConcurrentModificationException ConcurrentModificationException}.
     * However, iterators are designed to be used by only one thread at a time.
     * Bear in mind that the results of aggregate status methods including
     * {@code size}, {@code isEmpty}, and {@code containsValue} are typically
     * useful only when a map is not undergoing concurrent updates in other threads.
     * Otherwise the results of these methods reflect transient states
     * that may be adequate for monitoring or estimation purposes, but not
     * for program control.
     *
     * 检索操作(包括{@code get})通常不会阻塞，因此可能与更新操作(包括{@code put}和{@code remove})重叠。
     * 检索反映最近完成的更新操作在开始时的结果。(更正式地说，给定键的更新操作与报告更新值的键的任何(非null)检索都具有happens-before关系。)
     * 对于诸如{@code putAll}和{@code clear}之类的聚合操作，并发检索可能只反映插入或删除某些条目。
     * 类似地，迭代器、Spliterators和枚举返回反映哈希表在迭代器/枚举创建时或创建后的状态的元素。
     * 它们不会抛出{@link java.util.ConcurrentModificationException ConcurrentModificationException}。但是，迭代器一次只能被一个线程使用。
     * 请记住，聚合状态方法(包括{@code size}、{@code isEmpty}和{@code containsValue})的结果通常只有在映射没有在其他线程中进行并发更新时才有用。
     * 否则，这些方法的结果反映的瞬态状态可能足以用于监测或估计目的，但不用于程序控制。
     *
     * <p>The table is dynamically expanded when there are too many
     * collisions (i.e., keys that have distinct hash codes but fall into
     * the same slot modulo the table size), with the expected average
     * effect of maintaining roughly two bins per mapping (corresponding
     * to a 0.75 load factor threshold for resizing). There may be much
     * variance around this average as mappings are added and removed, but
     * overall, this maintains a commonly accepted time/space tradeoff for
     * hash tables.  However, resizing this or any other kind of hash
     * table may be a relatively slow operation. When possible, it is a
     * good idea to provide a size estimate as an optional {@code
     * initialCapacity} constructor argument. An additional optional
     * {@code loadFactor} constructor argument provides a further means of
     * customizing initial table capacity by specifying the table density
     * to be used in calculating the amount of space to allocate for the
     * given number of elements.  Also, for compatibility with previous
     * versions of this class, constructors may optionally specify an
     * expected {@code concurrencyLevel} as an additional hint for
     * internal sizing.  Note that using many keys with exactly the same
     * {@code hashCode()} is a sure way to slow down performance of any
     * hash table. To ameliorate impact, when keys are {@link Comparable},
     * this class may use comparison order among keys to help break ties.
     *
     * 当冲突太多时，表会动态扩展，每个映射维护大约两个桶的平均预期效果(对应于调整大小的0.75负载因子阈值)
     * 随着映射的添加和删除，这个平均值周围可能会有很大的差异，但总的来说，这维护了哈希表的普遍接受的时间/空间权衡。
     * 然而，调整这个或任何其他类型的散列 table可能是一个相对较慢的操作。
     * 如果可能，最好提供一个大小估计作为一个可选的{@code initialCapacity}构造函数参数。
     * 另外一个可选的{@code loadFactor}构造函数参数提供了定制初始表容量的进一步方法，它指定了用于计算给定元素数量分配的空间量的表密度。
     * 此外，为了与该类的以前版本兼容，构造函数可以选择指定一个预期的{@code concurrencyLevel}作为内部分级的额外提示。
     * 注意，使用具有完全相同的{@code hashCode()}的许多键肯定会降低任何散列表的性能。
     * 为了改善影响，当键是{@link Comparable}时，该类可以使用键之间的比较顺序来帮助断开连接。
     *
     *
     * <p>A {@link Set} projection of a ConcurrentHashMap may be created
     * (using {@link #newKeySet()} or {@link #newKeySet(int)}), or viewed
     * (using {@link #keySet(Object)} when only keys are of interest, and the
     * mapped values are (perhaps transiently) not used or all take the
     * same mapping value.
     *
     * 可以创建ConcurrentHashMap的{@link Set}投影(使用{@link #newKeySet()}或{@link #newKeySet(int)})，
     * 也可以查看(使用{@link #keySet(Object)}，如果只对键感兴趣，并且映射的值(可能暂时)没有使用，或者全部使用相同的映射值。
     *
     *
     * <p>A ConcurrentHashMap can be used as scalable frequency map (a
     * form of histogram or multiset) by using {@link
     * java.util.concurrent.atomic.LongAdder} values and initializing via
     * {@link #computeIfAbsent computeIfAbsent}. For example, to add a count
     * to a {@code ConcurrentHashMap<String,LongAdder> freqs}, you can use
     * {@code freqs.computeIfAbsent(k -> new LongAdder()).increment();}
     *
     * ConcurrentHashMap可以用作可伸缩的频率映射（直方图或多集的一种形式）。
     *
     * <p>This class and its views and iterators implement all of the
     * <em>optional</em> methods of the {@link Map} and {@link Iterator}
     * interfaces.
     *  这个类，视图，迭代器都是实现了Map和Iterator接口。
     *
     * <p>Like {@link Hashtable} but unlike {@link HashMap}, this class
     * does <em>not</em> allow {@code null} to be used as a key or value.
     * 这个类不允许null键与null值。
     *
     * <p>ConcurrentHashMaps support a set of sequential and parallel bulk
     * operations that, unlike most {@link Stream} methods, are designed
     * to be safely, and often sensibly, applied even with maps that are
     * being concurrently updated by other threads; for example, when
     * computing a snapshot summary of the values in a shared registry.
     * There are three kinds of operation, each with four forms, accepting
     * functions with Keys, Values, Entries, and (Key, Value) arguments
     * and/or return values. Because the elements of a ConcurrentHashMap
     * are not ordered in any particular way, and may be processed in
     * different orders in different parallel executions, the correctness
     * of supplied functions should not depend on any ordering, or on any
     * other objects or values that may transiently change while
     * computation is in progress; and except for forEach actions, should
     * ideally be side-effect-free. Bulk operations on {@link Entry}
     * objects do not support method {@code setValue}.
     *
     * ConcurrentHashMaps支持一组顺序的和并行的批量操作，与大多数{@link Stream}方法不同，
     * 这些操作的设计是安全的，而且通常是明智的，甚至可以应用于由其他线程并发更新的映射;
     * 例如，在计算共享注册表中值的快照摘要时。有三种操作，每种都有四种形式，接受带有键、值、条目和(键、值)参数和/或返回值的函数。
     * 因为ConcurrentHashMap的元素不是命令在任何特定的方式,并可能在不同的订单处理并行执行的正确性提供功能不应该依赖于任何命令,
     * 或任何其他对象或值可能是暂时性的变化而计算正在进行中;除了每个动作，最好是没有副作用的。
     * 对象上的批量操作不支持方法{@code setValue}。
     *
     * <ul>
     * <li> forEach: Perform a given action on each element.
     * A variant form applies a given transformation on each element
     * before performing the action.</li>
     *
     * 对每个元素执行给定的操作。变体形式在执行操作之前对每个元素应用给定的转换。
     *
     * <li> search: Return the first available non-null result of
     * applying a given function on each element; skipping further
     * search when a result is found.</li>
     *
     * 搜索:返回对每个元素应用给定函数的第一个可用的非空结果;找到结果时跳过进一步的搜索。
     *
     * <li> reduce: Accumulate each element.  The supplied reduction
     * function cannot rely on ordering (more formally, it should be
     * both associative and commutative).  There are five variants:
     *
     * 减少:积累每个元素。所提供的约简函数不能依赖于排序(更正式地说，它应该是结合的和交换的)。有五种变体:
     *
     * <ul>
     *
     * <li> Plain reductions. (There is not a form of this method for
     * (key, value) function arguments since there is no corresponding
     * return type.)</li>
     *
     * 普通的减少。(对于(key, value)函数参数没有这种方法的形式，因为没有相应的返回类型。)
     *
     * <li> Mapped reductions that accumulate the results of a given
     * function applied to each element.</li>
     *
     * 将给定函数的结果累积到每个元素上的映射约简。
     *
     * <li> Reductions to scalar doubles, longs, and ints, using a
     * given basis value.</li>
     *
     * 使用给定的基值将其缩减为标量双精度、长精度和整数。
     *
     * </ul>
     * </li>
     * </ul>
     *
     * <p>These bulk operations accept a {@code parallelismThreshold}
     * argument. Methods proceed sequentially if the current map size is
     * estimated to be less than the given threshold. Using a value of
     * {@code Long.MAX_VALUE} suppresses all parallelism.  Using a value
     * of {@code 1} results in maximal parallelism by partitioning into
     * enough subtasks to fully utilize the {@link
     * ForkJoinPool#commonPool()} that is used for all parallel
     * computations. Normally, you would initially choose one of these
     * extreme values, and then measure performance of using in-between
     * values that trade off overhead versus throughput.
     *
     * 这些批量操作接受一个{@code parallel elismthreshold}参数。如果当前映射大小估计小于给定阈值，则按顺序执行。
     * 使用值{@code Long。MAX_VALUE}抑制所有并行。使用{@code 1}的值将划分为足够多的子任务，
     * 从而充分利用用于所有并行计算的{@link ForkJoinPool#commonPool()}，从而获得最大的并行性。
     * 通常，您首先会选择这些极值中的一个，然后度量使用介于开销和吞吐量之间的值的性能.
     *
     * <p>The concurrency properties of bulk operations follow
     * from those of ConcurrentHashMap: Any non-null result returned
     * from {@code get(key)} and related access methods bears a
     * happens-before relation with the associated insertion or
     * update.  The result of any bulk operation reflects the
     * composition of these per-element relations (but is not
     * necessarily atomic with respect to the map as a whole unless it
     * is somehow known to be quiescent).  Conversely, because keys
     * and values in the map are never null, null serves as a reliable
     * atomic indicator of the current lack of any result.  To
     * maintain this property, null serves as an implicit basis for
     * all non-scalar reduction operations. For the double, long, and
     * int versions, the basis should be one that, when combined with
     * any other value, returns that other value (more formally, it
     * should be the identity element for the reduction). Most common
     * reductions have these properties; for example, computing a sum
     * with basis 0 or a minimum with basis MAX_VALUE.
     *
     * 批量操作的并发性属性遵循ConcurrentHashMap的并发性属性:
     * 从{@code get(key)}和相关的访问方法返回的任何非null结果都与相关的插入或更新保持事前关系。
     * 任何批量操作的结果都反映了这些每个元素之间关系的组成(但对于整个映射来说，并不一定是原子关系，除非知道它是静态的)。
     * 相反，由于映射中的键和值从来都不是null，所以null可以作为当前缺少任何结果的可靠原子指示器。
     * 要维护此属性，null作为所有非标量约简操作的隐式基。
     * 对于double、long和int版本，基应该是与任何其他值组合时返回该其他值的基(更正式地说，它应该是还原的恒等元素)。
     * 大多数常见的约简具有这些性质;例如，计算以0为基底的和或以MAX_VALUE为基底的最小值。
     *
     * <p>Search and transformation functions provided as arguments
     * should similarly return null to indicate the lack of any result
     * (in which case it is not used). In the case of mapped
     * reductions, this also enables transformations to serve as
     * filters, returning null (or, in the case of primitive
     * specializations, the identity basis) if the element should not
     * be combined. You can create compound transformations and
     * filterings by composing them yourself under this "null means
     * there is nothing there now" rule before using them in search or
     * reduce operations.
     *
     * 作为参数提供的搜索和转换函数应该类似地返回null，以指示缺少任何结果(在这种情况下不使用它)。
     * 在映射约简的情况下，这还允许转换充当过滤器，如果元素不应该组合，则返回null(或者，在原始专门化的情况下，返回标识基)。
     * 您可以通过自己在这个null下组合它们来创建复合转换和筛选，这意味着在搜索或reduce操作中使用它们之前没有任何“规则”。
     *
     * <p>Methods accepting and/or returning Entry arguments maintain
     * key-value associations. They may be useful for example when
     * finding the key for the greatest value. Note that "plain" Entry
     * arguments can be supplied using {@code new
     * AbstractMap.SimpleEntry(k,v)}.
     *
     * 接受和/或返回条目参数的方法维护键值关联。
     * 例如，当查找值最大的键时，它们可能很有用。
     * 注意，可以使用{@code new AbstractMap.SimpleEntry(k,v)}提供“普通”条目参数。
     *
     * <p>Bulk operations may complete abruptly, throwing an
     * exception encountered in the application of a supplied
     * function. Bear in mind when handling such exceptions that other
     * concurrently executing functions could also have thrown
     * exceptions, or would have done so if the first exception had
     * not occurred.
     *
     * 批量操作可能会突然完成，从而引发在所提供的函数的应用程序中遇到的异常。
     * 在处理此类异常时，请记住，其他并发执行的函数也可能抛出异常,或者，如果第一个异常没有发生，就会这样做。
     *
     * <p>Speedups for parallel compared to sequential forms are common
     * but not guaranteed.  Parallel operations involving brief functions
     * on small maps may execute more slowly than sequential forms if the
     * underlying work to parallelize the computation is more expensive
     * than the computation itself.  Similarly, parallelization may not
     * lead to much actual parallelism if all processors are busy
     * performing unrelated tasks.
     *
     * 与顺序形式相比，并行形式的加速比较常见，但不能保证。
     * 如果并行计算的底层工作比计算本身更昂贵，那么涉及小映射上的简短函数的并行操作执行起来可能比顺序形式慢。
     * 类似地，如果所有处理器都忙于执行不相关的任务，并行化可能不会导致太多实际的并行。
     *
     * <p>All arguments to all task methods must be non-null.
     *
     * 所有任务方法的所有参数必须是非空的。
     *
     * <p>This class is a member of the
     * <a href="{@docRoot}/../technotes/guides/collections/index.html">
     * Java Collections Framework</a>.
     *
     * 该类是Java集合框架的成员
     *
     * @since 1.5
     * @author Doug Lea
     * @param <K> the type of keys maintained by this map
     * @param <V> the type of mapped values
     */
```

### 综述：

```
       /*
         * Overview:
         *  综述:
         *
         *
         * The primary design goal of this hash table is to maintain
         * concurrent readability (typically method get(), but also
         * iterators and related methods) while minimizing update
         * contention. Secondary goals are to keep space consumption about
         * the same or better than java.util.HashMap, and to support high
         * initial insertion rates on an empty table by many threads.
         *
         * 这个哈希表的主要设计目标是维护并发可读性(通常是方法get()，也包括迭代器和相关方法)，同时最小化更新争用。
         * 次要目标是保持空间消耗与java.util相同或更好。并支持多个线程对空表的高初始插入率。
         *
         * This map usually acts as a binned (bucketed) hash table.  Each
         * key-value mapping is held in a Node.  Most nodes are instances
         * of the basic Node class with hash, key, value, and next
         * fields. However, various subclasses exist: TreeNodes are
         * arranged in balanced trees, not lists.  TreeBins hold the roots
         * of sets of TreeNodes. ForwardingNodes are placed at the heads
         * of bins during resizing. ReservationNodes are used as
         * placeholders while establishing values in computeIfAbsent and
         * related methods.  The types TreeBin, ForwardingNode, and
         * ReservationNode do not hold normal user keys, values, or
         * hashes, and are readily distinguishable during search etc
         * because they have negative hash fields and null key and value
         * fields. (These special nodes are either uncommon or transient,
         * so the impact of carrying around some unused fields is
         * insignificant.)
         *
         * 这个映射通常充当一个桶哈希表。每个键值映射都保存在一个节点中。
         * 大多数节点是具有散列、键、值和next字段的基本节点类的实例。
         * 然而，存在各种子类:树节点被安排在平衡的树中，而不是列表中。
         * TreeBins是拥有根节点的树节点。在调整大小时，ForwardingNodes被放置在箱子的顶部。
         * ReservationNodes用作占位符，同时在computeIfAbsent和相关方法中建立值。
         * TreeBin、forward节点和ReservationNode类型不包含普通的用户键、值或散列，
         * 在搜索过程中很容易区分，因为它们具有负散列字段和空键和值字段。
         * (这些特殊节点不是不常见就是短暂的，因此携带一些未使用的字段的影响是微不足道的。)
         *
         * The table is lazily initialized to a power-of-two size upon the
         * first insertion.  Each bin in the table normally contains a
         * list of Nodes (most often, the list has only zero or one Node).
         * Table accesses require volatile/atomic reads, writes, and
         * CASes.  Because there is no other way to arrange this without
         * adding further indirections, we use intrinsics
         * (sun.misc.Unsafe) operations.
         *
         * 第一次插入时，该表被惰性地初始化为2的幂大小。表中的每个桶通常包含一个节点列表(大多数情况下，列表只有0个或一个节点)。
         * 表访问需要volatile/atomic读写和CASes。
         * 因为没有其他方法可以在不添加进一步间接的情况下安排此操作，
         * 所以我们使用了(sun.misc.Unsafe)操作。
         *
         * We use the top (sign) bit of Node hash fields for control
         * purposes -- it is available anyway because of addressing
         * constraints.  Nodes with negative hash fields are specially
         * handled or ignored in map methods.
         *
         * 我们使用节点哈希字段的顶部(符号)位进行控制 --无论如何，它都是可用的，因为要处理约束。
         * 在map方法中，具有负哈希字段的节点被特殊处理或忽略。
         *
         * Insertion (via put or its variants) of the first node in an
         * empty bin is performed by just CASing it to the bin.  This is
         * by far the most common case for put operations under most
         * key/hash distributions.  Other update operations (insert,
         * delete, and replace) require locks.  We do not want to waste
         * the space required to associate a distinct lock object with
         * each bin, so instead use the first node of a bin list itself as
         * a lock. Locking support for these locks relies on builtin
         * "synchronized" monitors.
         * 将第一个节点插入到空容器中，只需通过CAS将其装箱即可。
         * 到目前为止，这是大多数键/散列分布下put操作最常见的情况。
         * 一些更新操作需要锁。我们不想浪费将一个不同的锁对象与每个bin关联所需的空间，所以应该使用bin列表本身的第一个节点作为锁。
         * 对这些锁的锁定支持依赖于内置的“synchronized”监视器。
         *
         * Using the first node of a list as a lock does not by itself
         * suffice though: When a node is locked, any update must first
         * validate that it is still the first node after locking it, and
         * retry if not. Because new nodes are always appended to lists,
         * once a node is first in a bin, it remains first until deleted
         * or the bin becomes invalidated (upon resizing).
         *
         * 使用列表的第一个节点作为锁本身还不够:当一个节点被锁定时，任何更新必须首先验证它仍然是锁定后的第一个节点，如果不是，则重试。
         * 因为新节点总是被附加到列表中，一旦一个节点在一个bin中是第一个节点，它就会保持在第一个，直到删除或bin失效(调整大小后)。
         *
         * The main disadvantage of per-bin locks is that other update
         * operations on other nodes in a bin list protected by the same
         * lock can stall, for example when user equals() or mapping
         * functions take a long time.  However, statistically, under
         * random hash codes, this is not a common problem.  Ideally, the
         * frequency of nodes in bins follows a Poisson distribution
         * (http://en.wikipedia.org/wiki/Poisson_distribution) with a
         * parameter of about 0.5 on average, given the resizing threshold
         * of 0.75, although with a large variance because of resizing
         * granularity. Ignoring variance, the expected occurrences of
         * list size k are (exp(-0.5) * pow(0.5, k) / factorial(k)). The
         * first values are:
         *
         * 0:    0.60653066
         * 1:    0.30326533
         * 2:    0.07581633
         * 3:    0.01263606
         * 4:    0.00157952
         * 5:    0.00015795
         * 6:    0.00001316
         * 7:    0.00000094
         * 8:    0.00000006
         * more: less than 1 in ten million
         *
         * per-bin锁的主要缺点是，受同一锁保护的bin列表中的其他节点上的其他更新操作可能会停止，例如当user equals()或映射函数花费很长时间时。
         * 然而，统计上，在随机哈希码下，这不是一个常见的问题。理想情况下，在给定调整大小阈值为0.75的情况下，
         * 箱中节点的频率遵循泊松分布(Poisson distribution)，其参数平均约为0.5，尽管由于调整粒度而存在较大的方差。
         * 忽略方差，列表大小k的预期出现次数为(exp(-0.5) * pow(0.5, k) / factorial(k))。
         * 第一个值是：
         * 0:    0.60653066
         * 1:    0.30326533
         * 2:    0.07581633
         * 3:    0.01263606
         * 4:    0.00157952
         * 5:    0.00015795
         * 6:    0.00001316
         * 7:    0.00000094
         * 8:    0.00000006
         * more: 少于千分之一
         *
         * Lock contention probability for two threads accessing distinct
         * elements is roughly 1 / (8 * #elements) under random hashes.
         * 在随机散列下，两个线程访问不同元素的锁争用概率大约为1 /(8 * #elements)。
         *
         * Actual hash code distributions encountered in practice
         * sometimes deviate significantly from uniform randomness.  This
         * includes the case when N > (1<<30), so some keys MUST collide.
         * Similarly for dumb or hostile usages in which multiple keys are
         * designed to have identical hash codes or ones that differs only
         * in masked-out high bits. So we use a secondary strategy that
         * applies when the number of nodes in a bin exceeds a
         * threshold. These TreeBins use a balanced tree to hold nodes (a
         * specialized form of red-black trees), bounding search time to
         * O(log N).  Each search step in a TreeBin is at least twice as
         * slow as in a regular list, but given that N cannot exceed
         * (1<<64) (before running out of addresses) this bounds search
         * steps, lock hold times, etc, to reasonable constants (roughly
         * 100 nodes inspected per operation worst case) so long as keys
         * are Comparable (which is very common -- String, Long, etc).
         * TreeBin nodes (TreeNodes) also maintain the same "next"
         * traversal pointers as regular nodes, so can be traversed in
         * iterators in the same way.
         *
         * 实际的哈希码分布在实践中有时会明显偏离均匀随机性。这包括当N >(1<<30)时，因此一些键必须碰撞。
         * 类似地，在一些愚蠢或恶意的用法中，多个键被设计为具有相同的哈希码，或者只有在屏蔽的高位上不同的哈希码。
         * 因此，当一个bin中的节点数量超过阈值时，我们使用一个辅助策略。
         * 这些TreeBins使用一个平衡树来保存节点(红黑树的一种特殊形式)，将搜索时间限制到O(log N)。
         * 每个搜索一步TreeBin至少两倍慢在一个常规列表,但是鉴于N不能超过(1 < < 64)（在地址用完之前）这个界限搜索步骤，锁定持有时间，等,
         * 合理的常量(大约100节点检查每个操作最坏的情况)只要键具有可比性(这是很常见的,String, Long,等等)。
         * TreeBin节点(TreeNodes)也像普通节点一样维护相同的“next”遍历指针，因此可以以相同的方式在迭代器中遍历。
         *
         * The table is resized when occupancy exceeds a percentage
         * threshold (nominally, 0.75, but see below).  Any thread
         * noticing an overfull bin may assist in resizing after the
         * initiating thread allocates and sets up the replacement array.
         * However, rather than stalling, these other threads may proceed
         * with insertions etc.  The use of TreeBins shields us from the
         * worst case effects of overfilling while resizes are in
         * progress.  Resizing proceeds by transferring bins, one by one,
         * from the table to the next table. However, threads claim small
         * blocks of indices to transfer (via field transferIndex) before
         * doing so, reducing contention.  A generation stamp in field
         * sizeCtl ensures that resizings do not overlap. Because we are
         * using power-of-two expansion, the elements from each bin must
         * either stay at same index, or move with a power of two
         * offset. We eliminate unnecessary node creation by catching
         * cases where old nodes can be reused because their next fields
         * won't change.  On average, only about one-sixth of them need
         * cloning when a table doubles. The nodes they replace will be
         * garbage collectable as soon as they are no longer referenced by
         * any reader thread that may be in the midst of concurrently
         * traversing table.  Upon transfer, the old table bin contains
         * only a special forwarding node (with hash field "MOVED") that
         * contains the next table as its key. On encountering a
         * forwarding node, access and update operations restart, using
         * the new table.
         *
         * 当占有率超过百分比阈值(通常为0.75，但看情况)时，将调整表的大小。
         * 任何线程注意到一个过满的bin都可以在初始化线程分配并设置替换数组之后帮助调整大小。
         * 但是，这些其他线程可能会继续执行插入等操作，而不是停止。
         * TreeBins的使用保护了我们从最坏的情况下，过量填充的影响，而调整的过程中。
         * 调整大小的方法是将箱子一个一个地从表转移到下一个表。
         * 然而，线程在这样做之前声明要传输一小块索引(通过字段transferIndex)，从而减少争用。
         * 字段sizeCtl中的生成戳记确保大小调整不会重叠。
         * 因为我们使用的是2的幂展开，所以每个bin中的元素必须保持相同的索引，或者以2的幂偏移量移动。
         * 我们通过捕获可以重用旧节点的情况来消除不必要的节点创建，因为它们的下一个字段不会更改。
         * 平均而言，当表翻倍时，只有大约六分之一的表需要克隆。
         * 它们替换的节点一旦不再被并发遍历表中的任何读线程引用，就会成为垃圾收集节点。
         * 在传输时，旧表bin只包含一个特殊的转发节点(散列字段“MOVED”)，该节点包含下一个表作为键。
         * 遇到转发节点时，使用新表重新启动访问和更新操作。
         *
         *
         * Each bin transfer requires its bin lock, which can stall
         * waiting for locks while resizing. However, because other
         * threads can join in and help resize rather than contend for
         * locks, average aggregate waits become shorter as resizing
         * progresses.  The transfer operation must also ensure that all
         * accessible bins in both the old and new table are usable by any
         * traversal.  This is arranged in part by proceeding from the
         * last bin (table.length - 1) up towards the first.  Upon seeing
         * a forwarding node, traversals (see class Traverser) arrange to
         * move to the new table without revisiting nodes.  To ensure that
         * no intervening nodes are skipped even when moved out of order,
         * a stack (see class TableStack) is created on first encounter of
         * a forwarding node during a traversal, to maintain its place if
         * later processing the current table. The need for these
         * save/restore mechanics is relatively rare, but when one
         * forwarding node is encountered, typically many more will be.
         * So Traversers use a simple caching scheme to avoid creating so
         * many new TableStack nodes. (Thanks to Peter Levart for
         * suggesting use of a stack here.)
         *
         * 每个bin传输都需要它的bin锁，它可以在调整大小时等待锁定。
         * 但是，由于其他线程可以加入并帮助调整大小，而不是争用锁，所以随着调整大小的进行，平均聚合等待时间会变得更短。
         * 传输操作还必须确保旧表和新表中所有可访问的箱子都可用于任何遍历。
         * 这部分是通过从最后一个bin(table.length - 1)向上到第一个。
         * 当看到一个转发节点时，遍历(参见类遍历器)会安排移动到新表，而不需要重新访问节点。
         * 为了确保即使在无序移动时也不会跳过中间节点，在遍历过程中第一次遇到转发节点时会创建一个堆栈(请参见类TableStack)，
         * 以便在以后处理当前表时维护其位置。对这些保存/恢复机制的需求相对较少，但当遇到一个转发节点时，通常会遇到更多。
         * 因此遍历器使用一个简单的缓存方案来避免创建如此多的新表栈节点。(感谢Peter Levart建议在这里使用堆栈。)
         *
         *
         * The traversal scheme also applies to partial traversals of
         * ranges of bins (via an alternate Traverser constructor)
         * to support partitioned aggregate operations.  Also, read-only
         * operations give up if ever forwarded to a null table, which
         * provides support for shutdown-style clearing, which is also not
         * currently implemented.
         *
         * 遍历方案还适用于桶范围的部分遍历(通过另一个遍历构造函数)，以支持分区的聚合操作。
         * 此外，如果将只读操作转发到空表，则将放弃只读操作，空表提供了对关闭样式清除的支持，而关闭样式清除目前也没有实现。
         *
         * Lazy table initialization minimizes footprint until first use,
         * and also avoids resizings when the first operation is from a
         * putAll, constructor with map argument, or deserialization.
         * These cases attempt to override the initial capacity settings,
         * but harmlessly fail to take effect in cases of races.
         *
         * 延迟表初始化在第一次使用之前将占用的空间最小化，并且当第一个操作来自putAll、
         * 带有map参数的构造函数或反序列化时，还可以避免调整大小。
         * 这些情况试图覆盖初始容量设置，但在种族的情况下没有造成伤害。
         *
         * The element count is maintained using a specialization of
         * LongAdder. We need to incorporate a specialization rather than
         * just use a LongAdder in order to access implicit
         * contention-sensing that leads to creation of multiple
         * CounterCells.  The counter mechanics avoid contention on
         * updates but can encounter cache thrashing if read too
         * frequently during concurrent access. To avoid reading so often,
         * resizing under contention is attempted only upon adding to a
         * bin already holding two or more nodes. Under uniform hash
         * distributions, the probability of this occurring at threshold
         * is around 13%, meaning that only about 1 in 8 puts check
         * threshold (and after resizing, many fewer do so).
         *
         * 元素计数使用LongAdder的专门化来维护。为了隐式访问，我们需要合并专门化，
         * 而不是仅仅使用LongAdder竞争感应，导致多个对抗细胞的产生。
         * 计数器机制可以避免更新上的争用，但是如果在并发访问期间读得太频繁，可能会遇到缓存抖动。
         * 为了避免频繁读取，仅在添加到已经包含两个或多个节点的bin时才尝试在争用下调整大小。
         * 在均匀哈希分布下，这种情况发生在阈值处的概率约为13%，这意味着只有大约八分之一的人设置了检查阈值(调整大小后，这样做的人更少)。
         *
         * TreeBins use a special form of comparison for search and
         * related operations (which is the main reason we cannot use
         * existing collections such as TreeMaps). TreeBins contain
         * Comparable elements, but may contain others, as well as
         * elements that are Comparable but not necessarily Comparable for
         * the same T, so we cannot invoke compareTo among them. To handle
         * this, the tree is ordered primarily by hash value, then by
         * Comparable.compareTo order if applicable.  On lookup at a node,
         * if elements are not comparable or compare as 0 then both left
         * and right children may need to be searched in the case of tied
         * hash values. (This corresponds to the full list search that
         * would be necessary if all elements were non-Comparable and had
         * tied hashes.) On insertion, to keep a total ordering (or as
         * close as is required here) across rebalancings, we compare
         * classes and identityHashCodes as tie-breakers. The red-black
         * balancing code is updated from pre-jdk-collections
         * (http://gee.cs.oswego.edu/dl/classes/collections/RBCell.java)
         * based in turn on Cormen, Leiserson, and Rivest "Introduction to
         * Algorithms" (CLR).
         *
         * TreeBins使用一种特殊的比较形式进行搜索和相关操作(这是我们不能使用现有集合(如TreeMaps)的主要原因)。
         * TreeBins包含可比较的元素，但也可能包含其他元素，以及对相同的T具有可比性但不一定具有可比性的元素，因此我们不能在它们之间调用compareTo。
         * 为了处理这个问题，树的顺序主要是按散列值排序，然后按Comparable.compareTo 排序，如果适用的话。
         * 在节点查找时，如果元素不可比较或比较为0，那么在绑定哈希值的情况下，可能需要搜索左子元素和右子元素。
         * (这对应于完整的列表搜索，如果所有元素都是不可比较的，并且已经绑定了散列，那么这将是必要的。)
         * 在插入时，为了保持跨重新平衡的总顺序(或尽可能接近这里的要求)，我们将类和identityHashCodes作为连接符进行比较。
         * 红黑平衡代码是根据Cormen、Leiserson和Rivest的“算法介绍”(CLR)，从pre-jdk-collections更新的。
         *
         *
         * TreeBins also require an additional locking mechanism.  While
         * list traversal is always possible by readers even during
         * updates, tree traversal is not, mainly because of tree-rotations
         * that may change the root node and/or its linkages.  TreeBins
         * include a simple read-write lock mechanism parasitic on the
         * main bin-synchronization strategy: Structural adjustments
         * associated with an insertion or removal are already bin-locked
         * (and so cannot conflict with other writers) but must wait for
         * ongoing readers to finish. Since there can be only one such
         * waiter, we use a simple scheme using a single "waiter" field to
         * block writers.  However, readers need never block.  If the root
         * lock is held, they proceed along the slow traversal path (via
         * next-pointers) until the lock becomes available or the list is
         * exhausted, whichever comes first. These cases are not fast, but
         * maximize aggregate expected throughput.
         *
         * TreeBins还需要一个额外的锁定机制。虽然即使在更新期间，读取器也始终可以执行列表遍历，
         * 但是树遍历不行，这主要是因为树的旋转可能会更改根节点和/或其链接。
         * TreeBins包含一个简单的读写锁机制，寄生在主要的bin同步策略上:
         * 与插入或删除相关的结构调整已经锁定(因此不能与其他作者发生冲突)，但必须等待正在进行的读者完成。
         * 由于只能有一个这样的waiter，所以我们使用一个简单的方案，使用一个“waiter”字段来阻止编写器。
         * 然而，读者永远不需要阻塞。如果持有根锁，它们将沿着缓慢的遍历路径(通过下一个指针)进行，直到锁可用或列表耗尽，无论哪个先出现。
         * 这些情况并不快，但是可以最大限度地提高总预期吞吐量。
         *
         * Maintaining API and serialization compatibility with previous
         * versions of this class introduces several oddities. Mainly: We
         * leave untouched but unused constructor arguments refering to
         * concurrencyLevel. We accept a loadFactor constructor argument,
         * but apply it only to initial table capacity (which is the only
         * time that we can guarantee to honor it.) We also declare an
         * unused "Segment" class that is instantiated in minimal form
         * only when serializing.
         *
         * 维护这个类以前版本的API和序列化兼容性会带来一些奇怪的现象。主要：
         * 我们保留未修改但未使用的构造函数参数来引用concurrencyLevel。
         * 我们接受loadFactor构造函数参数，但只将其应用于初始表容量(这是我们能够保证遵守它的惟一时间)。
         * 我们还声明了一个未使用的“Segment”类，该类仅在序列化时以最小形式实例化。
         *
         * Also, solely for compatibility with previous versions of this
         * class, it extends AbstractMap, even though all of its methods
         * are overridden, so it is just useless baggage.
         *
         * 而且，仅为了与该类的以前版本兼容，它扩展了AbstractMap，即使覆盖了它的所有方法，所以它只是无用的包袱。
         *
         * This file is organized to make things a little easier to follow
         * while reading than they might otherwise: First the main static
         * declarations and utilities, then fields, then main public
         * methods (with a few factorings of multiple public methods into
         * internal ones), then sizing methods, trees, traversers, and
         * bulk operations.
         *
         * 这个文件的组织是为了让阅读的时候更容易理解:
         * 首先是主静态声明和实用程序，然后是字段，然后是主公共方法(将多个公共方法分解为内部方法)，然后是调整方法大小、树、遍历器和批量操作。
         */
```