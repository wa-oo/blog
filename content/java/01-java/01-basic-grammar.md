## 1、注释

- 平时我们编写代码，在代码量比较少的时候，我们还可以看懂自己写的，但是当项目结构一旦复杂起来，我们就需要用到注释啦
- 注释并不会被执行，是写给我们写代码的人看的
- **书写注释是一个非常好的习惯**



#### Java中的注释有三种：

- 单行注释
- 多行注释
- 文档注释



## 2、标识符

### 关键字

|    abstract    |      assert      |    boolean    |     break      |    byte    |
| :------------: | :--------------: | :-----------: | :------------: | :--------: |
|    **case**    |    **catch**     |   **char**    |   **class**    | **const**  |
|  **continue**  |   **default**    |    **do**     |   **double**   |  **else**  |
|    **enum**    |   **extends**    |   **final**   |  **finally**   | **float**  |
|    **for**     |     **goto**     |    **if**     | **implements** | **import** |
| **instanceof** |     **int**      | **interface** |    **long**    | **native** |
|    **new**     |   **package**    |  **private**  | **protected**  | **public** |
|   **return**   |   **strictfp**   |   **short**   |   **static**   | **super**  |
|   **switch**   | **synchronized** |   **this**    |   **throw**    | **throws** |
| **transient**  |     **try**      |   **void**    |  **volatile**  | **while**  |

**Java所有的组成部分都需要名字，类名，变量名以及方法名都被称为标志符**

### 标识符注意点

- 所有的标识符都应该以字母（A - Z或者a - z），美元符（$），或者下划线（_）开始
- 首字符之后可以是字母（A - Z或者a - z），美元符（$），或者下划线（_）或者数字的任何字符组合
- **不能使用关键字作为变量名或方法名**
- 标识符是**大小写敏感**的
- 合法标识符举例：age、$salary、_value、1_value
- 非法标识符举例：123abc、-salary、#abc
- 可以使用中文命名，但是一般不建议这样使用，也不建议使用拼音，很LOW



## 3、数据类型

- 强类型语言
	- 要求变量的使用严格符合规定，所有变量都必须先定义后才能使用

- 弱类型语言



### Java的数据类型分为两大类

- 基本类型（primitive type）
	- 数值类型
		- 整数类型
			- byte 占1个字节，范围：-128～127
			- short 占2个字节，范围：-32768～23767
			- int 占4个字节，范围：-2147483648～2147483647
			- long 占8个字节，范围：-9223372036854775808～9223372036854775807
		- 浮点类型
			- float 占4个字节
			- double 占8个字节
		- 字符类型 char 占2个字节
	- boolean 类型：占1位其值只有 true 和 false 两个
- 引用类型（reference type）
	- 类
	- 接口
	- 数组

### 什么是字节

- 位（bit）：是计算机内部数据存储的最小单位，11001100是一个八位二进制数。
- 字节（byte）：是计算机数据处理的基本单位，习惯上用大写B来表示
- 1B（byte，字节）= 8bit（位）
- 字符：是计算机中使用的字母、数字、字和符号



- 1bit 表示1位
- 1Byte 表示1个字节 1B = 8b
- 1024B = 1KB
- 1024KB = 1M
- 1024M = 1G



```java
// 整数拓展
int a = 10;		// 十进制：输出10
int b = 010;	// 八进制：输出8
int c = 0x10;	// 十六进制（0～9 A～F）：输出16

// 浮点数拓展
// 最好完全避免使用浮点数进行比较
// 银行业务怎么表示？BigDecimal数学工具类
float d = 0.1f;
double e = 1.0/10;
System.out.print(d==e);			// false

float f1 = 1256456485f;
float f2 = f1 + 1;	
System.out.print(f1==f2);		// true

// 字符拓展
char c1 = 'a';
char c2 = '中';
(int)c1;										// 强制转换
// 所有的字符本质还是数字
// 编码		Unicode		2字节		0-65536
char c3 = '\u0061';

// 转义字符
// \t		制表符
// \n 	换行
```



### 类型转换

- 由于 java 是强类型语言，所以要进行有些 运算的时候，需要用到类型转换

	```java
	// 低 ------------------> 高
	byte, short, char -> int -> long -> float -> double
	```

- 运算中，不同类型的数据先转化为同一类型，然后进行运算

- 强制类型转换 **(类型)变量名** 

	```java
	int i = 127;
	byte b = (byte)i;
	```

- 自动类型转换(**低类型转高类型**)

	```java
	int i = 127;
	double d = i;
	```

- 注意点：
	- 不能对布尔值进行转换
	- 不能把对象类型转为不相干的类型
	- 在把高容量转换为低容量的时候，强制转换
	- 转换的时候可能存在内存溢出，或者精度问题

- 问题：

	```java
	// 操作比较大的数的时候，注意溢出问题
	int money = 10_0000_0000;			// JDK7新特性，数字之间可以用下划线分割
	int years = 20;
	int total = money * years;		// -1474836480，计算的时候溢出了
	long total2 = money * years;	// -1474836480，默认是int，转换之前已经存在问题了
	long total3 = money * ((long)years);	//先把一个数转换为long
	```

	

## 4、变量

- 变量是什么：就是可以变化的量！

- Java 是一种强类型语言，每个变量都必须声明其类型。

- Java 变量是程序中最基本的存储单元，其要素包括变量名，变量类型和**作用域**。

	```java
	type varName [= value] [{, varName [= value]}] ;
	// 数据类型  变量名  =  值； 可以使用逗号隔开来声明多个同类型变量。
	int a=1,b=2,c=3;	// 不推荐，影响程序可读性
	```

- 注意事项：

	- 每个变量都有类型，类型可以是基本类型，也可以是引用类型。
	- 变量名必须是合法的标识符。
	- 变量声明是一条完整的语句，因此每一个声明都必须以分号结束。

### 变量作用域

- 类变量
- 实例变量：从属于对象，如果不自行初始化，则会用这个类型的默认值
- 局部变量：必须声明和初始化值

```java
public class Variable {
  static int allClicks = 0;			// 类变量
  String str = "hello world";		// 实例变量
  
  public void method() {
    int i = 0;									// 局部变量
    System.out.pritln(i);
    
    // 实例变量输出
    Variable variable = new Variable();
    System.out.pritln(variable.str);
  }
}
```

### 常量

- 常量（Constant）：初始化之后不能再改变值！不会变动的值
- 所谓常量可以理解成一种特殊的变量，它的值被设定后，在程序运行过程中不允许被改变。
- 常量名一般使用大写字符。

```java
final 常量名 = 值;
final double PI = 3.14;
```

### 变量的命名规范

- 所有变量、方法、类名：见名知意
- 类成员变量：首字母小写和驼峰原则：monthSalary
- 局部变量：首字母小写和驼峰原则
- 常量：大写字母和下划线：MAX_VALUE
- 类名：首字母大写和驼峰原则：Man，GoodMan
- 方法名：首字母小写和驼峰原则：run()，runRun()



## 5、运算符（operator）

- 算术运算符：+，-，*，/，%，++，--

	```java
	int a = 3;
	int b = a++;		// 先赋值后运算，b=3
	int c = ++a;		// 先运算后赋值，c=5
	```

- 赋值运算符：=

- 关系运算符：>，<，>=，<=，==，!=，instanceof

- 逻辑运算符：&&，||，!

- 位运算符：&，｜，^，～，>>，<<，>>>(了解！)

- 条件运算符：? :

- 扩展赋值运算符：+=，-=，*=，/=

	```java
	int a = 10;
	int b = 20;
	
	a += b; // a = a + b
	a -= b; // a = a - b
	// 字符串连接符
	"" + a + b;		// 输出1020
	a + b + ""; 	// 输出30
	```

	

## 6、包机制

- 更好的组织类，Java提供了包机制，用于区别类名的命名空间。

- 包语言的语法格式：

	```java
	package pkg1[.pkg2[.pkg3...]];
	```

- **一般利用公司域名倒置作为包名**。

- 为了能够使用某一个包的成员，我们需要在Java程序中明确导入该包，使用“import“语句可完成此功能。

	```java
	import package1[.package2...].(className|*);
	```



## 7、JavaDOC

- javadoc命令是用来生成自己API文档的
- 参数信息
	- @author 作者
	- @version 版本号
	- @since 指明需要最早使用的jdk版本
	- @param 参数名
	- @return 返回值
	- @throws 异常抛出情况

```shell
javadoc -encoding UTF-8 -charset UTF-8 Doc.java
```

