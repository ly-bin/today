##### 1、题目：https://leetcode-cn.com/problems/lru-cache/     中等 
##### 2、思路：
> 1、借助于java的LinkHashMap是最好事实现的，它已经实现了最近最久未使用的算法，每次put的时候会将当前元素放在链表的尾部（tail），每次访问一个元素的时候也会将该元素放在链表的尾部，
> 我们只需要实现根据元素数量判断是否要删除元素即可，明日时不删除的，即无限的缓存。
> 实际的代码分析如下
```
// 获取元素时
java.util.LinkedHashMap#get
public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        // accessOrder 是关键，为true的时候会按照访问频次来排序
        if (accessOrder)
            // 处理已有元素的链表顺序，当前访问的调整到最前面
            afterNodeAccess(e);
        return e.value;
    }
    
// 实际移动已存在的元素
java.util.LinkedHashMap#afterNodeAccess
 void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            //最终将此次访问的元素调整到最近的位置
            tail = p;
            ++modCount;
        }
    }
// 添加元素时
java.util.HashMap#put
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    // 当旧kay和新key不是同一个key时，直接新增一个节点，并放在链表最后，此时e为空，预示着下文的
                    // afterNodeAccess方法不执行，
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 添加完新元素再进行校验是否达到链表树化的阈值
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            // 发生了hash冲突，新加的元素必定在链表或者红黑树上且新元素的key和旧元素的key是一个，替换key的旧值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                // 需要调整最新的元素的顺序，和get时是同一个方法，都是有HashMap的子类LinkHashMap实现的
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        // 后置方法，在LinkHashMap中实现
        afterNodeInsertion(evict);
        return null;
    }
java.util.LinkedHashMap#afterNodeInsertion
// 根据条件删除元素 removeEldestEntry是需要我们自己实现逻辑的
void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
```
##### 3、代码
```
public class LRUCache {
    private static LinkedHashMap<Integer, Integer> map = null;
    private int MAX_ENTRIES = 0;

    public LRUCache(int capacity) {
        float loadFactor = 0.75f;
        MAX_ENTRIES = capacity;
        map = new LinkedHashMap(capacity,loadFactor,true){
            protected boolean removeEldestEntry(Map.Entry eldest) {
                return size() > MAX_ENTRIES;
            }
        };
    }

    public int get(int key) {
        Integer v = map.get(key);
        if (v==null){
            return -1;
        }
        return v;
    }

    public void put(int key, int value) {
        map.put(key,value);
    }

    public static void main(String[] args) {
        LRUCache lRUCache = new LRUCache(2);
        lRUCache.put(1, 1); // 缓存是 {1=1}
        lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
        System.out.println(lRUCache.get(1));    // 返回 1
        lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
        System.out.println(lRUCache.get(2));    // 返回 -1 (未找到)
        lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
        System.out.println(lRUCache.get(1));    // 返回 -1 (未找到)
        System.out.println(lRUCache.get(3));    // 返回 3
        System.out.println(lRUCache.get(4));    // 返回 4
    }
}

```
