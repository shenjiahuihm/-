###  Java成员初始化

---

#### 一、方法的局部变量

​	Java尽力保证：所有变量在使用前都能得到恰当的初始化。对于方法的局部变量，Java以编译时错误的形式来贯彻这种保证。所以如果写成：

```java
void f(){
    int i;
    i++; // 错误，i没有初始化
}
```

就会得到一条出错消息，告诉你i可能尚未初始化。这样强制程序员提供一个初始值，往往能够帮助找出程序里的缺陷。(对于特殊的main()方法也是一样的)

#### 二、 类的数据成员

​	要是类的数据成员（即字段）是基本类型，情况就会变得有些不同。

​	类的每个基本类型数据成员保证都会有一个初始值。

​	下面的程序可以验证这类情况，并显示它们的值：

```java
//: InitialValues.java 
// Shows default initial values 
class Measurement { 
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
            "double " + d); } 
}
public class InitialValues { 
    public static void main(String[] args) { 
        Measurement d = new Measurement(); 
        d.print(); 
        /* 你也可以这样写: new Measurement().print(); */ 
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