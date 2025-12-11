**节点内部属性：**

TreeMap中，每一个元素都是Entry对象：

```java
    static final class Entry<K,V> implements Map.Entry<K,V> {
        K key;
        V value;
        Entry<K,V> left;
        Entry<K,V> right;
        Entry<K,V> parent;
        boolean color = BLACK;
```



**一些成员变量：**

```java
    private final Comparator<? super K> comparator;

    private transient Entry<K,V> root;

    /**
     * The number of entries in the tree
     */
    private transient int size = 0;
```

comparator表示比较方法，root为根节点地址值，size为节点数量。



**空参构造：**

```java
    public TreeMap() {
        comparator = null;
    }
```



**带参构造：**

```java
 public TreeMap(Comparator<? super K> comparator) {
        this.comparator = comparator;
    }
```



**添加元素：**

```java
 public V put(K key, V value) {
        Entry<K,V> t = root;
        if (t == null) {
            compare(key, key); // type (and possibly null) check

            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // split comparator and comparable paths
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        Entry<K,V> e = new Entry<>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }

```

首先，获取根节点的地址值，赋值给局部变量t

如果t为null，表示当前是第一次添加，创建根节点，返回null，表示没有覆盖任何元素。

如果不为null，接着往下看。

cmp是两个元素的键比较之后的结果，负数表示当前要添加的元素较小，放在左子树。正数表示当前要添加的元素较大，放在右子树。

新建一个名为parent的Entry对象，表示要添加的节点的父节点。

将成员变量comparator赋值给cpr，表示比较规则。如果我们采取的是默认的自然排序，那么comparator为null，即cpr为null

如果有比较器对象，走if语句：

```java
        if (cpr != null) {
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
```



如果是自然排序，走else语句，由于结构类似，只分析else语句：

```java
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
```

可以看到这里将key强转为Comparable类型，因此键必须要实现Comparable接口

接着，将根节点当作当前节点的父节点。调用`compareTo`方法，比较当前节点与根节点键的大小关系。如果为负数，就去左子树找最合适的父节点，反之去右子树找最合适的父节点。如果相等，调用`setValue`方法：

```java
        public V setValue(V value) {
            V oldValue = this.value;
            this.value = value;
            return oldValue;
        }
```

可以看到这个方法实际上是做了一个覆盖，并返回被覆盖的值。

循环走这个过程，直到t为空。再来看看t为空之后又做了些什么：

```java
        Entry<K,V> e = new Entry<>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
```

由于在每一次do语句中都会把t赋值给parent，因此最终parent必然会是待添加节点最合适的父节点。创建一个Entry对象表示待添加的节点，然后根据cmp的大小决定它应该放父节点的左边还是右边，并建立连接。

我们知道这并不符合红黑树的规则，因此还要在`fixAfterInsertion`中进行调整。

```java
   private void fixAfterInsertion(Entry<K,V> x) {
        x.color = RED;

        while (x != null && x != root && x.parent.color == RED) {
            if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
                Entry<K,V> y = rightOf(parentOf(parentOf(x)));
                if (colorOf(y) == RED) {
                    setColor(parentOf(x), BLACK);
                    setColor(y, BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                } else {
                    if (x == rightOf(parentOf(x))) {
                        x = parentOf(x);
                        rotateLeft(x);
                    }
                    setColor(parentOf(x), BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    rotateRight(parentOf(parentOf(x)));
                }
            } else {
                Entry<K,V> y = leftOf(parentOf(parentOf(x)));
                if (colorOf(y) == RED) {
                    setColor(parentOf(x), BLACK);
                    setColor(y, BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                } else {
                    if (x == leftOf(parentOf(x))) {
                        x = parentOf(x);
                        rotateRight(x);
                    }
                    setColor(parentOf(x), BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    rotateLeft(parentOf(parentOf(x)));
                }
            }
        }
        root.color = BLACK;
    }
```

在该方法中，一上来就将节点的颜色调整为红色。毕竟红黑树中添加的节点默认都是红色的。

几个函数：

parentOf(x)：获取x的父节点

parentOf(parentOf(x))：获取x的爷爷节点

leftOf(x)：获取x的左子节点

rightOf(x)：获取x的右子节点

通过这四个函数，可以获得x的叔叔节点：leftOf(parentOf(parentOf(x)))或者rightOf(parentOf(parentOf(x)))



最后的几个问题：

:question:TreeMap添加元素的时候，是否需要重写`hashCode`与`equals`方法？

答：不需要。put中没有涉及到这两个方法。这两个方法是和HashMap有关的，见上一篇文章《JavaHashMap源码分析》

:question:既然有红黑树，HashMap的键是否需要实现Comparable接口或者传递比较器对象呢？

答：不需要。因为在HashMap的底层，默认是利用键的哈希值大小关系来创建红黑树的。

这里我们回到上一篇文章的内容，回到HashMap源码中的`putVal`方法：

```java
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
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
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
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

注意到这里：

```java
			else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
```

我们查看一下`putTreeVal`方法：

```java
        final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                                       int h, K k, V v) {
            Class<?> kc = null;
            boolean searched = false;
            TreeNode<K,V> root = (parent != null) ? root() : this;
            for (TreeNode<K,V> p = root;;) {
                int dir, ph; K pk;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    return p;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0) {
                    if (!searched) {
                        TreeNode<K,V> q, ch;
                        searched = true;
                        if (((ch = p.left) != null &&
                             (q = ch.find(h, k, kc)) != null) ||
                            ((ch = p.right) != null &&
                             (q = ch.find(h, k, kc)) != null))
                            return q;
                    }
                    dir = tieBreakOrder(k, pk);
                }

                TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    Node<K,V> xpn = xp.next;
                    TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    xp.next = x;
                    x.parent = x.prev = xp;
                    if (xpn != null)
                        ((TreeNode<K,V>)xpn).prev = x;
                    moveRootToFront(tab, balanceInsertion(root, x));
                    return null;
                }
            }
        }

```

进一步查看find方法：

```java
        final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
            TreeNode<K,V> p = this;
            do {
                int ph, dir; K pk;
                TreeNode<K,V> pl = p.left, pr = p.right, q;
                if ((ph = p.hash) > h)
                    p = pl;
                else if (ph < h)
                    p = pr;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    return p;
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

通过`(ph == p.hash) > h`等语句可以看出，的确是使用键的哈希值大小关系来创建红黑树。

:question:TreeMap和HashMap谁的效率更高？

答：如果是最坏情况，添加的8个元素全都形成了链表，那就是TreeMap的效率更高。前者的查询时间复杂度为O(n)，后者为O(logn)。但这种情况出现的概率很小，所以一般而言HashMap的效率更高。最优情况下其查询时间复杂度为O(1)

:question:三种双列集合如何选择？

答：默认情况下使用HashMap，一般情况下它的效率最高

如果要保证存取有序，用LinkedHashMap，因为它底层是双向链表

如果要进行排序，用TreeMap