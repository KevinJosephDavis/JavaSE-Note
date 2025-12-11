选中HashMap，按Ctrl+B，就会跳转到源码部分。

**节点：**

```java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
```

我们知道在HashMap中，每个元素都是一个Node对象，而它实现了Map里的Entry接口，所以也可以说每个元素都是一个Entry对象。这是因为HashMap在数组的基础上融入了链表与红黑树，所以每个元素都必须是节点。

一个Entry对象有4个字段：

1.hash：它是通过Key值计算出的哈希值

2.Key：键

3.Value：值

4.Next：作为链表中节点，它要知道如何找到下一个元素，因此通过Next记录下一个元素的地址值

那么红黑树呢？红黑树就不能只存储Next了，还必须要存储左右节点等其它信息。

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
```

可以看到，`TreeNode`继承了`LinkedHashMap`中的Entry，我们进一步看看这个Entry：

```java
 static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

可以看到这个Entry继承了HashMap中的Node，也就是一开始我们见到的Node，这也意味着`TreeNode`除了记录左右节点、父节点等这些专属于红黑树的信息，还保留了作为链表的信息。因为当链表长度达到8并且数组长度达到64，链表就要转为红黑树。



**成员变量**

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
static final int MAXIMUM_CAPACITY = 1 << 30;
static final float DEFAULT_LOAD_FACTOR = 0.75f;
transient Node<K,V>[] table;
```

这个table数组就是HashMap的存储基础。它的默认初始长度是16，最大容量是2的30次方，是个天文数字。默认扩容因子是0.75，这意味着当已经被填充的位置数量达到了容量的0.75就会进行扩容操作，每次扩容都会将容量扩充为原来的2倍。



**空参构造**

```java
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
```

可以看到，当我们调用HashMap的空参构造时，实际上只做了一件事：将默认扩容因子赋值给该对象的扩容因子成员变量。此时table还是NULL



**那么，table什么时候创建？**

当我们添加元素的时候table就必须要创建出来了，因此我们定位到put函数：

```java
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```

看一下hash方法：

```java
   static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

hash方法通过一些计算，返回了键的哈希值。可以看到，键的哈希值与Value没有任何关系，只和Key有关系。



再看`putVal`函数：

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

前三个参数分别为键的哈希值、键、值。第四个参数表示“当前的数据是否保留”，也就是覆盖与否的问题。

```java
Node<K,V>[] tab; Node<K,V> p; int n, i;
```

在`putVal`方法中又声明了一个tab数组。这是因为：成员变量在堆上，方法运行在栈上。如果运行在栈上的方法要多次去堆中找table，开销就会比较大。

另外，p是一个第三方变量，用来记录节点。n表示数组长度，i表示索引。



添加元素分为三种情况：

1.数组位置为null

2.数组位置不为null，键重复，覆盖

3.数组位置不为null，键不重复，形成链表或红黑树



**第一种情况：**

```java
 		if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
```

这里把table赋值给了tab，因为tab本身就是要在栈中充当table的角色。对于n同理，也把数组的长度赋值给了它。如果是第一次添加元素，那么这个条件就会成立，因为第一次添加元素必然没有创建tab，tab长度也必然为0.就会调用if中的语句：将resize()赋值给tab，再把tab的长度赋值给n

看看resize做了什么：

```java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
```

由于还未创建table数组，因此`oldTab`为空，因此`oldCap`为0，所以不会进入到`if(oldCap > 0)`的判断中，直接看else：将默认初始容量赋值给`newCap`，将扩容阈值赋值给`newThr`，为创建数组做准备。

由于`newThr`不为0，所以不会进入最后一个if语句。

将`newThr`赋值给成员变量`threshold`，再创建一个大小为`newCap`的Node数组`newTab`，再将`newTab`赋值给table，这下就完成了数组的创建。

最后resize方法会返回`newTab`，所以将resize()返回的结果赋值给tab，实际上就是创建了一个大小为16，扩容阈值为12的Node数组。

```java
 if ((p = tab[i = (n - 1) & hash]) == null)
    tab[i] = newNode(hash, key, value, null);
```

前面说参数的时候我们知道，这里的hash传进来的是hash(key)，即键的哈希值。将键的哈希值与数组长度-1做与运算，得到了该对象在数组中存放的索引，赋值给i，将tab[i]赋值给p

如果p为空，即该位置没有元素，那么直接在该位置创建一个新节点。

```java
   Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
        return new Node<>(hash, key, value, next);
    }
```

`newNode`方法比较简单，就是创建了一个Node对象。



最后判断++size是否大于threshold，对于第一次添加元素的情况来说是否定的。这个if语句就是用来判断是否需要扩容的。最终返回null，这也很符合我们的使用认知：覆盖之后返回被覆盖的值。由于原本为空，相当于覆盖了空，所以返回空。



**第三种情况：**

```java
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
```

上文说过，p就是tab[i]，即这个对象应该在数组中存放的位置上的元素地址值。

我们假设现在在下标为2的位置存了一个地址为`0x0011`，键为`aaa`，值为111的对象A。现在我们将一个地址为`0x0033`，键为`ccc`，值为333的对象C加入HashMap中，恰好算出来的应该存入哈希表中的位置就是下标为2的位置，发生重复。

那么此时这个第三方节点p记录的就是地址为`0x0011`，键为`aaa`，值为111的对象，它是一个副本。

首先判断`p.hash`是否等于当前要加入对象的哈希值，如果不等于，由于是&&，就直接跳过。

如果p是某个红黑树节点的地址，那么就将这个待加入节点加入红黑树。

如果不是红黑树的节点，那就要涉及到链表了：

```java
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
```

如果这个对象A的next为空，那么就原地新建一个节点作为它的下一个节点。再判断长度是否超过8，如果超过8就调用`treeifyBin`方法，在这个方法中还会判断数组长度是否大于64，如果两个条件都满足，就会将链表转换为红黑树。判断完是否需要转为红黑树后，退出循环。由于`p.next`原本为空，所以e为null，不会走后面的if语句。

如果`p.next`原本不为空，就会判断e的哈希值与待加入节点的哈希值是否相同，如果不相同的话就将e赋值给p再循环，相当于双指针迭代遍历链表。只要没有哈希值一样，最终p一定会是链表末尾的节点的地址，e最终一定为空。只要哈希值一样，e就是那个哈希值相同的节点的地址，即进入下文的第二种情况。



**第二种情况：**

现在，在对象C后面我们又存入了一个地址为`0x0044`，键为`ddd`，值为444的对象D。我们要存入一个地址为`0x0055`，键为`ddd`，值为555的对象D1。假设要存入的位置依然是下标为2的位置。

首先会将D的键的哈希值与A的键的哈希值进行比较，如果不一样就看A后面有没有元素，如果一样就进行覆盖。由于键不一样，那么由于A后面还有一个对象C，C后面还有D，再比较，发现键的哈希值相同，就要进行覆盖操作。我们来看看在源码中具体是如何实现的。

首先，p还是对象A的地址，因为它表示“应该存入的位置上的对象 的地址”。现在p不为null，走else语句：

```java
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
```

p（对象A的地址）的哈希值不等于D1的哈希值，也不是树节点，走内层的else语句：

```java
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
```

将p的下一个节点赋值给e，此时e为对象C的地址，不为空，走第二个if语句：

```java
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
```

e（对象C的地址）的哈希值也不等于D1的哈希值，不会进入if中，最后将e赋值给p，也就是原本为对象A的p，现在变成了对象C，这实际上相当于双指针迭代遍历链表。e是p的下一个节点的地址，然后通过将e赋值给p移动p，然后直接通过next得到p的下一个节点。

现在p是对象C的地址，没有break，所以还会继续在这个for循环中。将p的下一个节点地址赋值给e，此时e为对象D的地址，不为空，走第二个if语句。发现`e.hash==hash`成立，由于是&&，继续往后看。`e.key==key`也成立，由于是||，不用看了，必然成立，因此整条语句返回true，进入if语句，调用break，退出循环。

此时e不为空，走if语句进行赋值：

```java
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
```

将待覆盖的节点的value取出来赋值给`oldValue`，`onlyIfAbsent`为true，那么就进入if语句，将待加入的节点的value赋值给待覆盖节点的value，再返回待覆盖节点原本的value

也就是说，我们并没有对链表进行删除并新增节点的操作，只不过是将哈希值一样的节点的Value进行了替换而已。即：现在链表中依然还是A -> C -> D，没有D1对象，只不过D的Value从原本的444改为了555，然后将原本的444返回给了我们。