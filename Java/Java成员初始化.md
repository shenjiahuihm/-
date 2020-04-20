#  Java成员初始化

## 一、方法的局部变量

​	Java尽力保证：所有变量在使用前都能得到恰当的初始化。对于方法的局部变量，Java以编译时错误的形式来贯彻这种保证。所以如果写成：

```java
void f(){
    int i;
    i++; // 错误，i没有初始化
}
```

就会得到一条出错消息，告诉你i可能尚未初始化。这样强制程序员提供一个初始值，往往能够帮助找出程序里的缺陷。(对于特殊的main()方法也是一样的)

## 二、 类的数据成员

​	要是类的数据成员（即字段）是基本类型，情况就会变得有些不同。

​	类的每个基本类型数据成员保证都会有一个初始值。

​	下面的程序可以验证这类情况，并显示它们的值：

```java
class InitialValues { 
    boolean t; 
    char c; 
    byte b; 
    short s; 
    int i; 
    long l; 
    float f; 
    double d; 
    void print() { 
        System.out.println( 
            "Data type Inital value\n" + 
            "boolean " + t + "\n" + 
            "char " + c + "\n" + 
            "byte " + b + "\n" + 
            "short " + s + "\n" + 
            "int " + i + "\n" + 
            "long " + l + "\n" + 
            "float " + f + "\n" + 
            "double " + d); 
    } 
    public static void main(String[] args) { 
        InitialValues iv = new InitialValues(); 
        iv.print(); 
        /* 你也可以这样写: new InitialValues().print(); */ 
    } 
}
```

​	输出结果如下：

```
Data type Inital value 
boolean false 
char []
byte 0 
short 0 
int 0 
long 0 
float 0.0 
double 0.0
```

​	可见尽管数据成员的初值没有给出，但它们确实有初值（char值为0，所以显示为空白）。这样至少不会冒“未初始化变量”的风险了。

​	在类里定义一个对象引用时，如果不将其初始化，此引用就会获得一个特殊值null。

## 三、指定初始化

如果想为某个变量赋初值，该怎么做呢？有一种很直接的办法，就是在定义类成员变量的地方为其赋初值。以下代码片段修改了**InitialValues**类成员变量的定义，直接提供了初值。

```java
public class InitialValues{
    boolean bool = true;
    char ch = 'x';
    byte b = 47;
    short s = 0xff;
    int i = 999;
    long lng = 1;
    float f =3.14f;
    double d =3.14159;
}
```

也可以用同样的方法初始化非基本类型的对象。如果Depth是一个类，那么可以像下面这样创建一个对象并初始化它：

```java
class Depth{}

public class Measurement{
	Depth d = new Depth();
    // ...
}
```

## 四、 构造器初始化

可以用构造器来进行初始化，这样使编程更加灵活。但要牢记：**无法阻止自动初始化的进行**，它将在构造器被调用之前发生。因此，假如使用以下代码：

```java
class Counter { 
    int i; 
    Counter() { i = 7; }
}// . . .
```

那么i首先会被置0，然后变成7。对于所有基本类型和对象引用，包括在定义时已经指定初值的变量，这种情况都是成立的。

### 初始化顺序

在类的内部，**变量定义的先后顺序决定了初始化的顺序**。即便变量定义散布于方法定义之间，他们仍旧会在任何方法（**包括构造器**）被调用之前得到初始化。例如：

```java
//: OrderOfInitialization.java 
// Demonstrates initialization order. 
// When the constructor is called, to create a 
// Tag object, you'll see a message: 
class Tag { 
    Tag(int marker) { 
        System.out.println("Tag(" + marker + ")"); 
    } 
}

class Card { 
    Tag t1 = new Tag(1); 
    // Before constructor 
    Card() { 
        // Indicate we're in the constructor: 
        System.out.println("Card()"); 
        t3 = new Tag(33); 
        // Re-initialize t3 
    }
    Tag t2 = new Tag(2); 
    // After constructor 
    void f() { 
        System.out.println("f()"); 
    }
    Tag t3 = new Tag(3); 
    // At end 
}

public class OrderOfInitialization { 
    public static void main(String[] args) { 
        Card t = new Card(); 
        t.f(); 
        // Shows that construction is done 
    } 
} ///:
```

在Card中，Tag对象的定义故意到处散布，以证明它们全都会在构建器进入或者发生其他任何 事情之前得到初始化。除此之外，t3在构建器内部得到了重新初始化。它的输出结果如下：

```java
Tag(1) 
Tag(2) 
Tag(3) 
Card() 
Tag(33) 
f()
```

因此，t3句柄会被初始化两次，一次在构建器调用前，一次在调用期间（第一此引用对象会被丢弃，并作为垃圾回收）。若定义了一个重载的构建器，它没有初始化t3；同时在t3的定义里也没有默认值，那么会产生什么后果呢？ 所以从表面看，尽管这样做似乎效率低下，但它能保证正确的初始化。

### 静态数据的初始化

若数据是静态的（static），那么同样的事情就会发生；如果它属于一个基本类型，而且未对其初始化，就会自动获得自己的标准基本类型初始值；如果它是一个对象引用，那么它的默认初始化值就是null。 

如果想在定义的同时进行初始化，采取的方法与非静态数据没什么不同。

但由于static 值只有一个存储区域，所以无论创建多少个对象，都必然会遇到何时对那个存储区域进行初始化的问题。下面这个例子可将这个问题说更清楚一些： 

```java
//: StaticInitialization.java 
// Specifying initial values in a 
// class definition. 
class Bowl { 
    Bowl(int marker) { 
        System.out.println("Bowl(" + marker + ")"); }
    void f(int marker) { 
        System.out.println("f(" + marker + ")"); 
    } 
}

class Table { 
    static Bowl b1 = new Bowl(1); 
    Table() { 
        System.out.println("Table()"); 
        b2.f(1); 
    }
    void f2(int marker) { 
        System.out.println("f2(" + marker + ")"); 
    }
    static Bowl b2 = new Bowl(2); 
}

class Cupboard { 
    Bowl b3 = new Bowl(3);
    static Bowl b4 = new Bowl(4);
    Cupboard() { 
        System.out.println("Cupboard()"); 
        b4.f(2); 
    }
    void f3(int marker) { 
        System.out.println("f3(" + marker + ")"); 
    }
    static Bowl b5 = new Bowl(5); 
}

public class StaticInitialization { 
    public static void main(String[] args) { 						System.out.println( "Creating new Cupboard() in main");
        new Cupboard(); 
        System.out.println( "Creating new Cupboard() in main"); 
        new Cupboard(); 
        t2.f2(1); 
        t3.f3(1); 
    }
    static Table t2 = new Table(); 
    static Cupboard t3 = new Cupboard(); 
} ///:~
```

Bowl允许我们检查一个类的创建过程，而Table和Cupboard能创建散布于类定义中的Bowl的 static成员。注意在static定义之前，Cupboard先创建了一个非static的Bowl b3。它的输出结果如下： 

```
Bowl(1)
Bowl(2)
Table()
f(1) 
Bowl(4) 
Bowl(5) 
Bowl(3) 
Cupboard() 
f(2) 
Creating new Cupboard() in main 
Bowl(3) 
Cupboard() 
f(2) 
Creating new Cupboard() in main 
Bowl(3) 
Cupboard() 
f(2) 
f2(1) 
f3(1)
```

static初始化只有在必要的时候才会进行。如果不创建一个Table对象，而且永远都不引用 Table.b1或Table.b2，那么static Bowl b1和b2永远都不会创建。

然而，只有在创建了第一个 Table对象之后（或者发生了第一次static访问），它们才会创建。在那以后，static对象不会 重新初始化。

初始化的顺序是首先static（如果它们尚未由前一次对象创建过程初始化），接 着是非static对象。大家可从输出结果中找到相应的证据。 

总结一下对象的创建过程。请考虑一个名为Dog的类： 

1. 类型为Dog的一个对象首次创建时，或者Dog类的static方法／static字段首次访问时， Java解释器必须找到Dog.class（在事先设好的类路径里搜索）。 

2. 找到Dog.class后（它会创建一个Class对象，这将在后面学到），它的所有static初始化模 块都会运行。因此，static初始化仅发生一次——在Class对象首次载入的时候。 

3. 创建一个new Dog()时，Dog对象的构建进程首先会在内存堆（Heap）里为一个Dog对象 分配足够多的存储空间。 

4. 这种存储空间会清为零，将Dog中的所有基本类型设为它们的默认值（对数字来说是0，对 布尔型和字符型也相同）。 

5. 执行所有出现于字段定义处的初始化动作。 

6. 执行构造器。会牵涉很多继承的东西。 