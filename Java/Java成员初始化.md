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

