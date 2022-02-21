## 1、用户交互Scanner

> java.util.Scanner是Java5的新特性，可以通过Scanner类来获取用户的输入

### Scanner对象

- 基础语法

```java
Scanner s = new Scanner(System.in);
```

- 通过Scanner类的next()与nextLine()方法获取输入的字符串，在读取前我们一般需要使用hasNext()于hasNextLine()判断是否含有输入的数据

```java
// 创建一个扫描器对象，用于接收键盘数据
Scanner scanner = new Scanner(System.in);
System.out.println("使用next方式接收：");
// 判断用户有没有输入字符串
if (scanner.hasNext()){
	// 使用next方式接收
  String str = scanner.next();
  System.out.println("输出内容为：" + str);
}   
// 凡是使用IO流的类如果不关闭会一直占用资源，要养成良好的习惯用完就关闭
scanner.close();
```

- next()：
	1. 一定要读取到有效字符后才可以结束输入。
	2. 对输入有效字符之前遇到的空白，next()方法会自动将其去掉。
	3. 只有输入有效字符后才将其后面输入的空白作为分隔符或者结束符。
	4. next()不能得到带空格的字符串
- nextLine()：
	1. 以Enter为结束符，也就是说nextLine()方法返回的是输入回车之前的所有字符。
	2. 可以获得空白。



## 2、顺序结构

- Java的基础结构就是顺序结构，除非特别指明，否则就按照顺序一句一句执行。

- 顺序结构是最简单的算法结构

	<img src="https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210303211625688.png" alt="image-20210303211625688" style="zoom:50%;" />

- 语句与语句之间，框与框之间是按从上到下的顺序进行的，它是由若干个一次执行的处理步骤组成的，**它是任何一个算法都离不开的一种基本算法结构**。



## 3、选择结构

- if单选择结构

  我们很多时候需要判断一个东西是否可行，然后我们采取执行，这样一个过程在程序中用if语句来表示

  语法：

```java
if(布尔表达式){
  // 如果布尔表达式为true将执行的语句
}
```

<img src="https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210303212722923.png" alt="image-20210303212722923" style="zoom:50%;" />

- if双选择结构

  需要两个判断，就需要一个双选择结构，所以就用if - else结构

  语法：

```java
if (布尔表达式){
  // 如果布尔表达式的值为true
} else {
  // 如果布尔表达式的值为false
}
```

<img src="https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210303213718386.png" alt="image-20210303213718386" style="zoom:50%;" />

- if多选择结构

	语法：

```java
if (布尔表达式 1){
  // 如果布尔表达式 1的值为true执行代码
} else if (布尔表达式 2) {
  // 如果布尔表达式 2的值为true执行代码
} else if (布尔表达式 3) {
  // 如果布尔表达式 3的值为true执行代码
} else {
  // 如果以上布尔表达式都不为true执行代码
}
```

- 嵌套的if结构

	语法：

```java
if (布尔表达式 1) {
  // 如果布尔表达式 1的值为true执行代码
  if (布尔表达式 2) {
    // 如果布尔表达式 2的值为true执行代码
  }
}
```

- switch多选择结构

```java
switch (expression) {
  case value :
    // 语句
    break; // 可选
  case value :
    // 语句
    break; // 可选
  // 可以有任意数量的case语句
  default : // 可选
    // 语句
}
```

​		判断一个变量与一系列值中某个值是否相等，每个值称为一个分支。

​		语法：

switch语句中的变量类型可以是：

- byte、short、int或者char。
- 从Java7开始，switch支持字符串String类型了
- case标签必须为字符串常量或字面量。



## 4、循环结构

#### while循环

```java
while (布尔表达式) {
  // 循环内容
}
```

- 只有布尔表达式为true，循环就会一直执行下去。
- **我们大多数情况是会让循环停止下来的，我们需要一个让表达式失效的方式来结束循环**。
- 少部分情况需要循环一直执行，比如服务器的请求响应监听等。
- 循环条件一直为true就会造成无限循环【死循环】，我们正常的业务编程中应该尽量避免死循环，会影响程序性能或者造成程序卡死崩溃！

#### do...while循环

```java
do {
  // 代码语句
} while (布尔表达式);
```

- 对于while语句来说，如果不满足条件，则不能进入循环，但有时候我们需要即使不满足条件，也至少执行一次。
- do...while循环和while循环相似，不同的是，do...while循环至少会执行一次。

> while和do...while的区别：
>
> 1. while先判断后执行，do...while是先执行后判断。
> 2. do...while总是保证循环体会被至少执行一次！

#### for循环

```java
for(初始化; 布尔表达式; 更新) {
  // 代码语句
}
```

- 是支持迭代的一种通用结构，**是最灵活，最有效的循环结构**。
- 循环执行的次数实在执行前就确定的。

- 在Java5中引入了一种主要用于数组的增强型for循环 

	```java
	for(声明语句 : 表达式) {
	  // 代码语句
	}
	```

	- 主要用于数组或集合的增强型for循环。
	- 声明语句：声明新的局部变量，该变量的类型必须和数组元素的类型匹配。其作用域限定在循环语句块，其值与此时数组元素的值相等。
	- 表达式：表达式是要访问的数组名，或者是返回值为数组的方法。



## 5、break & continue

- break在任何循环语句的主体部分，均可用break控制循环流程。break**用于强行退出循环**，不执行循环中剩余的语句。（break语句也在switch语句中使用）
- continue语句用在循环语句体中，**用于终止某次循环过程**，即跳过循环体尚未执行的语句，接着进行下一次是否执行循环的判定。



- **关于goto关键字**
	- goto关键字很早就在程序设计语言中出现，尽管goto仍是Java的一个保留字，但并未在语言中得到正式使用；Java没有goto，然而，在break和continue这两个关键字的身上我们仍然能看出一些goto的影子---带标签的break和continue。
	- “标签”是指后面跟一个冒号的标识符，例如：label:
	- 对Java来说唯一用到标签对地方实在循环语句之前，而在循环之前设置标签的唯一理由：我们希望在其中嵌套另一个循环，又要break和continue关键字通常只中断当前循环，但若随同标签使用，它们就会中断到存在标签的地方。

