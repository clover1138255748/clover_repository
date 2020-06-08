基于JDK1.8的CopyOnWriteArrayList的简单翻译：

```
    /**
     * A thread-safe variant of {@link java.util.ArrayList} in which all mutative
     * operations ({@code add}, {@code set}, and so on) are implemented by
     * making a fresh copy of the underlying array.
     * <p>
     * ArrayList的线程安全的变体，所有的可变操作如add、set等都是通过数组复制实现的。
     *
     * <p>This is ordinarily too costly, but may be <em>more</em> efficient
     * than alternatives when traversal operations vastly outnumber
     * mutations, and is useful when you cannot or don't want to
     * synchronize traversals, yet need to preclude interference among
     * concurrent threads.  The "snapshot" style iterator method uses a
     * reference to the state of the array at the point that the iterator
     * was created. This array never changes during the lifetime of the
     * iterator, so interference is impossible and the iterator is
     * guaranteed not to throw {@code ConcurrentModificationException}.
     * The iterator will not reflect additions, removals, or changes to
     * the list since the iterator was created.  Element-changing
     * operations on iterators themselves ({@code remove}, {@code set}, and
     * {@code add}) are not supported. These methods throw
     * {@code UnsupportedOperationException}.
     * <p>
     * 这通常开销太大，但是当遍历操作的数量远远超过突变时，它可能比替代方法更有效;
     * 当您不能或不想同步遍历，但又需要排除并发线程之间的干扰时，它很有用。
     * “快照”样式迭代器方法使用对创建迭代器时数组状态的引用。
     * 该数组在迭代器的生命周期内从未更改，因此不可能发生干扰，
     * 并且保证迭代器不会抛出{@code ConcurrentModificationException}。
     * 自创建迭代器以来，迭代器不会反映列表的添加、删除或更改。
     * 不支持对迭代器本身({@code remove}、{@code set}和{@code add})进行元素更改操作。
     * 这些方法会抛出{@code UnsupportedOperationException}。
     *
     *
     * <p>All elements are permitted, including {@code null}.
     * <p>
     * 所有元素都被允许，包括null。
     *
     * <p>Memory consistency effects: As with other concurrent
     * collections, actions in a thread prior to placing an object into a
     * {@code CopyOnWriteArrayList}
     * <a href="package-summary.html#MemoryVisibility"><i>happen-before</i></a>
     * actions subsequent to the access or removal of that element from
     * the {@code CopyOnWriteArrayList} in another thread.
     * <p>
     * 内存一致性效果：与其他并发集合一样，在将对象放入{@code CopyOnWriteArrayList}之前的线程中的操作
     * 发生在从另一个线程的{@code CopyOnWriteArrayList}中访问或删除该元素之后的操作之前。
     *
     * <p>This class is a member of the
     * <a href="{@docRoot}/../technotes/guides/collections/index.html">
     * Java Collections Framework</a>.
     *
     * @param <E> the type of elements held in this collection
     * @author Doug Lea
     * @since 1.5
     */
    public class CopyOnWriteArrayList<E>
            implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
        private static final long serialVersionUID = 8673264195747942595L;

        /**
         * The lock protecting all mutators
         * 重入锁
         */
        final transient ReentrantLock lock = new ReentrantLock();

        /**
         * The array, accessed only via getArray/setArray.
         * 只能通过 getArray/setArray 访问的数组
         */
        private transient volatile Object[] array;

        /**
         * Gets the array.  Non-private so as to also be accessible
         * from CopyOnWriteArraySet class.
         * 获取数组，非私有方法以便于CopyOnWriteArraySet类的访问
         */
        final Object[] getArray() {
            return array;
        }

        /**
         * Sets the array.
         * 设置数组
         */
        final void setArray(Object[] a) {
            array = a;
        }

        /**
         * Creates an empty list.
         * 创建一个空数组
         */
        public CopyOnWriteArrayList() {
            setArray(new Object[0]);
        }

        /**
         * Creates a list containing the elements of the specified
         * collection, in the order they are returned by the collection's
         * iterator.
         * 创建一个包含指定集合元素的列表，按照指定集合的迭代器返回的顺序。
         *
         * @param c the collection of initially held elements
         * @throws NullPointerException if the specified collection is null
         */
        public CopyOnWriteArrayList(Collection<? extends E> c) {
            Object[] elements;
            //如果c的类类型为CopyOnWriteArrayList
            if (c.getClass() == CopyOnWriteArrayList.class)
                //直接获取其数组
                elements = ((CopyOnWriteArrayList<?>) c).getArray();

                //如果c的类类型不为CopyOnWriteArrayList
            else {
                //通过toArray转数组，
                elements = c.toArray();
                // c.toArray might (incorrectly) not return Object[] (see 6260652)
                //如果c.toArray返回的不是 Object[]类型，则通过数组拷贝
                if (elements.getClass() != Object[].class)
                    elements = Arrays.copyOf(elements, elements.length, Object[].class);
            }
            //设置数组
            setArray(elements);
        }

        /**
         * Creates a list holding a copy of the given array.
         * 创建包含给定数组副本的列表。
         *
         * @param toCopyIn the array (a copy of this array is used as the
         *                 internal array)
         * @throws NullPointerException if the specified array is null
         */
        public CopyOnWriteArrayList(E[] toCopyIn) {
            setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
        }

        /**
         * Returns the number of elements in this list.
         * 获取元素的数量
         *
         * @return the number of elements in this list
         */
        public int size() {
            return getArray().length;
        }

        /**
         * Returns {@code true} if this list contains no elements.
         * 判断列表元素是否为空
         *
         * @return {@code true} if this list contains no elements
         */
        public boolean isEmpty() {
            return size() == 0;
        }

        /**
         * Tests for equality, coping with nulls.
         * 判断o1 o2是否相等
         */
        private static boolean eq(Object o1, Object o2) {
            return (o1 == null) ? o2 == null : o1.equals(o2);
        }

        /**
         * static version of indexOf, to allow repeated calls without
         * needing to re-acquire array each time.
         * 静态版本的indexOf，允许重复调用，而不需要每次重新获取数组。
         *
         * @param o        element to search for 要查找的元素
         * @param elements the array 数组
         * @param index    first index to search 查询开始位置
         * @param fence    one past last index to search 要搜索的最后一个索引
         * @return index of element, or -1 if absent 元素的索引，-1代表不存在
         */
        private static int indexOf(Object o, Object[] elements,
                                   int index, int fence) {
            //如果要查找的元素为null
            if (o == null) {
                //迭代数组
                for (int i = index; i < fence; i++)
                    //当前位置为null则返回当前索引
                    if (elements[i] == null)
                        return i;
            } else {
                //同上
                for (int i = index; i < fence; i++)
                    if (o.equals(elements[i]))
                        return i;
            }
            return -1;
        }

        /**
         * static version of lastIndexOf.
         * 静态版本的lastIndexOf
         *
         * @param o        element to search for 要查找的元素
         * @param elements the array 数组
         * @param index    first index to search 开始查找的索引
         * @return index of element, or -1 if absent 返回元素的索引或者-1（代表不存在）
         */
        private static int lastIndexOf(Object o, Object[] elements, int index) {
            if (o == null) {
                //倒叙查找
                for (int i = index; i >= 0; i--)
                    if (elements[i] == null)
                        return i;
            } else {
                for (int i = index; i >= 0; i--)
                    if (o.equals(elements[i]))
                        return i;
            }
            return -1;
        }

        /**
         * Returns {@code true} if this list contains the specified element.
         * More formally, returns {@code true} if and only if this list contains
         * at least one element {@code e} such that
         * <tt>(o==null ? e==null : o.equals(e))</tt>.
         * 查看是否存在要查找的元素
         *
         * @param o element whose presence in this list is to be tested
         * @return {@code true} if this list contains the specified element
         */
        public boolean contains(Object o) {
            Object[] elements = getArray();
            return indexOf(o, elements, 0, elements.length) >= 0;
        }

        /**
         * Returns a shallow copy of this list.  (The elements themselves
         * are not copied.)
         * 浅拷贝
         *
         * @return a clone of this list
         */
        public Object clone() {
            try {
                @SuppressWarnings("unchecked")
                CopyOnWriteArrayList<E> clone =
                        (CopyOnWriteArrayList<E>) super.clone();
                //重置锁定
                clone.resetLock();
                return clone;
            } catch (CloneNotSupportedException e) {
                // this shouldn't happen, since we are Cloneable
                throw new InternalError();
            }
        }

        /**
         * Returns an array containing all of the elements in this list
         * in proper sequence (from first to last element).
         * 返回原数组的拷贝
         *
         * <p>The returned array will be "safe" in that no references to it are
         * maintained by this list.  (In other words, this method must allocate
         * a new array).  The caller is thus free to modify the returned array.
         *
         * <p>This method acts as bridge between array-based and collection-based
         * APIs.
         *
         * @return an array containing all the elements in this list
         */
        public Object[] toArray() {
            Object[] elements = getArray();
            return Arrays.copyOf(elements, elements.length);
        }

        /**
         * {@inheritDoc}
         * 获取原数组中元素
         *
         * @throws IndexOutOfBoundsException {@inheritDoc}
         */
        public E get(int index) {
            return get(getArray(), index);
        }

        /**
         * Replaces the element at the specified position in this list with the
         * specified element.
         * 用指定的元素替换列表中指定位置的元素。
         *
         * @throws IndexOutOfBoundsException {@inheritDoc}
         */
        public E set(int index, E element) {
            //获取当前锁
            final ReentrantLock lock = this.lock;
            //锁定
            lock.lock();
            try {
                //获取原数组
                Object[] elements = getArray();
                //获取老的值
                E oldValue = get(elements, index);

                //如果老的值与给定的值不相等
                if (oldValue != element) {
                    int len = elements.length;
                    //原数组拷贝
                    Object[] newElements = Arrays.copyOf(elements, len);
                    //将新数组中的索引位置修改为新值
                    newElements[index] = element;
                    //将原数组替换为新数组
                    setArray(newElements);
                } else {
                    // 如果老的值与给定的值相等
                    // Not quite a no-op; ensures volatile write semantics
                    // 不完全是无操作;确保易失性写语义
                    setArray(elements);
                }
                //返回老的值
                return oldValue;
            } finally {
                //释放锁
                lock.unlock();
            }
        }

        /**
         * Appends the specified element to the end of this list.
         * 将指定的元素追加到此列表的末尾。
         *
         * @param e element to be appended to this list 要附加到此列表中的元素
         * @return {@code true} (as specified by {@link Collection#add})
         */
        public boolean add(E e) {
            //获取重入锁
            final ReentrantLock lock = this.lock;
            //锁定
            lock.lock();
            try {
                //获取原数组
                Object[] elements = getArray();
                int len = elements.length;
                //原数组拷贝 并增加一个空位
                Object[] newElements = Arrays.copyOf(elements, len + 1);
                //将指定元素增加到新数组新增的空位中
                newElements[len] = e;
                //新数组替换原数组
                setArray(newElements);
                //返回成功
                return true;
            } finally {
                lock.unlock();
            }
        }

        /**
         * Inserts the specified element at the specified position in this
         * list. Shifts the element currently at that position (if any) and
         * any subsequent elements to the right (adds one to their indices).
         * <p>
         * 将指定元素插入到列表中的指定位置。
         * 将当前位于该位置的元素(如果有的话)和随后的任何元素向右移动(将一个元素添加到它们的索引中)。
         *
         * @throws IndexOutOfBoundsException {@inheritDoc}
         */
        public void add(int index, E element) {
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                Object[] elements = getArray();
                int len = elements.length;
                if (index > len || index < 0)
                    throw new IndexOutOfBoundsException("Index: " + index +
                            ", Size: " + len);
                Object[] newElements;
                int numMoved = len - index;
                if (numMoved == 0)
                    newElements = Arrays.copyOf(elements, len + 1);
                else {
                    newElements = new Object[len + 1];
                    System.arraycopy(elements, 0, newElements, 0, index);
                    System.arraycopy(elements, index, newElements, index + 1,
                            numMoved);
                }
                newElements[index] = element;
                setArray(newElements);
            } finally {
                lock.unlock();
            }
        }

        /**
         * Removes the element at the specified position in this list.
         * Shifts any subsequent elements to the left (subtracts one from their
         * indices).  Returns the element that was removed from the list.
         * <p>
         * 移除列表中指定位置的元素。
         * 将任何后续元素向左移动(从它们的索引中减去一个)。
         * 返回从列表中删除的元素。
         *
         * @throws IndexOutOfBoundsException {@inheritDoc}
         */
        public E remove(int index) {
            //获取锁并且锁定
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                //获取原数组
                Object[] elements = getArray();
                int len = elements.length;
                //获取要删除的元素值
                E oldValue = get(elements, index);
                //要移动的值
                int numMoved = len - index - 1;
                //如果为0，则删除的是最后一个元素
                if (numMoved == 0)
                    //数组拷贝
                    setArray(Arrays.copyOf(elements, len - 1));
                else {
                    //新建数组
                    Object[] newElements = new Object[len - 1];
                    //将原数组中，索引index之前的所有数据，拷贝到新数组中
                    System.arraycopy(elements, 0, newElements, 0, index);
                    //将元素组，索引index+1 之后的numMoved个元素，复制到新数组，索引index之后
                    System.arraycopy(elements, index + 1, newElements, index,
                            numMoved);
                    //替换原数组
                    setArray(newElements);
                }
                //返回老的值
                return oldValue;
            } finally {
                //释放锁
                lock.unlock();
            }
        }

        /**
         * Removes all of the elements from this list.
         * The list will be empty after this call returns.
         * 清除数组
         */
        public void clear() {
            //获取锁并锁定
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                //将原数组用空数组代替
                setArray(new Object[0]);
            } finally {
                //释放锁
                lock.unlock();
            }
        }

        /**
         * Returns a view of the portion of this list between
         * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.
         * The returned list is backed by this list, so changes in the
         * returned list are reflected in this list.
         * <p>
         * 返回列表中包含的{@code fromIndex}和排除的{@code toIndex}之间部分的视图。
         * 返回的列表由该列表支持，因此返回列表中的更改将反映在该列表中。
         *
         * <p>The semantics of the list returned by this method become
         * undefined if the backing list (i.e., this list) is modified in
         * any way other than via the returned list.
         * <p>
         * 如果以非通过返回列表的任何方式修改支持列表，则此方法返回的列表的语义将成为未定义的。
         *
         * @param fromIndex low endpoint (inclusive) of the subList
         * @param toIndex   high endpoint (exclusive) of the subList
         * @return a view of the specified range within this list
         * @throws IndexOutOfBoundsException {@inheritDoc}
         */
        public List<E> subList(int fromIndex, int toIndex) {
            //锁定
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                //获取原数组
                Object[] elements = getArray();
                int len = elements.length;
                //校验
                if (fromIndex < 0 || toIndex > len || fromIndex > toIndex)
                    throw new IndexOutOfBoundsException();
                //返回COWSubList子类
                return new COWSubList<E>(this, fromIndex, toIndex);
            } finally {
                lock.unlock();
            }
        }

        /**
         * 操作的还是原数组，只是提供了视图
         * Sublist for CopyOnWriteArrayList.
         * This class extends AbstractList merely for convenience, to
         * avoid having to define addAll, etc. This doesn't hurt, but
         * is wasteful.  This class does not need or use modCount
         * mechanics in AbstractList, but does need to check for
         * concurrent modification using similar mechanics.  On each
         * operation, the array that we expect the backing list to use
         * is checked and updated.  Since we do this for all of the
         * base operations invoked by those defined in AbstractList,
         * all is well.  While inefficient, this is not worth
         * improving.  The kinds of list operations inherited from
         * AbstractList are already so slow on COW sublists that
         * adding a bit more space/time doesn't seem even noticeable.
         */
        private static class COWSubList<E>
                extends AbstractList<E>
                implements RandomAccess {
            private final CopyOnWriteArrayList<E> l;
            private final int offset;
            private int size;
            private Object[] expectedArray;

            // only call this holding l's lock
            COWSubList(CopyOnWriteArrayList<E> list,
                       int fromIndex, int toIndex) {
                l = list;
                expectedArray = l.getArray();
                offset = fromIndex;
                size = toIndex - fromIndex;
            }

            // only call this holding l's lock
            private void checkForComodification() {
                if (l.getArray() != expectedArray)
                    throw new ConcurrentModificationException();
            }

            // only call this holding l's lock
            private void rangeCheck(int index) {
                if (index < 0 || index >= size)
                    throw new IndexOutOfBoundsException("Index: " + index +
                            ",Size: " + size);
            }

            public E set(int index, E element) {
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    rangeCheck(index);
                    checkForComodification();
                    E x = l.set(index + offset, element);
                    expectedArray = l.getArray();
                    return x;
                } finally {
                    lock.unlock();
                }
            }

            public E get(int index) {
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    rangeCheck(index);
                    checkForComodification();
                    return l.get(index + offset);
                } finally {
                    lock.unlock();
                }
            }

            public int size() {
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    checkForComodification();
                    return size;
                } finally {
                    lock.unlock();
                }
            }

            public void add(int index, E element) {
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    checkForComodification();
                    if (index < 0 || index > size)
                        throw new IndexOutOfBoundsException();
                    l.add(index + offset, element);
                    expectedArray = l.getArray();
                    size++;
                } finally {
                    lock.unlock();
                }
            }

            public void clear() {
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    checkForComodification();
                    l.removeRange(offset, offset + size);
                    expectedArray = l.getArray();
                    size = 0;
                } finally {
                    lock.unlock();
                }
            }

            public E remove(int index) {
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    rangeCheck(index);
                    checkForComodification();
                    E result = l.remove(index + offset);
                    expectedArray = l.getArray();
                    size--;
                    return result;
                } finally {
                    lock.unlock();
                }
            }

            public boolean remove(Object o) {
                int index = indexOf(o);
                if (index == -1)
                    return false;
                remove(index);
                return true;
            }

            public Iterator<E> iterator() {
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    checkForComodification();
                    return new COWSubListIterator<E>(l, 0, offset, size);
                } finally {
                    lock.unlock();
                }
            }

            public ListIterator<E> listIterator(int index) {
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    checkForComodification();
                    if (index < 0 || index > size)
                        throw new IndexOutOfBoundsException("Index: " + index +
                                ", Size: " + size);
                    return new COWSubListIterator<E>(l, index, offset, size);
                } finally {
                    lock.unlock();
                }
            }

            public List<E> subList(int fromIndex, int toIndex) {
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    checkForComodification();
                    if (fromIndex < 0 || toIndex > size || fromIndex > toIndex)
                        throw new IndexOutOfBoundsException();
                    return new COWSubList<E>(l, fromIndex + offset,
                            toIndex + offset);
                } finally {
                    lock.unlock();
                }
            }

            public void forEach(Consumer<? super E> action) {
                if (action == null) throw new NullPointerException();
                int lo = offset;
                int hi = offset + size;
                Object[] a = expectedArray;
                if (l.getArray() != a)
                    throw new ConcurrentModificationException();
                if (lo < 0 || hi > a.length)
                    throw new IndexOutOfBoundsException();
                for (int i = lo; i < hi; ++i) {
                    @SuppressWarnings("unchecked") E e = (E) a[i];
                    action.accept(e);
                }
            }

            public void replaceAll(UnaryOperator<E> operator) {
                if (operator == null) throw new NullPointerException();
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    int lo = offset;
                    int hi = offset + size;
                    Object[] elements = expectedArray;
                    if (l.getArray() != elements)
                        throw new ConcurrentModificationException();
                    int len = elements.length;
                    if (lo < 0 || hi > len)
                        throw new IndexOutOfBoundsException();
                    Object[] newElements = Arrays.copyOf(elements, len);
                    for (int i = lo; i < hi; ++i) {
                        @SuppressWarnings("unchecked") E e = (E) elements[i];
                        newElements[i] = operator.apply(e);
                    }
                    l.setArray(expectedArray = newElements);
                } finally {
                    lock.unlock();
                }
            }

            public void sort(Comparator<? super E> c) {
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    int lo = offset;
                    int hi = offset + size;
                    Object[] elements = expectedArray;
                    if (l.getArray() != elements)
                        throw new ConcurrentModificationException();
                    int len = elements.length;
                    if (lo < 0 || hi > len)
                        throw new IndexOutOfBoundsException();
                    Object[] newElements = Arrays.copyOf(elements, len);
                    @SuppressWarnings("unchecked") E[] es = (E[]) newElements;
                    Arrays.sort(es, lo, hi, c);
                    l.setArray(expectedArray = newElements);
                } finally {
                    lock.unlock();
                }
            }

            public boolean removeAll(Collection<?> c) {
                if (c == null) throw new NullPointerException();
                boolean removed = false;
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    int n = size;
                    if (n > 0) {
                        int lo = offset;
                        int hi = offset + n;
                        Object[] elements = expectedArray;
                        if (l.getArray() != elements)
                            throw new ConcurrentModificationException();
                        int len = elements.length;
                        if (lo < 0 || hi > len)
                            throw new IndexOutOfBoundsException();
                        int newSize = 0;
                        Object[] temp = new Object[n];
                        for (int i = lo; i < hi; ++i) {
                            Object element = elements[i];
                            if (!c.contains(element))
                                temp[newSize++] = element;
                        }
                        if (newSize != n) {
                            Object[] newElements = new Object[len - n + newSize];
                            System.arraycopy(elements, 0, newElements, 0, lo);
                            System.arraycopy(temp, 0, newElements, lo, newSize);
                            System.arraycopy(elements, hi, newElements,
                                    lo + newSize, len - hi);
                            size = newSize;
                            removed = true;
                            l.setArray(expectedArray = newElements);
                        }
                    }
                } finally {
                    lock.unlock();
                }
                return removed;
            }

            public boolean retainAll(Collection<?> c) {
                if (c == null) throw new NullPointerException();
                boolean removed = false;
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    int n = size;
                    if (n > 0) {
                        int lo = offset;
                        int hi = offset + n;
                        Object[] elements = expectedArray;
                        if (l.getArray() != elements)
                            throw new ConcurrentModificationException();
                        int len = elements.length;
                        if (lo < 0 || hi > len)
                            throw new IndexOutOfBoundsException();
                        int newSize = 0;
                        Object[] temp = new Object[n];
                        for (int i = lo; i < hi; ++i) {
                            Object element = elements[i];
                            if (c.contains(element))
                                temp[newSize++] = element;
                        }
                        if (newSize != n) {
                            Object[] newElements = new Object[len - n + newSize];
                            System.arraycopy(elements, 0, newElements, 0, lo);
                            System.arraycopy(temp, 0, newElements, lo, newSize);
                            System.arraycopy(elements, hi, newElements,
                                    lo + newSize, len - hi);
                            size = newSize;
                            removed = true;
                            l.setArray(expectedArray = newElements);
                        }
                    }
                } finally {
                    lock.unlock();
                }
                return removed;
            }

            public boolean removeIf(Predicate<? super E> filter) {
                if (filter == null) throw new NullPointerException();
                boolean removed = false;
                final ReentrantLock lock = l.lock;
                lock.lock();
                try {
                    int n = size;
                    if (n > 0) {
                        int lo = offset;
                        int hi = offset + n;
                        Object[] elements = expectedArray;
                        if (l.getArray() != elements)
                            throw new ConcurrentModificationException();
                        int len = elements.length;
                        if (lo < 0 || hi > len)
                            throw new IndexOutOfBoundsException();
                        int newSize = 0;
                        Object[] temp = new Object[n];
                        for (int i = lo; i < hi; ++i) {
                            @SuppressWarnings("unchecked") E e = (E) elements[i];
                            if (!filter.test(e))
                                temp[newSize++] = e;
                        }
                        if (newSize != n) {
                            Object[] newElements = new Object[len - n + newSize];
                            System.arraycopy(elements, 0, newElements, 0, lo);
                            System.arraycopy(temp, 0, newElements, lo, newSize);
                            System.arraycopy(elements, hi, newElements,
                                    lo + newSize, len - hi);
                            size = newSize;
                            removed = true;
                            l.setArray(expectedArray = newElements);
                        }
                    }
                } finally {
                    lock.unlock();
                }
                return removed;
            }

            public Spliterator<E> spliterator() {
                int lo = offset;
                int hi = offset + size;
                Object[] a = expectedArray;
                if (l.getArray() != a)
                    throw new ConcurrentModificationException();
                if (lo < 0 || hi > a.length)
                    throw new IndexOutOfBoundsException();
                return Spliterators.spliterator
                        (a, lo, hi, Spliterator.IMMUTABLE | Spliterator.ORDERED);
            }

        }

        private static class COWSubListIterator<E> implements ListIterator<E> {
            private final ListIterator<E> it;
            private final int offset;
            private final int size;

            COWSubListIterator(List<E> l, int index, int offset, int size) {
                this.offset = offset;
                this.size = size;
                it = l.listIterator(index + offset);
            }

            public boolean hasNext() {
                return nextIndex() < size;
            }

            public E next() {
                if (hasNext())
                    return it.next();
                else
                    throw new NoSuchElementException();
            }

            public boolean hasPrevious() {
                return previousIndex() >= 0;
            }

            public E previous() {
                if (hasPrevious())
                    return it.previous();
                else
                    throw new NoSuchElementException();
            }

            public int nextIndex() {
                return it.nextIndex() - offset;
            }

            public int previousIndex() {
                return it.previousIndex() - offset;
            }

            public void remove() {
                throw new UnsupportedOperationException();
            }

            public void set(E e) {
                throw new UnsupportedOperationException();
            }

            public void add(E e) {
                throw new UnsupportedOperationException();
            }

            @Override
            public void forEachRemaining(Consumer<? super E> action) {
                Objects.requireNonNull(action);
                int s = size;
                ListIterator<E> i = it;
                while (nextIndex() < s) {
                    action.accept(i.next());
                }
            }
        }

        // Support for resetting lock while deserializing
        // 支持反序列化时重置锁定
        private void resetLock() {
            UNSAFE.putObjectVolatile(this, lockOffset, new ReentrantLock());
        }

        private static final sun.misc.Unsafe UNSAFE;
        private static final long lockOffset;

        static {
            try {
                UNSAFE = sun.misc.Unsafe.getUnsafe();
                Class<?> k = CopyOnWriteArrayList.class;
                lockOffset = UNSAFE.objectFieldOffset
                        (k.getDeclaredField("lock"));
            } catch (Exception e) {
                throw new Error(e);
            }
        }
    }
```