# 枚举类型

```java
public class test {
  public static void main(String args[]) {
    for(Spiciness s: Spiciness.values()){
      System.out.println(s+", ordinal"+s.ordinal());
    }
  }
  public enum Spiciness{
    NOT,MILD,MEDIUM,HOT,FLAMING
  }
}
```

输出结果：

```
NOT, ordinal 0
MILD, ordinal 1
MEDIUM, ordinal 2
HOT, ordinal 3
FLAMING, ordinal 4
```

