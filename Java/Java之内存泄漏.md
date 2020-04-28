## 如果长生命周期的对象持有短生命周期的引用，就很可能会出现内存泄露

比如下面的代码，这里的object实例，其实我们期望它只作用于method1()方法中，且其他地方不会再用到它，但是，当method1()方法执行完成后，object对象所分配的内存不会马上被认为是可以被释放的对象，只有在Simple类创建的对象被释放后才会被释放，严格的说，这就是一种内存泄露。

```
public class Simple {
 
    Object object;
 
    public void method1(){
        object = new Object();
        //...其他代码
    }
}
```

怎么解决上面的问题呢，加上下面的蓝色代码注释就好了

```
public class Simple {
 
    Object object;
 
    public void method1(){
        object = new Object();
        //...其他代码
        // 蓝色代码注释开始
        object = null;
        // 蓝色代码注释结束
    }
}
```

## 集合里面的内存泄漏

#### 集合里面的数据都设置成null，但是集合内存还是存在的

比如下面的代码
因为你已经在下面的蓝色代码注释里面进行company=null了，所以下面的list集合里面的数据都是无用的了，但是此时list集合里面的所有元素都不会进行垃圾回收

```java
package com.four;

import java.util.ArrayList;
import java.util.List;

public class Hello {
    public static void main(String[] args) {
        List<Company> list = new ArrayList<Company>();
        int i=0;
        for(int j=0;j<10;j++){
            Company company = new Company();
            company.setName("ali");
            list.add(company);
            // 蓝色代码注释开始
            company = null;
            // 蓝色代码注释结束
        }

        System.gc();
        while(true){
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("已经测试了"+(++i)+"秒");
        }
    }

}


class Company {
    private String name;

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("回收Comapny");
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

怎么解决上面的问题呢，就是把上面的list集合变量也变成null，比如加上下面的红色代码注释

## 还有就是使用remove()方法进行移除元素的时候，也可能会造成内存泄漏

什么意思呢，
就比如ArrayList里面的pop()，如果是下面的写法就会造成内存泄漏，因为下面的elementData[--size]这个元素移除之后，并没有进行设置成null

```java
public E pop(){
    if(size == 0)
        return null;
    else
        return (E) elementData[size];
}
```

所以上面的代码应该变成下面这样，此时注意下面的蓝色代码注释里面的size值比下面的红色代码注释里面的size小1

```java
public E pop(){
    if(size == 0)
        return null;
    else{
        // 红色代码注释开始
        E e = (E) elementData[--size];
        // 红色代码注释结束
        // 蓝色代码注释开始
        elementData[size] = null;
        // 蓝色代码注释结束
        return e;
    }
}
```

## 连接没有关闭会泄漏

比如数据库连接（dataSourse.getConnection()），网络连接(socket)和io连接，这些链接在使用的时候，除非显式的调用了其close()方法（或类似方法）将其连接关闭，否则是不会自动被GC回收的。其实原因依然是长生命周期对象持有短生命周期对象的引用。所以我们经常在网上看到在连接调用结束的时候要进行调用close()进行关闭，这样可以回收不用的内存对象，增加可用内存。

### finalize()

finalize()是Object类中的方法。

​    了解C++的都知道有个析构函数，但是注意，finalize()绝不等于C++中的析构函数。

​    Java编程思想中是这么解释的：一旦GC准备好释放对象所占用的的存储空间，将先调用其finalize()方法，并在下一次GC回收动作发生时，才会真正回收对象占用的内存，所以一些清理工作，我们可以放到finalize()中。

​    该方法的一个重要的用途是：当在java中调用非java代码（如c和c++）时，在这些非java代码中可能会用到相应的申请内存的操作（如c的malloc()函数），而在这些非java代码中并没有有效的释放这些内存，就可以使用finalize()方法，并在里面调用本地方法的free()等函数。

​    所以finalize()并不适合用作普通的清理工作。

​    不过有时候，该方法也有一定的用处：

​    如果存在一系列对象，对象中有一个状态为false，如果我们已经处理过这个对象，状态会变为true，为了避免有被遗漏而没有处理的对象，就可以使用finalize()方法：

```java

class MyObject{
    boolean state = false;
    public void deal(){
        //...一些处理操作
        state = true;
    }
    @Override
    protected void finalize(){
        if(!state){
            System.out.println("ERROR:" + "对象未处理！");
        }
    }
    //...
}
```

- 在C++中，对象是可以在栈上分配的，也可以在堆上分配。在栈上分配的对象，也就是函数的局部变量，当超出块的"}"时，生命期便结束了。在堆上分配的对象，使用delete的时候，对象的生命期也就结束了。因此**在C++中，对象的内存在哪个时刻被回收，是可以确定的**（假设程序没有缺陷）
- java秉承一切皆为对象的思想，对象仅能通过new来创建，因此java的对象是在堆上分配的内存。这些堆上的对象，如果没有作用了（无引用指向它），将等待垃圾回收器来回收其占用的内存。而垃圾回收期何时运行，无法提前预知，甚至有的时候直到程序退出都没有进行垃圾回收，程序所占内存直接由操作系统来回收。所以**在java中，对象的内存在哪个时刻回收，取决于垃圾回收器何时运行。**因此，C++与java中，对无用对象的回收时间是不同的。
- 一旦C++的对象要被回收了，在回收该对象之前对象的析构函数将被调用，然后释放对象占用的内存；而java中一旦垃圾回收器准备好释放对象占用的存储空间，将首先调用其finalize()方法， 并且在**下一次**垃圾回收动作发生时，才会**真正的**回收对象占用的内存
- 可见在java中，**调用GC不等于真正地回收内存资源**，而且在垃圾回收中对象存在状态的变化。
- C++的析构函数用来做一些必要的工作，例如释放掉指针成员所指向的对象所占的内存，因为C++没有java的垃圾回收器，所有new出来的对象，都要显式地delete掉，避免内存泄漏。《Effective C++》中提及，基类需要将析构函数声明为virtual函数，这是为了可以通过子类对象指针正确地释放掉基类的资源。总的来说，**在C++中，析构函数和资源的释放息息相关，能不能正确处理析构函数，关乎能否正确回收对象内存资源。**
- 在java中，所有的对象，包括对象中包含的其他对象，它们所占的内存的回收都依靠垃圾回收器，因此不需要一个函数如C++析构函数那样来做必要的垃圾回收工作。当然存在本地方法时需要finalize()方法来清理本地对象。在《java编程思想》中提及，finalize()方法的一个作用是用来回收“本地方法”中的本地对象——C/C++代码所分配的内存，由于这部分的内存只能由delete/free来释放，因此可以放在finalize()方法中来做。在实际生产环境中，我较少（或说基本没有）看到java类实现了finalize()方法。可以说java最大程度地弱化了内存管理对应用程序员的束缚，而c++则对此要求严格多了。