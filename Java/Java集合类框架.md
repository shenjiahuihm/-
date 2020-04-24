# Java集合类框架

## **List接口**

List接口对Collection进行了简单的扩充

```java
...<br> * @see AbstractList
 * @see AbstractSequentialList
 * @since 1.2
 */
 
public interface List<E> extends Collection<E> {
    // Query Operations
 
    /**
     * Returns the number of elements in this list.  If this list contains
     * more than <tt>Integer.MAX_VALUE</tt> elements, returns
     * <tt>Integer.MAX_VALUE</tt>.
<br>   ...
```

List接口中的元素的特点为:

List中存储的元素实现类排序，而且可以重复的存储相关元素。

同时List接口又有两个常用的实现类ArrayList和LinkedList

### 1. ArrayList

ArrayList数组线性表的特点为:类似数组的形式进行存储，因此它的随机访问速度极快。

ArrayList数组线性表的缺点为:不适合于在线性表中间需要频繁进行插入和删除操作。因为每次插入和删除都需要移动数组中的元素。

可以这样理解ArrayList就是基于数组的一个线性表，只不过数组的长度可以动态改变而已。

- 如果在初始化ArrayList的时候没有指定初始化长度的话，默认的长度为10.

  ```
  /**
       * Constructs an empty list with an initial capacity of ten.
       */
      public ArrayList() {
      this(10);
      }
  ```

- ArrayList在增加新元素的时候如果超过了原始的容量的话，ArrayList扩容ensureCapacity的方案为“原始容量*3/2+1"

```java
/**
 * Increases the capacity of this <tt>ArrayList</tt> instance, if
 * necessary, to ensure that it can hold at least the number of elements
 * specified by the minimum capacity argument.
 *
 * @param   minCapacity   the desired minimum capacity
 */
public void ensureCapacity(int minCapacity) {
modCount++;
int oldCapacity = elementData.length;
if (minCapacity > oldCapacity) {
    Object oldData[] = elementData;
    int newCapacity = (oldCapacity * 3)/2 + 1;
        if (newCapacity < minCapacity)
    newCapacity = minCapacity;
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
}
}
```

- ArrayList是线程不安全的，在多线程的情况下不要使用。

  如果一定在多线程使用List的，您可以使用Vector，因为Vector和ArrayList基本一致，区别在于Vector中的绝大部分方法都使用了同步关键字修饰，这样在多线程的情况下不会出现并发错误哦，还有就是它们的扩容方案不同，ArrayList是通过原始容量*3/2+1,而Vector是允许设置默认的增长长度，Vector的默认扩容方式为原来的2倍。

  切记Vector是ArrayList的多线程的一个替代品。

- ArrayList实现遍历的几种方法

  ```java
  package com.yonyou.test;
   
  import java.util.ArrayList;
  import java.util.Iterator;
  import java.util.List;
   
   
  /**
   * 测试类
   * @author 小浩
   * @创建日期 2015-3-2
   */
   
  public class Test{
  public static void main(String[] args) {
       List<String> list=new ArrayList<String>();
       list.add("Hello");
       list.add("World");
       list.add("HAHAHAHA");
       //第一种遍历方法使用foreach遍历List
       for (String str : list) {            //也可以改写for(int i=0;i<list.size();i++)这种形式
          System.out.println(str);
      }
   
       //第二种遍历，把链表变为数组相关的内容进行遍历
      String[] strArray=new String[list.size()];
      list.toArray(strArray);
      for(int i=0;i<strArray.length;i++) //这里也可以改写为foreach(String str:strArray)这种形式
      {
          System.out.println(strArray[i]);
      }
       
      //第三种遍历 使用迭代器进行相关遍历
       
       Iterator<String> ite=list.iterator();
       while(ite.hasNext())
       {
           System.out.println(ite.next());
       }
   }
  }
  ```

  ### 2. LinkedList

  LinkedList的链式线性表的特点为: 适合于在链表中间需要频繁进行插入和删除操作。

  LinkedList的链式线性表的缺点为: 随机访问速度较慢。查找一个元素需要从头开始一个一个的找。速度你懂的。

  可以这样理解LinkedList就是一种双向循环链表的链式线性表，只不过存储的结构使用的是链式表而已。

- LinkedList和ArrayList的区别和联系

  ArrayList数组线性表的特点为:

  类似数组的形式进行存储，因此它的随机访问速度极快。

  ArrayList数组线性表的缺点为:不适合于在线性表中间需要频繁进行插入和删除操作。因为每次插入和删除都需要移动数组中的元素。

  LinkedList的链式线性表的特点为: 适合于在链表中间需要频繁进行插入和删除操作。

  LinkedList的链式线性表的缺点为: 随机访问速度较慢。查找一个元素需要从头开始一个一个的找。

- LinkedList的内部实现

  LinkedList的内部是基于双向循环链表的结构来实现的。在LinkedList中有一个类似于c语言中结构体的Entry内部类。

  ​         在Entry的内部类中包含了前一个元素的地址引用和后一个元素的地址引用类似于c语言中指针。

- LinkedList不是线程安全的

   注意LinkedList和ArrayList一样也不是线程安全的，如果在对线程下面访问可以自己重写LinkedList

  然后在需要同步的方法上面加上同步关键字synchronized

- LinkedList的遍历方式

  ```
  package com.yonyou.test;
   
  import java.util.LinkedList;
  import java.util.List;
   
   
  /**
   * 测试类
   * @author 小浩
   * @创建日期 2015-3-2
   */
   
  public class Test{
  public static void main(String[] args) {
       
      List<String> list=new LinkedList<String>();
      list.add("Hello");
      list.add("World");
      list.add("龙不吟，虎不啸");
      //LinkedList遍历的第一种方式使用数组的方式
      String[] strArray=new String[list.size()];
      list.toArray(strArray);
      for(String str:strArray)
      {
          System.out.println(str);
      }
      //LinkedList遍历的第二种方式
      for(String str:list)
      {
          System.out.println(str);   
      }
      //至于还是否有其它遍历方式，我没查，感兴趣自己研究研究
               
   
    }
  }
  ```

- LinkedList可以被当做堆栈来使用

## 2. **Set接口**

Set接口也是Collection接口的一个常用子接口。

```java
* @see Collections#EMPTY_SET
 * @since 1.2
 */
 
public interface Set<E> extends Collection<E> {
    // Query Operations
 
    /**
     * Returns the number of elements in this set (its cardinality).  If this
     * set contains more than <tt>Integer.MAX_VALUE</tt> elements, returns
```

Set接口区别于List接口的特点在于:

Set中的元素实现了不重复，有点象集合的概念，无序，不允许有重复的元素,最多允许有一个null元素对象。

需要注意的是:虽然Set中元素没有顺序，但是元素在set中的位置是有由该元素的HashCode决定的，其具体位置其实是固定的。

此外需要说明一点，在set接口中的不重复是由特殊要求的。

举一个例子:对象A和对象B，本来是不同的两个对象，正常情况下它们是能够放入到Set里面的，但是

如果对象A和B的都重写了hashcode和equals方法，并且重写后的hashcode和equals方法是相同的话。那么A和B是不能同时放入到

Set集合中去的，也就是Set集合中的去重和hashcode与equals方法直接相关。

为了更好的理解，请看下面的例子:

```java
package com.yonyou.test;
 
import java.util.HashSet;
import java.util.Set;
 
 
/**
 * 测试类
 * @author 小浩
 * @创建日期 2015-3-2
 */
 
public class Test{
public static void main(String[] args) {
     Set<String> set=new HashSet<String>();
     set.add("Hello");
     set.add("world");
     set.add("Hello");
     System.out.println("集合的尺寸为:"+set.size());
     System.out.println("集合中的元素为:"+set.toString());
  }
}
```

由于String类中重写了hashcode和equals方法，所以这里的第二个Hello是加不进去的哦。

Set接口的常见实现类有HashSet,LinedHashSet和TreeSet这三个。下面我们将分别讲解这三个类。

### 1. HashSet

HashSet是Set接口的最常见的实现类了。其底层是基于Hash算法进行存储相关元素的。

下面是HashSet的部分源码:

```java
/**
    * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
    * default initial capacity (16) and load factor (0.75).
    */
   public HashSet() {
   map = new HashMap<E,Object>();
   }
```

你看到了什么，没错，对于HashSet的底层就是基于HashMap来实现的哦。

我们都知道在HashMap中的key是不允许重复的，你换个角度看看，那不就是说Set集合吗？

这里唯一一个需要处理的就是那个Map的value弄成一个固定值即可。

看来一切水到渠成啊~哈哈~这里的就是Map中的Key。

对于HashMap和Hash算法的讲解会在下面出现，先别着急，下面继续讲解HashSet。

下面讲解一下HashSet使用和理解中容易出现的误区:

- HashSet中存放null值

  HashSet中时允许出入null值的，但是在HashSet中仅仅能够存入一个null值哦。

- HashSet中存储元素的位置是固定的

  HashSet中存储的元素的是无序的，这个没什么好说的，但是由于HashSet底层是基于Hash算法实现的，使用了hashcode，所以HashSet中相应的元素的位置是固定的哦。

- 遍历HashSet的几种方法

  ```java
  package com.yonyou.test;
   
  import java.util.HashSet;
  import java.util.Iterator;
  import java.util.Set;
   
   
  /**
   * 测试类
   * @author 小浩
   * @创建日期 2015-3-2
   */
   
  public class Test{
  public static void main(String[] args) {
       Set<String> set=new HashSet<String>();
       set.add("Hello");
       set.add("world");
       set.add("Hello");
       //遍历集合的第一种方法，使用数组的方法
       String[] strArray=new String[set.size()];
       strArray=set.toArray(strArray);
       for(String str:strArray)//此处也可以使用for(int i=0;i<strArray.length;i++)
       {
           System.out.println(str);
       }
       //遍历集合的第二中方法，使用set集合直接遍历
       for(String str:set)
       {
           System.out.println(str);
       }
        
       //遍历集合的第三种方法，使用iterator迭代器的方法
       Iterator<String> iterator=set.iterator();
       while(iterator.hasNext())
       {
           System.out.println(iterator.next());
       }
  }
  }
  ```

### 2. LinkHashSet

LinkHashSet不仅是Set接口的子接口而且还是上面HashSet接口的子接口。

查看LinkedHashSet的部分源码如下:

```java
...<br> * @see     Hashtable
 * @since   1.4
 */
 
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
 
    private static final long serialVersionUID = -2851667679971038690L;
 
    /**
     * Constructs a new, empty linked hash set with the specified initial
<br>  ...
```

这里就可以发现Set接口时HashSet接口的一个子接口了吧~

通过查看LinkedHashSet的源码可以发现,其底层是基于LinkedHashMap来实现的哦。

LinkedHashSet中存储的元素是在哈希算法的基础上增加了链式表的结构。

### 3. TreeSet

TreeSet是一种排序二叉树。存入Set集合中的值，会按照值的大小进行相关的排序操作。底层算法是基于红黑树来实现的。

TreeSet和HashSet的主要区别在于TreeSet中的元素会按照相关的值进行排序~

**TreeSet和HashSet的区别和联系**

1. HashSet是通过HashMap实现的,TreeSet是通过TreeMap实现的,只不过Set用的只是Map的key
2. Map的key和Set都有一个共同的特性就是集合的唯一性.TreeMap更是多了一个排序的功能.
3. hashCode和equal()是HashMap用的, 因为无需排序所以只需要关注定位和唯一性即可.
   1. hashCode是用来计算hash值的,hash值是用来确定hash表索引的.
   2. hash表中的一个索引处存放的是一张链表, 所以还要通过equal方法循环比较链上的每一个对象才可以真正定位到键值对应的Entry.
   3. put时,如果hash表中没定位到,就在链表前加一个Entry,如果定位到了,则更换Entry中的value,并返回旧value
4. 由于TreeMap需要排序,所以需要一个Comparator为键值进行大小比较.当然也是用Comparator定位的.
   1. Comparator可以在创建TreeMap时指定
   2. 如果创建时没有确定,那么就会使用key.compareTo()方法,这就要求key必须实现Comparable接口.
   3. TreeMap是使用Tree数据结构实现的,所以使用compare接口就可以完成定位了.

下面是一个使用TreeSet的实例:

```
package com.yonyou.test;
 
import java.util.Iterator;
import java.util.TreeSet;
 
/**
 * TreeSet测试类
 * @author 小浩
 * @创建日期 2015-4-3
 */
public class Test{
    public static void main(String[] args) {
        //String实体类中实现Comparable接口，所以在初始化TreeSet的时候，
        //无需传入比较器
        TreeSet<String> treeSet=new TreeSet<String>();
        treeSet.add("d");
        treeSet.add("c");
        treeSet.add("b");
        treeSet.add("a");
        Iterator<String> iterator=treeSet.iterator();
        while(iterator.hasNext())
        {
            System.out.println(iterator.next());
        }
         
     }
}
```

## 3. **Map接口**

说到Map接口的话大家也许在熟悉不过了。Map接口实现的是一组Key-Value的键值对的组合。 Map中的每个成员方法由一个关键字（key）和一个值（value）构成。Map接口不直接继承于Collection接口（需要注意啦），因为它包装的是一组成对的“键-值”对象的集合，而且在Map接口的集合中也不能有重复的key出现，因为每个键只能与一个成员元素相对应。在我们的日常的开发项目中，我们无时无刻不在使用者Map接口及其实现类。Map有两种比较常用的实现：HashMap和TreeMap等。HashMap也用到了哈希码的算法，以便快速查找一个键，TreeMap则是对键按序存放，因此它便有一些扩展的方法，比如firstKey(),lastKey()等，你还可以从TreeMap中指定一个范围以取得其子Map。键和值的关联很简单，用pub(Object key,Object value)方法即可将一个键与一个值对象相关联。用get(Object key)可得到与此key对象所对应的值对象。

​       另外前边已经说明了，Set接口的底层是基于Map接口实现的。Set中存储的值，其实就是Map中的key，它们都是不允许重复的。

Map接口的部分源码如下:

接下来我们讲解Map接口的常见实现类HashMap、TreeMap、LinkedHashMap、Properties(继承HashTable)以及老版本的HashTable等。

### 1. HashMap

HashMap实现了Map、CloneMap、Serializable三个接口，并且继承自AbstractMap类。

HashMap基于hash数组实现，若key的hash值相同则使用链表方式进行保存。

遍历方式：

```java
import java.util.*;
 
public class MapTraverse {
	public static void main(String[] args) {
		String[] str = {"I love you", "You love he", "He love her", "She love me"};
		Map<Integer, String> m = new HashMap();
		for(int i=0; i<str.length; i++) {
			m.put(i, str[i]);
		}
		System.out.println("下面是使用useWhileSentence()方法输出的结果：");
		useWhileSentence(m);
		System.out.println("下面是使用useWhileSentence2()方法输出的结果：");
		useWhileSentence2(m);
		System.out.println("下面是使用useForSentence()方法输出的结果：");
		useForSentence(m);
	}
	
	public static void useWhileSentence(Map<Integer, String> m) {
		Set s = (Set<Integer>)m.keySet();
		Iterator<Integer> it = s.iterator();
		int Key;
		String value;
		while(it.hasNext()) {
			Key = it.next();
			value = (String)m.get(Key);
			System.out.println(Key+":\t"+value);
		}
	}
	
	public static void useWhileSentence2(Map m) {
		Set s = m.entrySet();
		Iterator<Map.Entry<Integer, String>> it = s.iterator();
		Map.Entry<Integer, String> entry;
		int Key;
		String value;
		while(it.hasNext()) {
			entry = it.next();
			Key = entry.getKey();
			value = entry.getValue();
			System.out.println(Key+":\t"+value);
		}
	}
	
	public static void useForSentence(Map<Integer, String> m) {
		int Key;
		String value;
		for(Map.Entry<Integer, String> entry : m.entrySet()) {
			Key = entry.getKey();
			value = entry.getValue();
			System.out.println(Key+":\t"+value);
		}
	}
	
}
```

