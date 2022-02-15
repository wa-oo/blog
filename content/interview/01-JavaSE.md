# Java基础

## JDK 和 JRE 有什么区别？
- JDK：Java Development Kit 的简称，Java开发工具包，提供了Java的开发环境和运行环境。
- JRE：Java Runtime Environment 的简称，Java运行环境，为Java的运行提供了所需要环境。

具体来说 JDK 其实包含了 JRE，同时还包含了编译 Java 源码的编译器 Javac，还包含了很多 Java 程序调试和分析的工具。简单来说：如果你需要运行 Java 程序，只需要安装 JRE 就可以了，如果你需要编写 Java 程序，需要安装 JDK。

## == 和 equals 的区别是什么？
**== 解读：**对于基本类型和引用类型 == 的作用效果是不同的。
- 基本类型：比较的是值是否相同；
- 引用类型：比较的是引用是否相同；
```java
String x = "string";
String y = "string";
String z = new String("string");
System.out.println(x == y); // true
System.out.println(x == z); // false
System.out.println(x.equals(y)); // true
System.out.println(x.equals(z)); // true
```
**代码解读：**因为 x 和 y 指向的是同一个引用，所以 == 也是 true，而 new String() 方法则重写开辟了内存空间，所以 == 结果为 false，而 equals 比较的一直是值，所以结果都为 true。

**equals 解读：**equals 本质上就是 ==，只不过 String 和 Integer 等重写了 equals 方法，把它变成了值比较。

首先看默认情况下 equals 比较一个相同值的对象：
```java
class Cat {
  public Cat(String name) {
    this.name = name;
  }
  
  private String name;
  
  public String getName() {
    return name;
  }
  
  public void setName(String name) {
    this.name = name;
  }
}

Cat c1 = new Cat("王老大");
Cat c2 = new Cat("王老大");

System.out.println(c1.equals(c2));	// false
```

输出结果为什么是false，看下 equals 源码就知道了：

```java
public boolean equals(Object obj) {
  return (this == obj);
}
```
原来 equals 本质上就是 ==。

那问题来了，两个相同值的 String 对象，为什么返回的是 true？
```java
String s1 = new String("王老大");
String s2 = new String("王老大");
System.out.println(s1.equals(s2));	// true
```