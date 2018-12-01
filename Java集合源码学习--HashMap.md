#Java集合--HashMap、HashSet、HashTable源码学习
[参考文献](https://www.zybuluo.com/kiraSally/note/819843)  

# 一、HashMap
HashMap作为我们常用的一个集合类，一直对它常用方法的实现很好奇，特在此将学习的笔记记录一下。

## 1. HashMap的数据结构  
### 1.1 类定义  

	public class HashMap<K,V>
    	extends AbstractMap<K,V>
    	implements Map<K,V>, Cloneable, Serializable  

### 1.2 重要的全局变量  
**链表散列的数据结构(数组+链表【冲突解决方案-封闭寻址方法】)**  

	//The default initial capacity - MUST be a power of two.
	//默认的初始的容量-必须是2的幂
	static final int DEFAULT_INITIAL_CAPACITY = 16;
	//The maximum capacity - MUST be a power of two <= 1<<30.
	//最大容量-必须是2的幂
	static final int MAXIMUM_CAPACITY = 1 << 30;
	//The load factor used when none specified in constructor.
	//默认的"负载因子"(负载因子是用来衡量桶被使用的上限的)
	static final float DEFAULT_LOAD_FACTOR = 0.75f;
	//The table, resized as necessary. Length MUST Always be a power of two.
	//放置元素的"桶"：每一个位置都是一个"链表"，用来解决哈希冲突
	transient Entry<K,V>[] table;
	//The number of key-value mappings contained in this map.
	//所有的key-value对：并不是桶的length
	transient int size;
	//The next size value at which to resize (capacity * load factor).
	//用于和当前的size进行比较，决定是否进行扩容
	int threshold;
	//The load factor for the hash table.
	//这个是可以通过构造器设置的负载因子
	final float loadFactor;
	/**
	  * The number of times this HashMap has been structurally modified
	  * Structural modifications are those that change the number of mappings in
	  * the HashMap or otherwise modify its internal structure (e.g.,
	  * rehash).  This field is used to make iterators on Collection-views of
	  * the HashMap fail-fast.  (See ConcurrentModificationException).
	  */
	transient int modCount;  

### 1.3 构造器  

	public HashMap(int initialCapacity, float loadFactor) {
	    if (initialCapacity < 0)
	        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
	    if (initialCapacity > MAXIMUM_CAPACITY)
	        initialCapacity = MAXIMUM_CAPACITY;
	    if (loadFactor <= 0 || Float.isNaN(loadFactor))
	        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
	    // Find a power of 2 >= initialCapacity
	        int capacity = 1;
	        while (capacity < initialCapacity)
	            capacity <<= 1;
	        this.loadFactor = loadFactor;
	        //阈值为容量*负载因子和最大容量+1之间的最小值 以此值作为容量翻倍的依据(不能超过最大容量)
	        threshold = (int)Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
	        //初始化一个2次幂的Entry类型数组 一个桶对应一个Entry对象
	        table = new Entry[capacity];
	        useAltHashing = sun.misc.VM.isBooted() && 
	        (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
	        init();
	}  

> 分析：首先构造器对参数进行控制，如果传入的**initialCapacity是合法的(0<initialCapacity<MAXIMUM_CAPACITY)**则设置为该值，注意：在while循环中**capacity<<=1保证了最终的capacity一定是一个2的幂，即使initialCapacity传入的不是2的幂**；然后，对负载因子进行赋值，初始化threshold，以及桶。  

### 1.4 Entry  

	/**
	  * 静态类 默认实现内部Entry接口 (接口中可定义内部接口-Map.Entry接口为Map的内部接口)
	  * PS:JDK8中引入default，作用为在接口中定义默认方法实现
	  */
	static class Entry<K,V> implements Map.Entry<K,V> {
	    final K key;//key具有引用不可变特性
	    V value;
	    Entry<K,V> next;//next指向下一个：单向链表，头插入
	    final int hash;
	    ……
	}  

> 分析：可以看到该类含有k和v，Map实际的key-value就是在这个类中封装的，而且还包括hash值，通过该类的**Entry<K,V> next**成员我们可以看出，这明显是一个**链点**，这也印证了上面说的table的元素是一个链表。  

## 2. HashMap的重要方法  
### 2.1 put方法  
**HashMap允许存放null的key和null的value，其存储过程如下：**  

* 当key==null时，若存在该entry则覆盖value，否则新增key为null的entry
* 根据hash计算该entry所在桶的位置 -> 即数组下标index
* 遍历链表，若该key存在则更新value，否则新增entry (有则更新，无则新增)  

代码：

	/**
	  * 存放键值对
	  * 允许存放null的key和null的value
	  * @return key不存在返回null，否则返回旧值
	  */
	public V put(K key, V value) {
	    /**
	     * 1.针对key为null的处理有两种方式：
	     *  1.1从tab[0]开始向后遍历，若存在key为null的entry，则覆盖其的value
	     *  1.2遍历完成但仍不存在key为null的entry，则新增key为null的entry
	     * 准则：一个HashMap只允许有一个key为null的键值对
	     * 注意：由于hash(null) = 0，因此key=null的键值对永远在table[0]
	     */
	    if (key == null)
	        return putForNullKey(value);
	    /**
	     * 根据key的hashCode并结合hash()算法计算出新的hash值
	     * 目的是为了增强hash混淆 -> 因为key.hashCode()很可能不太靠谱
	     * 而HashMap是极度依赖hash来实现元素离散的，通俗来说就是
	     * 尽可能的让键值对可以分到不同的桶中以便快速索引
	     */
	    int hash = hash(key);
	    //2.根据hash计算该元素所在桶的位置 -> 即数组下标索引
	    int i = indexFor(hash, table.length);
	    /**
	     * 3.遍历链表，若该key存在则更新value，否则新增entry
	     *   判断该key存在的依据有两个：
	     *     1.key的hash一致
	     *     2.key一致(引用一致或字符串一致)
	     *   put操作遵循：有则更新，无则新增
	     */
	    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
	        //使用临时变量k临时存储遍历时的e.key -> 主要是为了代码复用
	        Object k;
	        //hash一致 && (key引用相同 或 key字符串比较相同)
	        if (e.hash == hash && ((k = e.key) == key || key.equals(k))){
	            //值更新
	            V oldValue = e.value;
	            e.value = value;
	            //钩子方法
	            e.recordAccess(this);
	            //返回旧值
	            return oldValue;
	        }
	    }
	    //put操作属于结构变更操作
	    modCount++;
	    //无则新增
	    addEntry(hash, key, value, i);
	    //该key不存在就返回null -> 即新增返回null
	    return null;
	}

> 分析：首先因为HashMap支持key为null，所以对key为null的情况进行单独的处理；put方法最终要的就是一般情况了：首先，基本思想是通过hash找到存放的地点，如果冲突则通过链表解决，我们来看看HashMap是如何实现的。**首先，根据key的求出对应的hash值，然后通过这个hash值映射到同的下标(int i=indexFor(hash,table.length)；**然后，在for循环中进入**table[i]处的链表进行查找**，如果有该key值则设置为newValue，并返回oldValue，否则调用**addEntry(hash,key,value,i)添加新的元素**；值得注意的一点是：**用新的value替换老的value的前提是，这个key指向同一个引用域(或者equals方法自定义判断)，而且其hash值相等(index相等不代表hash相等)，因此这就解释了为甚自定义Class的时候其equals方法返回true的时候，hashCode必须相等，因为这样才能保证同一个key只能有一个(因为equals返回true说明是同一个key，只有hashCode相同才会进行覆盖，而不是添加新的)**  

### 2.2 hash方法  

	//JDK1.7
	final int hash(Object k) {
	    int h = 0;
	    if (useAltHashing) {
	        if (k instanceof String) {
	            return sun.misc.Hashing.stringHash32((String) k);
	        }
	        h = hashSeed;
	    }
	    //异或就是两个数的二进制形式，按位对比，相同取0，不同取一
	    //此算法加入了高位计算，防止低位不变，高位变化时，造成的 hash 冲突
	    h ^= k.hashCode();
	    h ^= (h >>> 20) ^ (h >>> 12);
	    return h ^ (h >>> 7) ^ (h >>> 4);
	}
	//JDK1.8 扰动函数 -> 散列值优化函数
	static final int hash(Object key) {
	    int h;
	    //把一个数右移16位即丢弃低16位，就是任何小于2^16的数，右移16后结果都为0
	    //2的16次方再右移刚好就是1 同时int最大值为32位
	    //任何一个数，与0按位异或的结果都是这个数本身
	    //对于非null的hash值，仅在其大于等于2^16的时候才会重新调整其值
	    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
	}  

> 分析：jdk1.7中的hash方法看不懂；**1.8中的hash方法就是通过，key的hashCode的高16为与低16为进行异或操作算的**

### 2.3 indexFor  

	/**
	  * Int范围(2^32)从-2147483648到2147483648，加起来大概40亿空间，内存不能直接读取
	  * 用之前还要先做对数组的长度取模运算，得到的余数才能用来访问数组下标
	  * @Param h 根据hash方法得到h
	  * @Param length 一定是2次幂
	  */
	static int indexFor(int h, int length) {
	    //2次幂-1 返回的结果的二进制为永远是都是1 比如 15 -> 1111 (16 -> 10000)
	    //与运算 只有 1 & 1 = 1 正好相当于一个“低位掩码”
	    //如果length-1中某一位为0，则不论h中对应位的数字为几，对应位结果都是0，这样就让两个h取到同一个结果，hash冲突
	    //同时这个操作可以保证索引不会大于数组的大小(见开头的描述)
	    return h & (length-1);
	}  

> 分析：这个方法是通过**key的hashCode(在调用的时候传入的是key的hashCode值)去映射桶的数组下标，计算下标的方法简单的出乎意料，就是hashCode对数组的长度进行一次求摸运算(h与length-1求与可以实现的原因是length是2的幂，所以length-1一定全部为1的序列，所以可以实现)**  

### 2.4 addEntry  

	//该方法为包访问 package java.util(本包私有性高于子类)
	void addEntry(int hash, K key, V value, int bucketIndex) {
	    /**
	     * 扩容条件：实际容量超过阈值 && 当前坐标数组非空
	     * 有个优雅的设计在于，若bucketIndex处没有Entry对象，
	     * 那么新添加的entry对象指向null，从而就不会有链了
	     */
	    if ((size >= threshold) && (null != table[bucketIndex])) {
	        //最大容量每次都扩容一倍
	        resize(2 * table.length);
	        //重算hash
	        hash = (null != key) ? hash(key) : 0;
	        //重算index
	        bucketIndex = indexFor(hash, table.length);
	    }
	    //新增Entry元素到数组的指定下标位置
	    createEntry(hash, key, value, bucketIndex);
	}
	//该方法为包访问 package java.util
	void createEntry(int hash, K key, V value, int bucketIndex) {
	    //获取数组中指定 bucketIndex 下标索引处的 Entry
	    Entry<K,V> e = table[bucketIndex];
	    /**
	     * 将新创建的Entry放入 bucketIndex 索引处，
	     * 并让新的Entry(next)指向原来的Entry形成链表
	     * 新加入的放入链表头部 -> 类似于栈，顶部是最新的入栈元素
	     * 对于"桶"的命名，笔者认为这词儿很形象生动：
	     *    1.桶具有"deep深度"的属性 -> 桶深即链表长度
	     *    2.桶的顶部永远是最新"扔"进入的元素
	     */
	    table[bucketIndex] = new Entry<>(hash, key, value, e);
	    //元素数量++
	    size++;
	}  

> 分析：这个方法可以帮助我们理解**threshold、loadFactor两个参数的作用**。在自定义类的时候，重写equals方法时常常要求hashCode方法也满足一定的要求，当时不太理解，现在明白了。HashMap在解决hash冲突的时候使用的是链表，如果hash冲突频繁出现的话，**冗长的链表带来的是查找的费事(数组的下标查找比链表要快的多)，所以尽可能的使得hashCode散列，冲突少一些**；所以，当**当前的元素数目大于threshold时，说明hash冲突的可能性已经不满足你的要求了，这时候进行扩容，可以减少冲突(indexFor的计算用到了length);因为threshold的计算使用了loadFactor，所以loadFactor(负载因子)就是衡量HashMap"对冲突的容忍度"的指标**。然后，可以看到Map的思想，我们知道根据数组的下标访问元素是非常快的，但是我们想要的是通过**"key"**来访问对应的元素，而key的类型多种多样，于是思想通过hash函数将一个个的key转换为int数，而这个int数来映射到数组的下标肯定是有冲突的，而解决冲突使用的链表法又浪费了效率，所以不难理解，为什么是用所有元素的个数(size)来和threshold进行比较进行扩容。  

### 2.5 resize  

**HashMap的扩容采用数组替换的方式:**  

* 先创建一个两倍于原容量的新数组
* 遍历旧数组对新数组赋值，期间会同时完成新旧索引的变更
* 新数组替换旧数组引用
* 重新计算阈值

代码：

	/**
	  * Rehashes the contents of this map into a new array with a
	  * larger capacity.  This method is called automatically when the
	  * number of keys in this map reaches its threshold.
	  * 
	  * 目的：通过增加内部数组的长度的方式，从而保证链表中只保留很少的Entry对象，从而降低put(),remove()和get()方法的执行时间
	  * 注意：如果两个Entry对象的键的哈希值不一样，但它们之前在同一个桶上，那么在调整以后，并不能保证它们依然在同一个桶上
	  */
	void resize(int newCapacity) {
	    /**
	     * 使用临时拷贝，保证当前数组长度的时效性(参见JAVA的`观察者`模式实现)
	     * 否则多线程环境下很容易造成oldCapacity的变化，
	     * 进而导致由一系列过期数据的造成的错误
	     */
	    Entry[] oldTable = table;
	    int oldCapacity = oldTable.length;
	    //达到最大容量时禁止扩容 - 边界保护
	    if (oldCapacity == MAXIMUM_CAPACITY) {
	        threshold = Integer.MAX_VALUE;
	        return;
	    }
	    //1.实例化一个newCapacity容量的新数组
	    Entry[] newTable = new Entry[newCapacity];
	    boolean oldAltHashing = useAltHashing;
	    useAltHashing |= sun.misc.VM.isBooted() &&
	            (newCapacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
	    boolean rehash = oldAltHashing ^ useAltHashing;
	    //2.遍历旧数组对新数组赋值
	    transfer(newTable, rehash);
	    //3.新数组替换旧数组
	    table = newTable;
	    //4.重新计算阈值
	    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
	}
  
### 2.6 transfer  

	/**
	  * Transfers all entries from current table to newTable.
	  */
	void transfer(Entry[] newTable, boolean rehash) {
	    int newCapacity = newTable.length;
	    //注意：如果在新表的数组索引位置相同，则链表元素会倒置，也就是先插入最近的
	    for (Entry<K,V> e : table) {
	        while(null != e) {
	            Entry<K,V> next = e.next;
	            if (rehash) {
	                //重新计算hash null的位置还是tab[0]不变
	                e.hash = null == e.key ? 0 : hash(e.key);
	            }
	            //重新计算下标索引(主要是因为容量变化成2倍)
	            int i = indexFor(e.hash, newCapacity);
	            //注意：多线程环境可能由于执行次序非有序造成next引用变更赋值出错导致环形链接出现，从而造成死循环
	            e.next = newTable[i];
	            newTable[i] = e;
	            e = next;
	        }
	    }
	}  

### 2.7 remove  
#### 2.7.1 HashMap的remove实现  

	public V remove(Object key) {
	    Entry<K,V> e = removeEntryForKey(key);
	    return (e == null ? null : e.value);
	}  

#### 2.7.2 HashMap.KeySet的remove实现  

	public boolean remove(Object o) {
	    return HashMap.this.removeEntryForKey(o) != null;
	}  

#### 2.7.3 HashMap.HashIterator的remove实现  

	public void remove() {
	    if (current == null)
	        throw new IllegalStateException();
	    if (modCount != expectedModCount)
	        throw new ConcurrentModificationException();
	    Object k = current.key;
	    current = null;
	    HashMap.this.removeEntryForKey(k);
	    //重要操作：迭代器中删除时同步了expectedModCount值与modCount相同
	    expectedModCount = modCount;
	}  

#### 2.7.4 removeEntryForKey  

	/**
	  * Removes and returns the entry associated with the specified key
	  * in the HashMap.  Returns null if the HashMap contains no mapping
	  * for this key.
	  */
	final Entry<K,V> removeEntryForKey(Object key) {
	    //计算hash
	    int hash = (key == null) ? 0 : hash(key);
	    //根据hash定位数组下标index索引
	    int i = indexFor(hash, table.length);
	    /**
	     * 记录前一个enrty元素(默认从链表首元素开始)
	     *  1.table[i] 表示链表首元素 - 即桶顶部元素
	     *  2.prev会记录遍历时的当前entry的前一个entry
	     */
	    Entry<K,V> prev = table[i];
	    //沿着链表从头到尾顺序遍历
	    Entry<K,V> e = prev;
	    //遍历key所在链表
	    while (e != null) {
	        //当前key对应entry的下一个entry
	        Entry<K,V> next = e.next;
	        Object k;
	        //1.一旦key一致，即找到该key对应entry便移除并返回该entry
	        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
	            //remove属于结构性更新，modCount计数+1
	            modCount++;
	            //实际元素数量-1
	            size--;
	            /**
	             * 移除当前key对应entry，实际上就是
	             * 通过变更前后链接将该entry从链表中移除
	             * 1.若当前key正好位于链首，则链首指向next
	             * 2.若当前key不位于链首，则该key之前的元素的next指向该key的下一个元素
	             */
	            if (prev == e)
	                table[i] = next;
	            else 
	                prev.next = next;
	            //LinkedHashMap专用钩子方法   
	            e.recordRemoval(this);
	            return e;
	        }
	        //2.找不到就继续往后找
	        //记录当前key对应entry
	        prev = e;
	        //指向下一次循环元素
	        e = next;
	    }
	    return e;
	}  


# 二、HashSet  
Set集合我们用的比较少
## 1. HashSet的数据结构  
### 1.1 类定义  

	public class HashSet<E>
	    extends AbstractSet<E>
	    implements Set<E>, Cloneable, java.io.Serializable  

### 1.2 重要的全局变量  

	//维护一个HashMap作为底层数据结构
	private transient HashMap<E,Object> map;
	//Dummy value， HashSet存储时，将该值作为底层map的默认value
	private static final Object PRESENT = new Object();  

> 分析：惊讶的发现，HashSet底层使用HashMap来存储数据的  

### 1.3 构造器  

	/**
	 * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
	 * default initial capacity (16) and load factor (0.75).
	 */
	public HashSet() {
	    map = new HashMap<>();
	}
	/**
	 * 构造一个包含指定collection中的元素的新set。
	 * 实际底层使用默认的加载因子0.75和足以包含指定collection中所有元素的初始容量来创建一个HashMap。
	 * @param c 其中的元素将存放在此set中的collection。
	 */
	public HashSet(Collection<? extends E> c) {
	    map = new HashMap<E,Object>(Math.max((int) (c.size()/.75f) + 1, 16));
	    addAll(c);
	}
	/**
	 * 以指定的initialCapacity和loadFactor构造一个空的HashSet。
	 * 实际底层以相应的参数构造一个空的HashMap。
	 * @param initialCapacity 初始容量。
	 * @param loadFactor 加载因子。
	 */
	public HashSet(int initialCapacity, float loadFactor) {
	    map = new HashMap<>(initialCapacity, loadFactor);
	}
	/**
	 * 以指定的initialCapacity构造一个空的HashSet。
	 * 实际底层以相应的参数及加载因子loadFactor为0.75构造一个空的HashMap。
	 * @param initialCapacity 初始容量。
	 */
	public HashSet(int initialCapacity) {
	    map = new HashMap<>(initialCapacity);
	}
	/**
	 * 以指定的initialCapacity和loadFactor构造一个新的空链接哈希集合。此构造函数为包访问权限，不对外公开，实际只是是对LinkedHashSet的支持。实际底层会以指定的参数构造一个空LinkedHashMap实例来实现。
	 * @param initialCapacity 初始容量。
	 * @param loadFactor 加载因子。
	 * @param dummy 标记。
	 */
	HashSet(int initialCapacity, float loadFactor, boolean dummy) {
	    map = new LinkedHashMap<>(initialCapacity, loadFactor);
	}  

> 分析：可以看到，构造器基本上就是初始化HashMap  

## 2. HashSet重要方法  
### 2.1 add方法  

	/**
	 * 使用HashMap作为底层数据结构
	 *   1.利用HashMap的key不重复原则实现非重
	 *   2.由于HashMap的非有序导致HashSet的非有序
	 *   3.HashMap的value使用一个final的static变量作为默认值
	 * @Return 若put成功，返回true否则false (该方法在添加 key 不重复的键值对的时候，会返回 null)
	 */
	public boolean add(E e) {
	    return map.put(e, PRESENT)==null;
	}  

### 2.2 clone方法  

	/**
	 * 返回此HashSet实例的浅表副本：并没有复制这些元素本身
	 * @Return 底层实际调用HashMap的clone()方法，获取HashMap的浅表副本
	 * 拓展：
	 *    1.浅复制就是仅复制类中的值类型成员
	 *    2.深复制就是复制类中的值类型成员和引用类型的成员
	 */
	public Object clone() {
	    try {
	        HashSet<E> newSet = (HashSet<E>) super.clone();
	        newSet.map = (HashMap<E, Object>) map.clone();
	        return newSet;
	    } catch (CloneNotSupportedException e) {
	        throw new InternalError();
	    }
	}  

### 2.3 iterator方法  

	/**
	* 1.基于迭代器模式(Set接口定义Iterator<E> iterator()方法)
	* 2.迭代map的keySet集合
	*/
	public Iterator<E> iterator() {
	    return map.keySet().iterator();
	}  

# 三、HashTable  
HashTable和HashMap很像，但是在速度上比不上HashMap，但是，HashTable的相关方法都是线程安全的，但即使这样我们也不推荐使用(ConcurrentHashMap也是线程安全的)  

## 1. HashTable的数据结构  
### 1.1 类定义  

	public class Hashtable<K,V>
	    extends Dictionary<K,V>
	    implements Map<K,V>, Cloneable, java.io.Serializable  

> 分析：与HashMap一个区别就是这里继承的是Dictionary，而不是AbstractMap  

### 1.2 
	
	//The hash table data.
	//底层维护一个Entry(键值对)数组
	private transient Entry<K,V>[] table;
	//The total number of entries in the hash table.
	//元素总量，等同于HashMap的size
	private transient int count;
	//The load factor for the hashtable.
	//负载因子
	private float loadFactor;
	/** 
	 * The table is rehashed when its size exceeds this threshold.
	 * (The value of this field is (int)(capacity * loadFactor).)
	 * 超过阈值进行rehash
	 */
	private int threshold;
	/**
	  * The number of times this Hashtable has been structurally modified
	  * Structural modifications are those that change the number of entries in
	  * the Hashtable or otherwise modify its internal structure (e.g.,
	  * rehash).  This field is used to make iterators on Collection-views of
	  * the Hashtable fail-fast.  (See ConcurrentModificationException).
	  * 结构性变动时modCount计数+1，用于遍历时的fail-fast机制生效
	  */
	private transient int modCount = 0;  

> 分析：可以看到，这些全局变量和HashMap的基本上是如出一辙  

### 1.3 构造器  

	/**
	 * initialCapacity 默认11
	 * loadFactor 默认 0.75
	 */
	public Hashtable() {
	    this(11, 0.75f);
	}
	/**
	  * 跟JDK1.7的HashMap基本一致
	  * @param initialCapacity 容量默认11
	  * @param loadFactor 负载因子默认0.75
	  */
	public Hashtable(int initialCapacity, float loadFactor) {
	    if (initialCapacity < 0)
	        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
	        if (loadFactor <= 0 || Float.isNaN(loadFactor))
	            throw new IllegalArgumentException("Illegal Load: "+loadFactor);
	        //区别在于没有强制令cap为2次幂，当initCap=0时，默认为1
	        if (initialCapacity==0)
	            initialCapacity = 1;
	        this.loadFactor = loadFactor;
	        table = new Entry[initialCapacity];
	        threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
	        //useAltHashing为boolean，其如果为真，则执行另一散列的字符串键，以减少由于弱哈希计算导致的哈希冲突的发生
	        useAltHashing = sun.misc.VM.isBooted() &&
	                (initialCapacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
	}  

> 分析：构造器和HashMap的做法也基本一样，只是有几个地方不同：默认的initialCapacity是11，而HashMap为16，这里在维护容量的时候**没有要求其为2的幂**，这也就导致了在通过hash值映射桶的下标的时候**只能通过"%"运算**来进行，相较于**HashMap中的"&"运算**，性能差一些。  

## 2. 重要方法  
### 2.1 put  

	public synchronized V put(K key, V value) {
	    // Make sure the value is not null，而HashMap选择将key为null永远存放为table[0]位置
	    if (value == null) {
	        throw new NullPointerException();
	    }
	    // Makes sure the key is not already in the hashtable.
	    //确保key不在hashtable中
	    //首先，通过hash方法计算key的哈希值，并计算得出index值，确定其在table[]中的位置
	    //其次，迭代index索引位置的链表，如果该位置处的链表存在相同的key，则替换value，返回旧的value
	    Entry tab[] = table;
	    int hash = hash(key);
	    //计算下标，这里使用%方法，性能远不及HashMap的位运算 (这也是不推荐使用HashTable的原因之一)
	    int index = (hash & 0x7FFFFFFF) % tab.length;
	    for (Entry<K,V> e = tab[index] ; e != null ; e = e.next) {
	        //直接使用equals比较，而HashMap多了一层地址比较 `((k = e.key) == key || key.equals(k))`
	        if ((e.hash == hash) && e.key.equals(key)) {
	            V old = e.value;
	            e.value = value;
	            return old;
	        }
	    }
	    //结构性变更操作 modCount计数+1
	    modCount++;
	    //HashMap选择新增addEntry方法封装一下逻辑
	    if (count >= threshold) {
	        // Rehash the table if the threshold is exceeded
	        //如果超过阀值，就进行rehash操作
	        rehash();
	        tab = table;
	        hash = hash(key);
	        index = (hash & 0x7FFFFFFF) % tab.length;
	    }
	    // Creates the new entry.
	    //将值插入，返回的为null
	    Entry<K,V> e = tab[index];
	    // 创建新的Entry节点，并将新的Entry插入Hashtable的index位置，并设置e为新的Entry的下一个元素
	    tab[index] = new Entry<>(hash, key, value, e);
	    //size++
	    count++;
	    return null;
	}  

> 分析：首先，**HashTable不支持key为null的情况**；由于在维护容量的时候**没有要求其为2的幂**，这也就导致了在通过hash值映射桶的下标的时候**只能通过"%"运算**来进行，相较于**HashMap中的"&"运算**，性能差一些。  

### 2.2 rehash方法  

	protected void rehash() {
	    int oldCapacity = table.length;
	    Entry<K,V>[] oldMap = table;//使用临时拷贝，保证当前数据时效性(参见JAVA的`观察者`模式实现)
	    // overflow-conscious code
	    //原容量的2倍+1
	    int newCapacity = (oldCapacity << 1) + 1;
	    if (newCapacity - MAX_ARRAY_SIZE > 0) {
	        if (oldCapacity == MAX_ARRAY_SIZE)
	            // Keep running with MAX_ARRAY_SIZE buckets
	            return;
	        newCapacity = MAX_ARRAY_SIZE;
	    }
	    Entry<K,V>[] newMap = new Entry[newCapacity];
	    //rehash也属于结构化变更，modCount计数+1
	    modCount++;
	    threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
	    boolean currentAltHashing = useAltHashing;
	    useAltHashing = sun.misc.VM.isBooted() && (newCapacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
	    boolean rehash = currentAltHashing ^ useAltHashing;
	    table = newMap;
	    //遍历原数组重新赋值给新数组
	    for (int i = oldCapacity ; i-- > 0 ;) {
	        for (Entry<K,V> old = oldMap[i] ; old != null ; ) {
	            Entry<K,V> e = old;
	            old = old.next;
	            if (rehash) {
	                e.hash = hash(e.key);//重新hash计算
	            }
	            //还是坑爹的%运算
	            int index = (e.hash & 0x7FFFFFFF) % newCapacity;
	            e.next = newMap[index];
	            newMap[index] = e;
	        }
	    }
	}  

### 2.3 get方法  

	public synchronized V get(Object key) {
	    Entry tab[] = table;//使用临时拷贝，保证当前数据时效性(参见JAVA的`观察者`模式实现)
	    int hash = hash(key);
	    int index = (hash & 0x7FFFFFFF) % tab.length;
	    for (Entry<K,V> e = tab[index] ; e != null ; e = e.next) {
	        if ((e.hash == hash) && e.key.equals(key)) {
	            return e.value;
	        }
	    }
	    return null;
	}  

> 分析：注意，该方法使用了synchronized修饰，是一个线程安全的方法  

### 2.4 Hashtable的迭代  

	//1、使用keys()
	Enumeration<String> en1 = table.keys();
	    while(en1.hasMoreElements()) {
	    en1.nextElement();
	}
	//2、使用elements()
	Enumeration<String> en2 = table.elements();
	    while(en2.hasMoreElements()) {
	    en2.nextElement();
	}
	//3、使用keySet()
	Iterator<String> it1 = table.keySet().iterator();
	    while(it1.hasNext()) {
	    it1.next();
	}
	//4、使用entrySet()
	Iterator<Entry<String, String>> it2 = table.entrySet().iterator();
	    while(it2.hasNext()) {
	    it2.next();
	}  

# 四、LinkedHashMap  
## 1. LinkedHashMap的数据结构   
### 1.1 类定义  

	/**
	  * 充分利用 `多态`，和HashMap操作数据的方法完全一样
	  */
	public class LinkedHashMap<K,V>
	    extends HashMap<K,V>
	    implements Map<K,V>  

> 分析：这个类继承了HashMap  

### 1.2 重要全局变量  

	/**
	  * The head of the doubly linked list.
	  * 新增一个双向链表维护所有元素，用于排序(header为链表表头)；Entry为LinkedHashMap的私有静态内部类实现
	  * header相当于一个串联器，header.before为尾元素，header.after为首元素
	  */
	private transient Entry<K,V> header;
	/**
	  * The iteration ordering method for this linked hash map: <tt>true</tt>
	  * for access-order, <tt>false</tt> for insertion-order.
	  * 该值为true：访问顺序（最近访问元素移至链表尾部）
	  * 该值为false：插入顺序 (先入在前，依次往后)
	  */
	private final boolean accessOrder;  

> 分析：这里的header是LinkedHashMap自己定义的一个类，继承自HashMap的同名的类，**这里的header变量只是用来维护一个双向链表来维护顺序，数据还是存储在了桶中**  

### 1.3 构造器  

	/**
	  * 1.直接复用HashMap构造器
	  * 2.排序方式默认插入顺序(先入在前，依次往后)
	  */
	public LinkedHashMap(int initialCapacity, float loadFactor) {
	    super(initialCapacity, loadFactor);
	    accessOrder = false;
	}
	/**
	  * @Param accessOrder false：插入顺序 ; true：访问顺序（最近访问元素移至链表尾部）
	  */
	public LinkedHashMap(int initialCapacity, float loadFactor,boolean accessOrder) {
	    super(initialCapacity, loadFactor);
	    this.accessOrder = accessOrder;  
	}

> 分析：构造器并没有什么特别的地方  

### 1.4 init方法  

	/**
	  * Called by superclass constructors and pseudoconstructors (clone,readObject)
	  * before any entries are inserted into the map.  Initializes the chain.
	  * 该方法采用模板方法模式，在 HashMap 的实现中并无意义，只是提供给子类实现相关的初始化调用
	  * LinkedHashMap会对header初始化
	  */
	@Override
	void init() {
	  header = new Entry<>(-1, null, null, null);
	  header.before = header.after = header;
	}  

> 分析：init方法只是初始化了header这个用来维护双向链表的变量  

* 假设header地址为0x00000000，那么init之后的header如图所示  

![](http://static.zybuluo.com/kiraSally/r66jylt2ha478zaxgfpzjxry/QQ%E6%88%AA%E5%9B%BE20170721134814.png)  

### 1.5 Entry  

	/**
	  * LinkedHashMap entry.
	  */
	private static class Entry<K,V> extends HashMap.Entry<K,V> {
	    // These fields comprise the doubly linked list used for iteration.
	    //before指向上一个entry，after指向下一个entry，从而形成双向链表，进而实现有序性
	    //迭代时直接沿双向链表遍历,因此迭代速度只与元素总量(size)有关，与容量(capacity)无关
	    Entry<K,V> before, after;
	    Entry(int hash, K key, V value, HashMap.Entry<K,V> next) {
	        super(hash, key, value, next);
	    }
	    /**
	      * Removes this entry from the linked list.
	      */
	    private void remove() {
	        before.after = after;
	        after.before = before;
	    }
	    /**
	      * Inserts this entry before the specified existing entry in the list.
	      * 新entry插入到双向链表头引用header的前面，这样新entry就成为双向链表中的最后一个元素
	      */
	    private void addBefore(Entry<K,V> existingEntry) {
	        //需要注意的是existingEntry=header
	        after  = existingEntry;
	        before = existingEntry.before;
	        before.after = this;
	        after.before = this;
	    }
	    /**
	      * This method is invoked by the superclass whenever the value
	      * of a pre-existing entry is read by Map.get or modified by Map.set.
	      * If the enclosing Map is access-ordered, it moves the entry
	      * to the end of the list; otherwise, it does nothing.
	      */
	    void recordAccess(HashMap<K,V> m) {
	        LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
	        if (lm.accessOrder) {
	            lm.modCount++;
	            remove();
	            addBefore(lm.header);
	        }
	    }
	    //模板方法
	    void recordRemoval(HashMap<K,V> m) {
	        remove();
	    }
	}  

> 分析：LinkedHashMap内部定义的这个类继承了HashMap类内部的同名类；值得注意的是**addBefore方法传入的参数在调用的时候始终传入的是LinkedHashMap里的那个header变量，也就是说双向链表的header的before代表表尾**；这个类定义的一个出彩的地方是**before和after在双向链表的维护中使用；而同时也可以直接加入到桶中，使用next变量**  

## 2. LinkHashMap的迭代顺序  
### 2.1 插入顺序  

	LinkedHashMap<Integer,String> linkedHashMap = new LinkedHashMap<Integer,String>();
    linkedHashMap.put(1,"新恒结衣");
    linkedHashMap.put(2,"长泽雅美");
    linkedHashMap.put(3,"佐佐木希");
    linkedHashMap.put(4,"石原里美");
    linkedHashMap.put(5,"堀北真希");
    Iterator iterator = linkedHashMap.entrySet().iterator();
    while (iterator.hasNext()) {
        Map.Entry entry = (Map.Entry) iterator.next();
        System.out.println(entry.getKey() + "=" + entry.getValue());
    }  

* 如图所示，按照插入顺序(先入在前，依次往后)  

![](http://static.zybuluo.com/kiraSally/emd4bgtdu3jxx2x7y4q0gfeh/QQ%E6%88%AA%E5%9B%BE20170721170305.png)  

* 注意：header的key-value永远为null，但before和after已经发生更改  

![](http://static.zybuluo.com/kiraSally/p6qurc3pril53b2gadb58r8p/QQ%E6%88%AA%E5%9B%BE20170721173256.png)  

* header的before指向链尾元素(结合after从而形成双向环路)   

![](http://static.zybuluo.com/kiraSally/ba3mejdwqk05q6rir5ew8agt/QQ%E6%88%AA%E5%9B%BE20170721173303.png)  

* header的after指向链首元素（除去header）  

![](http://static.zybuluo.com/kiraSally/d42bp4a9z0ae4irpavz1age1/QQ%E6%88%AA%E5%9B%BE20170721173313.png)  

> 分析：可以看到，在**插入顺序模式**下，双向链表维护的顺序确实是满足**先插在前，依次往后**的，而且**header.after代表表尾**  

### 2.2 访问顺序  

	//注意：访问排序生效需要 accessOrder = true
	LinkedHashMap<Integer,String> linkedHashMap = new LinkedHashMap<Integer,String>(16,0.75f,true);
	    linkedHashMap.put(1,"新恒结衣");
	    linkedHashMap.put(2,"长泽雅美");
	    linkedHashMap.put(3,"佐佐木希");
	    linkedHashMap.put(4,"石原里美");
	    linkedHashMap.put(5,"堀北真希");
	    //访问开始
	    linkedHashMap.get(3);
	    linkedHashMap.get(1);
	    Iterator iterator = linkedHashMap.entrySet().iterator();
	    while (iterator.hasNext()) {
	        Map.Entry entry = (Map.Entry) iterator.next();
	        System.out.println(entry.getKey() + "=" + entry.getValue());
	    }  

* 如图所示，按照访问顺序(最近访问元素移至链表尾部)   

![](http://static.zybuluo.com/kiraSally/1aff2zg3c5rlfg08aruok006/QQ%E6%88%AA%E5%9B%BE20170721173734.png)  

* header的before指向链尾元素  

![](http://static.zybuluo.com/kiraSally/fxvna43nr6rj4nlh3lpp9dmg/QQ%E6%88%AA%E5%9B%BE20170721173814.png)  

* header的after指向链首元素   

![](http://static.zybuluo.com/kiraSally/vnliofy64d5fqizf9xqs08g8/QQ%E6%88%AA%E5%9B%BE20170721173824.png)  

## 3. LinkedHashMap的重要方法  
### 3.1 put方法  

	 /**
	  * LinkedHashMap并未重写父类`HashMap`的put方法（因此不要惊讶找不到LinkedHashMap的put方法）
	  * 但重写了父类 HashMap 的 put 方法调用的子方法，以增加双向链表的实现
	  * 重写以下子方法 : recordAccess() 、addEntry()、createEntry()
	  */
	public V put(K key, V value) {
	    if (key == null)
	        return putForNullKey(value);
	    int hash = hash(key);
	    int i = indexFor(hash, table.length);
	    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
	        Object k;
	        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
	            V oldValue = e.value;
	            e.value = value;
	            //LinkedHashMap的Entry重写父类recordAccess方法
	            e.recordAccess(this);
	            return oldValue;
	        }
	    }
	    modCount++;
	    //LinkedHashMap重写父类recordAccess方法
	    addEntry(hash, key, value, i);
	    return null;
	}  

> 分析：正如注释中所说，**LinkedHashMap只是重写了父类的recordAccess子方法(其他的改变在该类的addEntry中也有体现)，并没有重写put方法**  

### 3.2 recordAccess方法  

	/**
	  * 记录访问顺序
	  */
	void recordAccess(HashMap<K,V> m) {
	    LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
	    // 如果定义了LinkedHashMap的迭代顺序为访问顺序，
	    // 则删除以前位置上的元素，并将最新访问的元素添加到链表表头
	    if (lm.accessOrder) {
	        //注意：结构性变动会使得modCount计数+1，用于fail-fast机制
	        lm.modCount++;
	        //变更链表的前后引用，保证有序性
	        remove();
	        //将最新访问的元素添加到链表表头
	        addBefore(lm.header);
	    }
	}  

> 分析：该方法只是在**访问模式**为访问顺序时被调用，用来维护该模式的规则  

### 3.3 addEntry方法  

		void addEntry(int hash, K key, V value, int bucketIndex) {
	    // 调用create方法，将新元素以双向链表的的形式加入到映射中
	    createEntry(hash, key, value, bucketIndex);
	    // 删除最近最少使用元素的策略定义
	    Entry<K,V> eldest = header.after;
	    //该方法默认返回false，也就意味着LinkedHashMap不会主动删除"过期"元素
	    if (removeEldestEntry(eldest)) { 
	        //直接调用父类HashMap的removeEntryForKey()
	        removeEntryForKey(eldest.key);
	    } else {
	        //超过阈值执行resize
	        if (size >= threshold)
	            //直接调用父类HashMap的resize方法，容量翻倍
	            resize(2 * table.length);
	    }
	}  


### 3.4 createEntry方法  

		void createEntry(int hash, K key, V value, int bucketIndex) {
	    HashMap.Entry<K,V> old = table[bucketIndex];
	    Entry<K,V> e = new Entry<K,V>(hash, key, value, old);
	    table[bucketIndex] = e;
	    //将元素加入到哈希、双向链表
	    e.addBefore(header);
	    size++;
	}  

> 分析：**e在创建的时候将old设置为next，添加入桶中；同时通过e.addBefore方法添加到双向链表中**  

### 3.5 get方法  

	/**
	  * 记录访问顺序，将最新访问的元素添加到双向链表的表头，并从原来的位置删除
	  * 链表的增加、删除操作是常量级
	  */
	public V get(Object key) {  
	    // 调用父类HashMap的getEntry()方法，取得要查找的元素  
	    Entry<K,V> e = (Entry<K,V>)getEntry(key);  
	    if (e == null)  
	        return null;  
	    // 记录访问顺序  
	    e.recordAccess(this);  
	    return e.value;  
	}   

### 3.6 removeEntryForKey方法  

	/**
	  * LinkedHashMap没有重写removeEntryForKey，而是使用模板方法模式巧妙的实现该方法
	  * 以下为HashMap的removeEntryForKey方法实现
	  */
	final Entry<K,V> removeEntryForKey(Object key) {
	    int hash = (key == null) ? 0 : hash(key);
	    int i = indexFor(hash, table.length);// hash&(table.length-1)
	    Entry<K,V> prev = table[i];// 得到冲突链表
	    Entry<K,V> e = prev;
	    while (e != null) {// 遍历冲突链表
	        Entry<K,V> next = e.next;
	        Object k;
	        if (e.hash == hash &&
	            ((k = e.key) == key || (key != null && key.equals(k)))) {// 找到要删除的entry
	            modCount++; size--;
	            // 1. 将e从对应bucket的冲突链表中删除
	            if (prev == e) table[i] = next;
	            else prev.next = next;
	            // 2. 将e从双向链表中删除  HashMap中的实际实现为 e.recordRemoval(this)
	            //等用于 before.after = after; after.before = before;
	            e.recordRemoval(this);
	            return e;
	        }
	        prev = e; e = next;
	    }
	    return e;
	}
	/**
	  * LinkedHashMap的删除实现
	  */
	void recordRemoval(HashMap<K,V> m) {
	    remove();
	}
	/**
	  * entry内部执行remove操作时，通过变更链表前后指向删除以前位置上的元素，保证迭代有序性
	  */
	private void remove() {
	    before.after = after;
	    after.before = before;
	}  

### 3.7 clear方法  

	public void clear() {
	    super.clear();
	    //header的所有引用重新指向自己，等同于位置reset
	    header.before = header.after = header;
	}  

> 分析：可见，clear方法不光要通过super的同名方法将map清空，还要把双向链表清空。