## 1、什么是异常

- 异常指程序中出现的不期而至的各种状况，如：文件找不到、网络链接失败、非法传参等
- 异常发生在程序运行中，它影响了正常的程序执行流程



## 2、异常体系结构

- Java 把异常当作对象来处理，并定义一个基类 java.lang.Throwable 作为所有异常的超类
- 在 Java API 中已经定义来许多异常类，这些异常类分为两大类，错误 ``Error`` 和异常 ``Exception``

![image-20210329193741700](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210329193741700.png)

### Error

- Error 类对象由 Java 虚拟机生成并抛出，大多数错误与代码编写者所执行的操作无关
- Java 虚拟机运行错误（Virtual Machine Error），当 JVM 不再有继续执行操作所需要的内存资源时，将出现 OutOfMemoryError。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止
- 还有发生在虚拟机试图执行应用时，如类定义错误（NoClassDefFoundError）、链接错误（LinkageError）。这些错误是不可查的，因为它们在应用程序的控制和处理能力之外，而且绝大多数是程序运行时不允许出现的状况



### Exception

- 在 Exception 分支中有一个重要的子类 RuntimeException（运行时异常）
	- ArrayIndexOutOfBoundsException（数组下标越界）
	- NullPointerException（空指针异常）
	- ArithmeticException（算术异常）
	- MissingResourceException（丢失资源）
	- ClassNotFoundException（找不到类）等异常，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理
- 这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生



### Error 和 Exception 的区别：

Error 通常是灾难性的致命的错误，是程序无法控制和处理的，当出现这种异常，Java虚拟机（JVM）一般会选择终止线程；Exception 通常情况下是可以被程序处理的，并且在程序中应该尽可能的去处理这些异常



## 3、Java 异常处理机制

- 抛出异常
- 捕获异常



- 异常处理五个关键字
	- try、catch、finally、throw、throws

![image-20210329200320165](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210329200320165.png)

快捷键：Ctrl + Alt + T

![image-20210329200912093](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210329200912093.png)



## 4、自定义异常

- 使用 Java 内置的异常类可以描述在编程时出现的大部分异常情况，除此之外，用户还可以自定义异常，用户自定义异常类，只需要继承 Exception 类即可



- 在程序中使用自定义异常类，大体可分为以下几个步骤：
	1. 创建自定义异常类
	2. 在方法中通过 throw 关键字抛出异常对象
	3. 如果在当前抛出异常的方法中处理异常，可以使用 try - catch 语句捕获并处理；否则在方法的声明处通过 throws 关键字指明要抛出给方法调用者的异常，继续进行下一步操作
	4. 在出现异常方法的调用者中捕获并处理异常

```java
public class MyException extends Exception {
  // 传递数字 > 10
  private int detail;
  
  public MyException(int a) {
    this.detail = a;
  }
  
  public String toString() {
    return "MyException{" +
      			"detail=" + detail + 
      			"}";
  }
}
```



### 经验总结：

- 处理运行异常时，采用逻辑去合理规避同时辅助 try - catch 处理
- 在多重 catch 块后面，可以加一个 catch （Exception）来处理可能会被遗漏的异常
- 对于不确定的代码，也可以加上 try - catch，处理潜在的异常
- 尽量去处理异常，切忌只是简单地调用 printStackTrace() 去打印输出
- 具体如何处理异常，要根据不同的业务需求和异常类型去决定
- 尽量添加 finally 语句块去释放占用的资源