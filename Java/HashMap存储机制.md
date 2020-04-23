# Hashmap存储机制

哈希表（hash table）也叫散列表，是一种非常重要的数据结构，应用场景及其丰富，许多缓存技术（比如memcached）的核心其实就是在内存中维护一张大的哈希表。

- 什么是哈希表
- Hashmap实现原理
- 为什么Hashmap的数组长度一定是2的次幂
- 重写equals方法需同时重写hashCode方法

## 哈希表

先大概了解数据结构在基础操作上的执行性能

- **数组**：采用一段连续的存储单元来存储数据。对于指定下标的查找，时间复杂度为O(1)；通过给定值进行查找，需要遍历数组，逐一比对给定关键字和数组元素，时间复杂度为O(n)，当然，对于有序数组，则可采用二分查找，插值查找，斐波那契查找等方式，可将查找复杂度提高为O(logn)；对于一般的插入删除操作，涉及到数组元素的移动，其平均复杂度也为O(n)
- **线性链表**：对于链表的新增，删除等操作（在找到指定操作位置后），仅需处理结点间的引用即可，时间复杂度为O(1)，而查找操作需要遍历链表逐一进行比对，复杂度为O(n)
- **二叉树**：对一棵相对平衡的有序二叉树，对其进行插入，查找，删除等操作，平均复杂度均为O(logn)。
- **哈希表**：相比上述几种数据结构，在哈希表中进行添加，删除，查找等操作，性能十分之高，不考虑哈希冲突的情况下，仅需一次定位即可完成，时间复杂度为O(1)，接下来我们就来看看哈希表是如何实现达到惊艳的常数阶O(1)的。

在数组中根据下标查找某个元素，一次定位就可以达到，哈希表利用了这种特性，哈希表的**主干就是数组**。我们要新增或查找某个元素，我们通过把当前元素的关键字 通过某个函数映射到数组中的某个位置，通过数组下标一次定位就可完成操作，这函数就是**哈希函数**。

### 1. 常见的hash算法

1. **直接定址法**：直接以关键字k或者k加上某个常数（k+c）作为哈希地址（H(k)=ak+b）。
2. **数字分析法**：提取关键字中取值比较均匀的数字作为哈希地址（如一组出生日期，相较于年-月，月-日的差别要大得多，可以降低冲突概率）
3. **分段叠加法**：按照哈希表地址位数将关键字分成位数相等的几部分，其中最后一部分可以比较短。然后将这几部分相加，舍弃最高进位后的结果就是该关键字的哈希地址。
4. **平方取中法**：如果关键字各个部分分布都不均匀的话，可以先求出它的平方值，然后按照需求取中间的几位作为哈希地址。
5. **伪随机数法**：选择一随机函数，取关键字的随机值作为散列地址，通常用于关键字长度不同的场合。
6. **除留余数法**：用关键字k除以某个不大于哈希表长度m的数p，将所得余数作为哈希表地址（H(k)=k%p, p<=m; p一般取m或素数）。

### 2. 哈希冲突

如果两个不同的元素，通过哈希函数得出的实际存储地址相同怎么办？也就是说，当我们对某个元素进行哈希运算，得到一个存储地址，然后要进行插入的时候，发现已经被其他元素占用了，其实这就是所谓的哈希冲突，也叫哈希碰撞。前面我们提到过，哈希函数的设计至关重要，好的哈希函数会尽可能地保证 计算简单和散列地址分布均匀,但是，我们需要清楚的是，数组是一块连续的固定长度的内存空间，再好的哈希函数也不能保证得到的存储地址绝对不发生冲突。那么哈希冲突如何解决呢？

### 3. Hash算法解决冲突的方法

**开放定址法**：所谓的开放定址法就是一旦发生了冲突，就去寻找下一个空的散列地址，只要散列表足够大，空的散列地址总能找到，并将记录存入 公式为：fi(key) = (f(key)+di) MOD m (di=1,2,3,……,m-1)

1. **线性探测** 
   以增量序列 1，2，……，（TableSize -1）循环试探下一个存储地址，即di = i。如果table[index+di]为空则进行插入，反之试探下一个增量。但是线性探测也有弊端，就是会造成元素聚集现象，降低查找效率。

![](https://github.com/shenjiahuihm/note/blob/master/imgs/Java/线性探测.png)

**特别对于开放定址法的删除操作，不能简单的进行物理删除，因为对于同义词来说，这个地址可能在其查找路径上，若物理删除的话，会中断查找路径，故只能设置删除标志。**

2. **平方探测** 

   以增量序列1，-1，4，-4…且q ≤ TableSize/2 循环试探下一个存储地址。

3. **双散列探测** 

   di 为i*h2(key)，h2(key)是另一个散列函数。探测序列成：h2(key)，2h2(key)，3h2(key)，……。对任意的key，h2(key) ≠ 0 ！探测序列还应该保证所有的散列存储单元都应该能够被探测到。选择以下形式有良好的效果： 
   h2(key) = p - (key mod p) 
   其中：p < TableSize，p、TableSize都是素数。

**再哈希法**：再哈希法又叫双哈希法，有多个不同的Hash函数，当发生冲突时，使用第二个，第三个，….，等哈希函数计算地址，直到无冲突。虽然不易发生聚集，但是增加了计算时间。

**链地址法**：每个哈希表节点都有一个next指针，多个哈希表节点可以用next指针构成一个单向链表，被分配到同一个索引上的多个节点可以用这个单向链表连接起来，如：键值对k2, v2与键值对k1, v1通过计算后的索引值都为2，这时及产生冲突，但是可以通过next指针将k2, k1所在的节点连接起来，这样就解决了哈希的冲突问题。
![](https://github.com/shenjiahuihm/note/blob/master/imgs/Java/链地址法.png)

**建立公共溢出区**：将哈希表分为基本表和溢出表两部分，凡是和基本表发生冲突的元素，一律填入溢出表。

### 4. Hashmap实现原理

#### 存储结构

- hashMap底层是以数组的方式进行存储，将key-value对作为数组中的一个元素存储。

- key-value都是Map.Entry中的属性，将key的值进行hash之后存储，每一个hash值对应一个数组下标，数组下标识根据hash值和数组长度计算。

- 由于不同的key可能hash值相同，hashmap采用链表形式存储。

  ```
  //HashMap的主干数组，可以看到就是一个Entry数组，初始值为空数组{}，主干数组的长度一定是2的次幂，至于为什么这么做，后面会有详细分析。
  transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
  ```

#### Entry结构分析

Entry是HashMap中封装key-value键值对的一个静态内部类。

```java
static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;//map中key值，可以为null
        V value;//map中value值，可以为null
        Entry<K,V> next;//存储指向下一个Entry的引用，单链表结构链表引用，防止可以值不同，hash值相同
        int hash;//对key的hashcode值进行hash运算后得到的值，存储在Entry，避免重复计算

        /**
         * Creates new entry.构造函数
         */
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        } 
        public final K getKey() {
            return key;
        }

        public final V getValue() {
            return value;
        }
        // 同一个key时，新值替换旧值，返回旧值
        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
         // key值重写equals方法
        public final boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry e = (Map.Entry)o;
            Object k1 = getKey();
            Object k2 = e.getKey();
            if (k1 == k2 || (k1 != null && k1.equals(k2))) {
                Object v1 = getValue();
                Object v2 = e.getValue();
                if (v1 == v2 || (v1 != null && v1.equals(v2)))
                    return true;
            }
            return false;
        }
        // 重写hashCode值
        public final int hashCode() {
            return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
        }

        public final String toString() {
            return getKey() + "=" + getValue();
        }
```

所以，HashMap的整体结构如下：

![](https://github.com/shenjiahuihm/note/blob/master/imgs/Java/hashmap.png)

简单来说，HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的，如果定位到的数组位置不含链表（当前entry的next指向null）,那么对于查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度为O(n)，首先遍历链表，存在即覆盖，否则新增；对于查找操作来讲，仍需遍历链表，然后通过key对象的equals方法逐一比对查找。所以，性能考虑，HashMap中的链表出现越少，性能才会越好。

#### HashMap属性分析

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable
{

    /**
     *默认情况下，hashmap大小为16.即1<<4就是1乘以2的4次幂=16
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
      * hashMap的最大值
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 默认加载加载因子，即使用空间达到总空间的0.75时，需要扩容。
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 声明hashmap一个空数组。
     */
    static final Entry<?,?>[] EMPTY_TABLE = {};

    /**
     * 最开始时，hashmap是一个空数组。
     */
    transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

    /**
     * map的元素的个数
     * 实际存储的key-value键值对的个数
     */
    transient int size;

    /*
     * hashmap的实际存储空间大小。这个空间是总空间*加载因子得出的大小。
     * 比如默认是16，加载因子是0.74。则threshold就是12。
     * 阈值，当table == {}时，该值为初始容量（初始容量默认为16）；当table被填充了，也就是为table分配内存空间后，threshold一般为 capacity*loadFactory。HashMap在进行扩容时需要参考threshold，后面会详细谈到
     */
    int threshold;

    /**
     * 加载因子，即使用空间达到总空间的0.75时，需要扩容。
     * 也说负载因子，代表了table的填充度有多少，默认是0.75
     */
    final float loadFactor;

    /**
     *用于快速失败，由于HashMap非线程安全，在对HashMap进行迭代时，如果期间其他线程的参与导致HashMap的结构发生变化了（比如put，remove等操作），需要抛出异常ConcurrentModificationException
     */
    transient int modCount;

    /**
     *  threshold这个值的最大值就是Integer.MAX_VALUE
     */
    static final int ALTERNATIVE_HASHING_THRESHOLD_DEFAULT = Integer.MAX_VALUE;
```

实际上HashMap有4个构造器，其他构造器如果用户没有传入initialCapacity 和loadFactor这两个参数，会使用默认值
initialCapacity默认为16，loadFactory默认为0.75，我们看下其中一个

```java
public HashMap(int initialCapacity, float loadFactor) {
　　　　　//此处对传入的初始容量进行校验，最大不能超过MAXIMUM_CAPACITY = 1<<30(230)
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        this.loadFactor = loadFactor;
        threshold = initialCapacity;
　　　　　
        init();//init方法在HashMap中没有实际实现，不过在其子类如 linkedHashMap中就会有对应实现
    }
```

#### put方法

从上面这段代码我们可以看出，在常规构造器中，没有为数组table分配内存空间（有一个入参为指定Map的构造器例外），而是在执行put操作的时候才真正构建table数组

```java
public V put(K key, V value) {
        //如果table数组为空数组{}，进行数组填充（为table分配实际内存空间），入参为threshold，此时threshold为initialCapacity 默认是1<<4(24=16)
        if (table == EMPTY_TABLE) {//首次存储元素，初始化存储空间
            inflateTable(threshold);
        }
       //如果key为null，将null放入元素的第一个位置,存储位置为table[0]或table[0]的冲突链上
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);//对key的hashcode进一步计算，确保散列均匀
        int i = indexFor(hash, table.length);//根据key的hash值，数组长度计算该Entry<key,value>的数组下标,获取在table中的实际位置
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        //如果该对应数据已存在，执行覆盖操作。用新value替换旧value，并返回旧value
            Object k;
            //判断同一个key，既要判断hash值相同，还要判断key是同一个key，因为相同的key有可能hash值也相同。双重判断保证是同一个key。
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;//如果是新的key需要存储，则增加操作次数modCount++,保证并发访问时，若HashMap内部结构发生变化，快速响应失败
        addEntry(hash, key, value, i);//新增一个entry,新增key-value键值对
        return null;
    }    
```

#### addEntry方法

addEntry方法是将新增的key-value键值对存入到map中。该方法主要完成两个功能：

1. 添加新元素前， 判断是否需要对map的数组进行扩容，如果需要扩容，则扩容空间大小是多少?
2. 对于新增key-value键值对，如果key的hash值相同，则构造单向列表。
   

```java
/**
**  hash:key的hash值
**  key:存储的键
**  value：存储的value对象值
*** bucketIndex：数组下标位置，即key-value在数组中的位置。
**/
void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);//当size超过临界阈值threshold，并且即将发生哈希冲突时进行扩容
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }
       // 往数组中添加新的key-value键值对
        createEntry(hash, key, value, bucketIndex);
    }
```

通过以上代码能够得知，当发生哈希冲突并且size大于阈值的时候，需要进行数组扩容，扩容时，需要新建一个长度为之前数组2倍的新的数组，然后将当前的Entry数组中的元素全部传输过去，扩容后的新数组长度为之前的2倍，所以扩容相对来说是个耗资源的操作。

#### createEntry方法

该方法主要完成两个功能，第一是添加新的key到Entry数组中，第二就是对于不同key的hash值相同的情况下，在同一个数组下标处，构建单向链表进行存储。

```java
void createEntry(int hash, K key, V value, int bucketIndex) {
       // 取出当前位置的元素，如果是新添加的key,则e为null，已经有的元素为不为空。
        Entry<K,V> e = table[bucketIndex];
        // 添加新的key-value值或构建链表
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        size++;
    }
```

先来看看inflateTable这个方法

```java
private void inflateTable(int toSize) {
        int capacity = roundUpToPowerOf2(toSize);//capacity一定是2的次幂
        threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);//此处为threshold赋值，取capacity*loadFactor和MAXIMUM_CAPACITY+1的最小值，capaticy一定不会超过MAXIMUM_CAPACITY，除非loadFactor大于1
        table = new Entry[capacity];
        initHashSeedAsNeeded(capacity);
    }
```

inflateTable这个方法用于为主干数组table在内存中分配存储空间，通过roundUpToPowerOf2(toSize)可以确保capacity为大于或等于toSize的最接近toSize的二次幂，比如toSize=13,则capacity=16;to_size=16,capacity=16;to_size=17,capacity=32

```java
private static int roundUpToPowerOf2(int number) {
        // assert number >= 0 : "number must be non-negative";
        return number >= MAXIMUM_CAPACITY
                ? MAXIMUM_CAPACITY
                : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
    }
```

roundUpToPowerOf2中的这段处理使得数组长度一定为2的次幂，Integer.highestOneBit是用来获取最左边的bit（其他bit位为0）所代表的数值.

#### hash函数

```
//这是一个神奇的函数，用了很多的异或，移位等运算，对key的hashcode进一步进行计算以及二进制位的调整等来保证最终获取的存储位置尽量分布均匀
final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
```

以上hash函数计算出的值，通过indexFor进一步处理来获取实际的存储位置

```
/**
     * 返回数组下标
     */
    static int indexFor(int h, int length) {
        return h & (length-1);
    }
```

h&（length-1）保证获取的index一定在数组范围内，举个例子，默认容量16，length-1=15，h=18,转换成二进制计算为

```
     1  0  0  1  0
    &   0  1  1  1  1
    __________________
        0  0  0  1  0    = 2
```

最终计算出的index=2。有些版本的对于此处的计算会使用 取模运算，也能保证index一定在数组范围内，不过位运算对计算机来说，性能更高一些（HashMap中有大量位运算）

![](https://github.com/shenjiahuihm/note/blob/master/imgs/Java/hashmap流程图.png)

## 为何HashMap的数组长度一定是2的次幂？

````
void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }
````

如果数组进行扩容，数组长度发生变化，而存储位置 index = h&(length-1),index也可能会发生变化，需要重新计算index，我们先来看看transfer这个方法

```java
void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
　　　　　//for循环中的代码，逐个遍历链表，重新计算索引位置，将老数组数据复制到新数组中去（数组不存储实际数据，所以仅仅是拷贝引用而已）
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
　　　　　　　　　 //将当前entry的next链指向新的索引位置,newTable[i]有可能为空，有可能也是个entry链，如果是entry链，直接在链表头部插入。
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
```

这个方法将老数组中的数据逐个链表地遍历，扔到新的扩容后的数组中，我们的数组索引位置的计算是通过 对key值的hashcode进行hash扰乱运算后，再通过和 length-1进行位运算得到最终数组索引位置。
　　hashMap的数组长度一定保持2的次幂，比如16的二进制表示为 10000，那么length-1就是15，二进制为01111，同理扩容后的数组长度为32，二进制表示为100000，length-1为31，二进制表示为011111。从下图可以我们也能看到这样会保证低位全为1，而扩容后只有一位差异，也就是多出了最左位的1，这样在通过 h&(length-1)的时候，只要h对应的最左边的那一个差异位为0，就能保证得到的新的数组索引和老数组索引一致(大大减少了之前已经散列良好的老数组的数据位置重新调换)。

![](https://github.com/shenjiahuihm/note/blob/master/imgs/Java/hashmap推导1.png)

还有，数组长度保持2的次幂，length-1的低位都为1，会使得获得的数组索引index更加均匀，比如：

![](https://github.com/shenjiahuihm/note/blob/master/imgs/Java/hashmap推导2.png)

我们看到，上面的&运算，高位是不会对结果产生影响的（hash函数采用各种位运算可能也是为了使得低位更加散列），我们只关注低位bit，如果低位全部为1，那么对于h低位部分来说，任何一位的变化都会对结果产生影响，也就是说，要得到index=21这个存储位置，h的低位只有这一种组合。这也是数组长度设计为必须为2的次幂的原因。

![](https://github.com/shenjiahuihm/note/blob/master/imgs/Java/hashmap推导3.png)

如果不是2的次幂，也就是低位不是全为1此时，要使得index=21，h的低位部分不再具有唯一性了，哈希冲突的几率会变的更大，同时，index对应的这个bit位无论如何不会等于1了，而对应的那些数组位置也就被白白浪费了。

#### 　get方法

```
public V get(Object key) {
　　　　 //如果key为null,则直接去table[0]处去检索即可。
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);
        return null == entry ? null : entry.getValue();
 }
```

get方法通过key值返回对应value，如果key为null，直接去table[0]处检索。我们再看一下getEntry这个方法

```java
final Entry<K,V> getEntry(Object key) {
            
        if (size == 0) {
            return null;
        }
        //通过key的hashcode值计算hash值
        int hash = (key == null) ? 0 : hash(key);
        //indexFor (hash&length-1) 获取最终数组索引，然后遍历链表，通过equals方法比对找出对应记录
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash && 
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }    
```

可以看出，get方法的实现相对简单，key(hashcode)–>hash–>indexFor–>最终索引位置，找到对应位置table[i]，再查看是否有链表，遍历链表，通过key的equals方法比对查找对应的记录。要注意的是，有人觉得上面在定位到数组位置之后然后遍历链表的时候，e.hash == hash这个判断没必要，仅通过equals判断就可以。其实不然，试想一下，如果传入的key对象重写了equals方法却没有重写hashCode，而恰巧此对象定位到这个数组位置，如果仅仅用equals判断可能是相等的，但其hashCode和当前对象不一致，这种情况，根据Object的hashCode的约定，不能返回当前对象，而应该返回null，后面的例子会做出进一步解释。
## 重写equals方法需同时重写hashCode方法
如果我们已经对HashMap的原理有了一定了解，这个结果就不难理解了。尽管我们在进行get和put操作的时候，使用的key从逻辑上讲是等值的（通过equals比较是相等的），但由于没有重写hashCode方法，所以put操作时，key(hashcode1)–>hash–>indexFor–>最终索引位置 ，而通过key取出value的时候 key(hashcode1)–>hash–>indexFor–>最终索引位置，由于hashcode1不等于hashcode2，导致没有定位到一个数组位置而返回逻辑上错误的值null（也有可能碰巧定位到一个数组位置，但是也会判断其entry的hash值是否相等，上面get方法中有提到。）
　　所以，在重写equals的方法的时候，必须注意重写hashCode方法，同时还要保证通过equals判断相等的两个对象，调用hashCode方法要返回同样的整数值。而如果equals判断不相等的两个对象，其hashCode可以相同（只不过会发生哈希冲突，应尽量避免）。

