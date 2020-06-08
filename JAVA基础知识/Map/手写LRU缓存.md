### 1. 实现思路

目的是把最不常用的数据淘汰掉，所以需要记录一下每个元素的访问次数。最简单的方法就是把所有元素按使用情况排序，最近使用的，移到末尾。缓存满了，就从头部删除。

### 2. 使用哪种数据结构实现？

常用的数据结构有数组、链表、栈、队列，考虑到要从两端操作元素，就不能使用栈和队列。
 每次使用一个元素，都要把这个元素移到末尾，包含一次删除和一次添加操作，使用数组会有大量的拷贝操作，不适合。
 又考虑到删除一个元素，要把这个元素的前一个节点指向下一个节点，使用双链接最合适。
 链表不适合查询，因为每次都要遍历所有元素，可以和HashMap配合使用。
 **双链表 + HashMap**

### 3. 代码实现

```
import java.util.HashMap;
import java.util.Map;

/**
 * @author yideng
 */
public class LRUCache<K, V> {

    /**
     * 双链表的元素节点
     */
    private class Entry<K, V> {
        Entry<K, V> before;
        Entry<K, V> after;
        private K key;
        private V value;
    }

    /**
     * 缓存容量大小
     */
    private Integer capacity;
    /**
     * 头结点
     */
    private Entry<K, V> head;
    /**
     * 尾节点
     */
    private Entry<K, V> tail;
    /**
     * 用来存储所有元素
     */
    private Map<K, Entry<K, V>> caches = new HashMap<>();

    public LRUCache(int capacity) {
        this.capacity = capacity;
    }

    public V get(K key) {
        final Entry<K, V> node = caches.get(key);
        if (node != null) {
            // 有访问，就移到链表末尾
            afterNodeAccess(node);
            return node.value;
        }
        return null;
    }

    /**
     * 把该元素移到末尾
     */
    private void afterNodeAccess(Entry<K, V> e) {
        Entry<K, V> last = tail;
        // 如果e不是尾节点，才需要移动
        if (last != e) {
            // 删除该该节点与前一个节点的联系，判断是不是头结点
            if (e.before == null) {
                head = e.after;
            } else {
                e.before.after = e.after;
            }

            // 删除该该节点与后一个节点的联系
            if (e.after == null) {
                last = e.before;
            } else {
                e.after.before = e.before;
            }

            // 把该节点添加尾节点，判断尾节点是否为空
            if (last == null) {
                head = e;
            } else {
                e.before = last;
                last.after = e;
            }
            e.after = null;
            tail = e;
        }
    }

    public V put(K key, V value) {
        Entry<K, V> entry = caches.get(key);
        if (entry == null) {
            entry = new Entry<>();
            entry.key = key;
            entry.value = value;
            // 新节点添加到末尾
            linkNodeLast(entry);
            caches.put(key, entry);
            // 节点数大于容量，就删除头节点
            if (this.caches.size() > this.capacity) {
                this.caches.remove(head.key);
                afterNodeRemoval(head);
            }
            return null;
        }
        entry.value = value;
        // 节点有更新就移动到未节点
        afterNodeAccess(entry);
        caches.put(key, entry);
        return entry.value;
    }

    /**
     * 把该节点添加到尾节点
     */
    private void linkNodeLast(Entry<K, V> e) {
        final Entry<K, V> last = this.tail;
        if (head == null) {
            head = e;
        } else {
            e.before = last;
            last.after = e;
        }
        tail = e;
    }

    /**
     * 删除该节点
     */
    void afterNodeRemoval(Entry<K, V> e) {
        if (e.before == null) {
            head = e.after;
        } else {
            e.before.after = e.after;
        }

        if (e.after == null) {
            tail = e.before;
        } else {
            e.after.before = e.before;
        }
    }

}
复制代码
```

### 4. 其实还有更简单的实现

```
import java.util.LinkedHashMap;
import java.util.Map;

/**
 * @author yideng
 */
public class LRUCache<K, V> extends LinkedHashMap<K, V> {

    // 最大容量
    private final int maximumSize;

    public LRUCache(final int maximumSize) {
        // true代表按访问顺序排序，false代表按插入顺序
        super(maximumSize, 0.75f, true);
        this.maximumSize = maximumSize;
    }

    /**
     * 当节点数大于最大容量时，就删除最旧的元素
     */
    @Override
    protected boolean removeEldestEntry(final Map.Entry eldest) {
        return size() > this.maximumSize;
    }
}
复制代码
```

**为啥继承了LinkedHashMap，重写了两个方法，就实现了LRU？
 下篇带你手撕LinkedHashMap源码，到时你会发现LinkedHashMap的源码和上面一灯写的LRU逻辑竟然有惊人的相似。**

