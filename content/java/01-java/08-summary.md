## 基础语法

### 注释

- 行内注释 //
- 多行注释 /\*\*/
- 文档注释 /\*\* \*/ javac 生成帮助文档

### 标识符

- 关键字

### 数据类型

- 基本数据类型
  - 整数
    - byte 1 字节
    - short 2 字节
    - int（默认） 4 字节
    - long 8 字节
    - 0b 二进制
    - 0x 十六进制
  - 浮点数
    - float 4 字节
    - double（默认） 8 字节
    - BigDecimal（金融）
  - 字符
    - char 2 字节
    - ascii
    - utf-8
    - Unicode
  - 布尔值
    - boolean 1 字节
- 引用数据类型
  - 类
  - 接口
  - 数组

### 类型转换

- 自动类型转换
  - 低转高
- 强制类型转换
  - 高转低 -> (低)高

### 变量和常量

- `type varName [=value];`
- 作用域

  - 类变量
  - 实例变量
  - 局部变量

- 常量

  - `final MAX_A = 10;`

- 命名规范
  - 见名知意
  - 驼峰命名（变量、方法）
  - 类（首字母大写，驼峰命名）
  - 常量（大写 + 下划线）
  - 不要使用拼音命名

### 运算符

- 算术运算符：`+ - * / ++ --`
- 赋值运算符：`=`
- 关系运算符：`> < >= <= != instanceof`
- 逻辑运算符：`&& || !`
- 位运算符：`& | ^ ~ >> << >>>`
- 条件运算符：`?:`
- 扩展运算符：`+= -= *= /=`

###

### 包机制

- 域名倒写
- 防止命名冲突
- `package`
- `import`

### JavaDoc

- JDK 帮助文档
- javadoc
  - @author（作者）
  - @Version（版本）
  - @Since（多少版本的 JDK 支持）
  - @param（参数）
  - @return（返回）
  - @throws（异常）

## 流程控制

### Scanner

- 用户交互（System.in）

### 顺序结构（程序默认结构，自上而下的执行）

### 选择结构

- if 单选择结构
- if - else 双选择结构
- if - else - if - else 多选择结构
- switch
  - JDK 支持了 String 类型
  - case 穿透现象
  - break
  - default

### 循环结构

- while（尽量避免死循环）
- do...while
- for：`for(int 1 = 0; i < 100; i++){}`
- 增强 for 循环

### break & continue

- break：跳出循环
- continue：终止当次循环
- return：结束方法的运行

## 方法

### 什么是方法：语句块的集合

### 方法的定义：`修饰符 返回值 方法名（参数名）{return 返回值;}`

### 方法调用

- 类名.方法
- 对象.方法

### 方法重载：名字相同，参数列表不同

### 命令行传参：给 main 方法传参

### 可变长参数：...

- 必须放在最后一个参数

### 递归：

- 自己调自己，给自己一个出口

## 数组

### 数组的定义

- `new int[5]`
- `{1, 2, 3}`
- 必须是同一个类型

### 数组的使用

- 通过下标拿到
- `ArrayIndexOutOfBounds`
- 增加 for 循环遍历

### 二维数组：`int[][]`

### Arrays 工具类

### 排序算法

- 冒泡排序
- 选择排序
- 插入排序
- 希尔排序
- 快速排序
- 归并排序
- 基数排序
- 堆排序

## 面向对象

### 什么是面向对象：用类组织代码，用对象组织数据

### 类与对象

- 类是对象的抽象：模版 Class
- 对象是类的抽象

### 构造方法：构造的重载

- 默认的无参构造
- 如果手动定义了有参构造就必须要手动再加一个无参构造
- 单例模式，需要构造器私有

### new 对象

- 栈存放引用
- 堆存放具体的对象

### 封装：属性私有，get、set

### 继承：

- extends
- Object
- 子类拥有父类的全部特性
- 方法重写
- this
- super
- Java 是单继承，只能继承一个父类

### 多态：父类的引用指向子类的对象

- `Person person = new Student();`

- `instanceof` 关键字，如果匹配，可以进行类型之间的转换

### 修饰符

- public
- protected
- private
- static
- final
- abstract

### 接口

- interface
- 约束，只能定义方法名
- 子类实现接口，必须重写其中的方法
- 只有一个方法的接口叫做函数式接口，可以使用 lambda 表达式简化
- 抽象类：接口比抽象类更抽象
- 一个类可以实现多个接口

### 内部类

- 成员内部类
- 局部内部类
- 静态内部类
- 匿名内部类（重点）

## 异常：Throwable

### Exception

- 运行时异常
  - 1/0
  - ClassNotFound
  - NullPoint
  - 下标越界异常

### Error

- AWT 错误
- JVM 错误
  - StackOverFlow（栈溢出）
  - OutOfMemory（内存溢出）

### 五个关键字

- try
- catch：先小后大
- finally
- throw：手动抛出异常
- throws：方法抛出异常

### 自定义异常：继承 Exception 类即可

### 常用类：

- Object 类
  - hashcode()
  - toString()
  - clone()
  - getClass()
  - notify()
  - wait()
  - equals()
- Math 类：常见的数学运算
- Random 类：生成随机数、UUID
- File 类
  - 创建文件
  - 查看文件
  - 修改文件
  - 删除文件
- 包装类：自动装箱和拆箱
- Date 类
  - Date
  - SimpleDateFormat
    - yyyy-MM-dd HH:mm:ss
  - Calendar（建议使用）
- String 类：不可变性 （final）数据量较少
- StringBuffer 类：可变长（`append()`）多线程数据量较大，效率低，安全
- StringBuilder 类：可变长 单线程数据量大，效率高，不安全

## 集合框架

### Collection

- List（有序可重复）
  - ArrayList
    - add
    - remove
    - contains
    - size
  - LinkedList
    - getFirst
    - getLast
    - removeFirst
    - ......
  - Vector
  - Stack
- Set（无序，不可重复）
  - HashSet（常用）
  - TreeSet

### Map

- HashMap（常用、重点）
  - JDK1.7 数组 + 链表
  - JDK1.8 数组 + 链表 + 红黑树
- TreeMap

### Collections 工具类

### 泛型 <> 约束，避免类型转换之间的问题

## IO 流
