# 访问权限

Java一共有四种访问权限，按照权限由大到小分别为public、protected、default和private，如果省略了访问修饰符，那访问权限就是defualt。四种访问权限的含义和权限控制见下面表格：

| 访问权限  | 含义         | 本类 | 本包的类 | 非本包子类 | 非本包非子类 |
| --------- | ------------ | ---- | -------- | ---------- | ------------ |
| public    | 公共的       | 是   | 是       | 是         | 是           |
| protected | 保护访问权限 | 是   | 是       | 是         | 否           |
| default   | 包访问权限   | 是   | 是       | 否         | 否           |
| private   | 私有的       | 是   | 否       | 否         | 否           |

`需要注意的是，所谓的访问可以分为两种不同的方式：1.通过对象实例访问；2.直接访问`

上面的表格实际上指的是通过对象实例访问，或者是通过类名访问静态方法

直接访问只有在继承了父类的子类中体现。

## 举一些栗子

例如有以下几个类，ClassA与ClassB在同一个包下，ClassC与ClassD在同一个包下，C继承了A。

ClassA：

![](https://github.com/shenjiahuihm/note/blob/master/imgs/Java/ClassA.png)

ClassA中从上到下分别是public、protected、default、private方法。

ClassB：

![](https://github.com/shenjiahuihm/note/blob/master/imgs/Java/ClassB.png)

ClassB因为与ClassA是同一个包，所以可以通过A的实例访问除了private以外的方法或变量。

ClassC：

![](https://github.com/shenjiahuihm/note/blob/master/imgs/Java/ClassC.png)

ClassC继承了ClassA，尽管它们在不同的包，但ClassC依然能直接访问protected方法，但是不能通过ClassA的实例访问。

可以这样理解，实际上C能访问到是它从A继承而来的protected成员（C自己的成员），而A的protected成员访问不到，因为它们不是同一个包。

如果有个类E和A是同一个包，并且E继承了A，那么E不仅能访问自己继承而来的protected成员（E自己的成员），也能访问它的父类A的protected成员（通过A的实例访问）。

另外总结一下，在同一个包中，子类是能继承父类的除了private成员的其他所有成员（即便是包访问权限default也是如此，因为子类也算在这个包里，当然能访问同一包里的default成员）；而在不同的包中，子类只能继承父类的public与protected成员，只能访问父类的pubilc成员。

ClassD：

![](https://github.com/shenjiahuihm/note/blob/master/imgs/Java/ClassD.png)

ClassD也很好理解了，不同包下只有public是可见的。