title: java8 HashMap详解
date: 2016-04-22 14:02:04
tags: [java 1.8,HashMap]
categories: [java]
---

# 总结
HashMap的原理：     
HashMap基于Hash算法，通过put(key,value)存储，get(key)来获取。当传入key时，HashMap会根据hash(K key)计算出hash值，根据hash值将value保存在数组里。  

当计算出的hash值相同时使用链表或者红黑树来解决Hash冲突，HashMap的做法是用链表和红黑树存储相同hash值的value。当Hash冲突的个数比较少时，使用链表，否则使用红黑树。这样做的好处是，最坏的情况下即所有的key都Hash冲突，采用链表的话查找时间为O(n),而采用红黑树为O(lgn)。  

java里面还有一些方法定义了没有实现，不知道会不会在1.9里面加入一些新功能

<!--more -->
# HashMap的构造
## 内部的变量
```java  
    //初始化容量
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    //最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    //装载因子，扩容条件：size >= 装载因子*容量的
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    //一个Node节点的数组，
    transient Node<K,V>[] table;
    //一个所有建值对的集合
    transient Set<Map.Entry<K,V>> entrySet;
    //当前大小
    transient int size;
    //被修改的次数
    transient int modCount;

   	//需要扩容是，扩容到threshold值
    int threshold;

    //扩容因子
    final float loadFactor;
    //当某一个index冲突因子达到这个值得时候，改为红黑树来存储冲突的节点
    static final int TREEIFY_THRESHOLD = 8;
```
## 三个构造方法
    ```java  
     public HashMap(int initialCapacity, float loadFactor)
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);//得到的threshold (是一个2的整数次幂-1) >=initialCapacity
    }
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
    ```
# HashMap 的存储实现

## 原理图
![hashMap原理图存储](/images/HashMap.jpg)
## hash算法
计算存储到哪一个table下标处
```java
 static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);//为什么这么做，不是很了解//TODO:
    }
```

## Node节点
链表节点，存储键值对，并含有一个next引用。
```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
    }
```
## 红黑树节点
红黑树节点的接口定义来自于LinkedHashMap
```java
 static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
    }
```

## put方法实现
```java
   //我们能够调用的put方法
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * 内部自己调用的put方法，
     *
     * @param hash hash值
     * @param key key值
     * @param value value值
     * @param onlyIfAbsent 如果是true，就不要改变原来已经有的值
     * @param evict 如果是false，标示table处于creation mode.TODO
     * @return 返回原来的值
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;//先判断了table是否没初始化，或者长度为0
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);//判断hash是否冲突，不冲突直接放下。冲突就else
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;//判断是不是要覆盖当前key
            else if (p instanceof TreeNode)//判断是不是采用红黑树存储的，TreeNode和Node之间是一种继承关系，HashMap.Node -->LinkedHashMap.Entry-->HashMap.TreeNode.
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);//插入值
            else {
                //不是用红黑树存储的，就遍历当前的链表查找插入位置或者覆盖已经存在的Key
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);//插入新节点
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);//冲突的个数特别高的时候，改为红黑树存储节点
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // 不是插入新节点，而是替换原来已经存在的值。
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        //判断是否是扩容
        if (++size > threshold)
            resize();
            //目前这个方法还是空方法，应该会在1.9里面实现一些新功能
        afterNodeInsertion(evict);
        return null;
    }
```

## get 方法实现
get方法比较简单，主要看如果是采用红黑树存储冲突的节点值的时候，怎么查找，这里有很多疑惑，在循环中多次判断hash值得大小，不应该是冲突的元素中的hash值都是一样的码，欢迎一起交流。
```java
 /**
         * Finds the node starting at root p with the given hash and key.
         * The kc argument caches comparableClassFor(key) upon first use
         * comparing keys.
         */
        final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
            TreeNode<K,V> p = this;
            do {
                int ph, dir; K pk;
                TreeNode<K,V> pl = p.left, pr = p.right, q;
                if ((ph = p.hash) > h)// 为什么要判断hash值。
                    p = pl;
                else if (ph < h)
                    p = pr;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    return p;//找到了
                else if (pl == null)
                    p = pr;
                else if (pr == null)
                    p = pl;
                else if ((kc != null ||
                          (kc = comparableClassFor(k)) != null) &&
                         (dir = compareComparables(kc, k, pk)) != 0)
                    p = (dir < 0) ? pl : pr;
                else if ((q = pr.find(h, k, kc)) != null)
                    return q;
                else
                    p = pl;
            } while (p != null);
            return null;
        }
```
