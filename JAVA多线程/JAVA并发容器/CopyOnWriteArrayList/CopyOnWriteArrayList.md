### 定义

CopyOnWrite容器即写时复制的容器。
当我们往容器添加元素的时候，先将当前容器进行Copy，复制出一个新的容器，
然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。

盗图

![2019103010014\_1.png](https://gitee.com/cdx_dayshow/picBed/raw/master/img/2019103010014_1.png)COW.png

### 优缺点

#### 优点：

读操作性能很高，比较适用于读多写少的并发场景。
Java的list在遍历时，若中途有别的线程对list容器进行修改，则会抛出ConcurrentModificationException异常。
而CopyOnWriteArrayList由于其"读写分离"的思想，遍历和修改操作分别作用在不同的list容器，
所以在使用迭代器进行遍历时候，也就不会抛出ConcurrentModificationException异常。

#### 缺点：

- 内存占用问题，执行写操作时会发生数组拷贝
- 无法保证实时性，Vector对于读写操作均加锁同步，可以保证读和写的强一致性。
  而CopyOnWriteArrayList由于其实现策略的原因，写和读分别作用在新老不同容器上，在写操作执行过程中，读不会阻塞但读取到的却是老容器的数据。

### 使用场景

CopyOnWrite并发容器用于读多写少的并发场景。比如白名单，黑名单，商品类目的访问和更新场景。

### 源码分析

#### 重点

```
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
```

#### get

直接操作原数组，并发读时，可保证吞吐量。

```
     /**
         * {@inheritDoc}
         * 获取原数组中元素
         *
         * @throws IndexOutOfBoundsException {@inheritDoc}
         */
        public E get(int index) {
            return get(getArray(), index);
        }
```

#### add

发生一次数组拷贝。

```
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
```

#### set

最多一次数组拷贝。

```
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
```

#### remove

最多两次数组拷贝。

```
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
```