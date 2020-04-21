# 可变参数列表

```java
public class test {
  public static void main(String args[]) {
    f(new Integer(47),new Float(3.14), new Double(11.11));
    f(47, 3.14F, 11,11);
    f("one", "two", "three");
    f(new A(), new A(), new A());
    f((Object[])new Integer[]{1,2,3,4});
    f();
  }
  static public void f(Object...args){
    for(Object obj:args){
      System.out.print(obj+" ");
    }
    System.out.println();
  }
}

class A{}
```

输出结果：

```
47 3.14 11.11
47 3.14 11 11
one two three
A@15db9742 A@6d06d69c A@7852e922
1 2 3 4
```



- 有了可变参数，就再也不用显式地编写数组语法了，当你指定参数时，编译器实际上会为你去填充数组。
- 这不仅仅只是从元素列表到数组地自动转换，main函数的倒数第二行，一个Integer数组（通过使用自动包装而创建的）被转型为一个Object数组，并且传递给了f()。很明显，编译器会发现它已经是一个数组了，所以不会在其上执行任何转换。