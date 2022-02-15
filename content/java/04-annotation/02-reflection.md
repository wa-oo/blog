---
title: 反射机制
date: 2021-04-24
categories: [Reflection]
toc: true
---

## 1、Java 反射机制概述

### 静态 VS 动态语言

![image-20210425192840004](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210425192840004.png)

### Java Reflection

![image-20210424103019495](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210424103019495.png)

### 反射的优点和缺点

#### 优点：

- 可以实现动态创建对象和编译，体现出很大的灵活性

#### 缺点：

- 对性能有影响

	使用反射基本上是一种解释操作，我们可以告诉 JVM，我们希望做什么并且它满足我们对要求。这类操作总是慢于直接执行相同的操作。

### 反射相关的主要 API

![image-20210424103732114](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210424103732114.png)

```java
// 什么是反射
public class Demo01 {
    public static void main(String[] args) throws ClassNotFoundException {
        // 通过反射获取类的class对象
        Class<?> c1 = Class.forName("com.allen.reflection.User");
        System.out.println(c1); // class com.allen.reflection.User

        // 一个类在内存中只有一个class对象
        // 一个类被加载后，类的整个结构都会被封装在Class对象中
        // c2,c3,c4 hassCode值都相同
        Class<?> c2 = Class.forName("com.allen.reflection.User");
        Class<?> c3 = Class.forName("com.allen.reflection.User");
        Class<?> c4 = Class.forName("com.allen.reflection.User");
    }
}

// 实体类：pojo，entity
class User {
    private String name;
    private int id;
    private int age;

    public User() {
    }

    public User(String name, int id, int age) {
        this.name = name;
        this.id = id;
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", id=" + id +
                ", age=" + age +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```



## 2、理解 Class 类并获取 Class 实例

### Class 类

![image-20210424104345664](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210424104345664.png)

------

![image-20210424104525118](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210424104525118.png)

### Class 类的常用方法

![image-20210424104809479](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210424104809479.png)

### 获取 Class 类的实例

![image-20210424105002989](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210424105002989.png)

```java
// 测试 Class 类的创建方式有哪些
public class Demo02 {
    public static void main(String[] args) throws ClassNotFoundException {
        Person person = new Person();
        System.out.println("这个人是："+person.name);
        // 方式一：通过对象获得
        Class c1 = person.getClass();

        // 方式二：forName获得
        Class<?> c2 = Class.forName("com.allen.reflection.Person");

        // 方式三：通过类名.class获得
        Class c3 = Person.class;

        // 方式四：基本内置类型的包装类都有一个type属性
        Class c4 = Integer.TYPE;
        
        // 获得父类类型
        Class c5 = c1.getSuperclass();
    }
}

class Person {
    String name;

    public Person() {
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                '}';
    }

    public Person(String name) {
        this.name = name;
    }
}

class Student extends Person {
    public Student() {
        this.name = "学生";
    }
}

class Teacher extends Person {
    public Teacher() {
        this.name = "老师";
    }
}
```

### 哪些类型可以有 Class 对象？

- class：外部类，成员（成员内部类，静态内部类），局部内部类，匿名内部类
- interface：接口
- []：数组
- enum：枚举
- annotation：注解@interface
- primitive type：基本数据类型
- void

```java
// 所有类的Class
public class Demo03 {
    public static void main(String[] args) {
        Class c1 = Object.class; // 类
        Class c2 = Comparable.class; // 接口
        Class c3 = String[].class; // 一维数组
        Class c4 = int[][].class; // 二维数组
        Class c5 = Override.class; // 注解
        Class c6 = ElementType.class; // 枚举
        Class c7 = Integer.class; // 基本数据类型
        Class c8 = void.class; // void
        Class c9 = Class.class; // Class

        // 只要元素类型与纬度一样，就是同一个class
        int[] a = new int[10];
        int[] b = new int[100];
        System.out.println(a.getClass().hashCode() == a.getClass().hashCode()); // true
    }
}
```



## 3、类的加载与 ClassLoader

### Java 内存分析

![image-20210424151805952](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210424151805952.png)

### 了解：类的加载过程

![image-20210424151942542](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210424151942542.png)

### 类的加载与 ClassLoader 的理解

![image-20210424152143664](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210424152143664.png)

```java
public class Demo04 {
    public static void main(String[] args) {
        A a = new A();
        System.out.println(A.m);
        /**
         * 1.加载内存后，会产生一个类对应Class对象
         * 2.链接，链接结束后 m = 0
         * 3.初始化
         *      <clinit>() {
         *          System.out.println("A类静态代码块初始化");
         *          m = 300;
         *          m = 100;
         *      }
         *      m = 100;
         */

    }
}

class A {
    static {
        System.out.println("A类静态代码块初始化"); // 第一个输出
        m = 300;
    }

    static int m = 100; // 第三个输出

    public A() {
        System.out.println("A类的无参构造初始化"); // 第二个输出
    }
}
```



![image-20210424163345983](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210424163345983.png)





## 4、创建运行时类的对象

### 什么时候回发生类初始化？

![image-20210424163911543](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210424163911543.png)

```java
// 测试类什么时候会初始化
public class Demo05 {
    static {
        System.out.println("Main被加载");
    }

    public static void main(String[] args) throws ClassNotFoundException {
        // 1.主动引用
        Son son = new Son();

        // 反射也会产生主动引用
        Class.forName("com.allen.reflection.Son");

        // 不会产生类的引用的方法
        System.out.println(Son.b);
        Son[] sons = new Son[5];
        System.out.println(Son.M);

    }
}

class Father {
    static int b = 3;
    static {
        System.out.println("父类被加载");
    }
}

class Son extends Father {
    static {
        System.out.println("子类被加载");
        m = 300;
    }

    static int m = 100;
    static final int M = 1;
}
```

### 类加载器的作用

![image-20210425193038908](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210425193038908.png)

![image-20210425193252804](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210425193252804.png)

![image-20210513212338123](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513212338123.png)



## 5、获取运行时类的完整结构

![image-20210513214145634](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513214145634.png)

![image-20210513214347376](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513214347376.png)

![image-20210513214711546](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513214711546.png)

![image-20210513214946562](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513214946562.png)



## 6、调用运行时类的指定结构

![image-20210513220338661](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513220338661.png)

![image-20210513220452532](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513220452532.png)

![image-20210513220523870](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513220523870.png)

![image-20210513220551267](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513220551267.png)

![image-20210513215631499](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513215631499.png)

![image-20210513215854348](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513215854348.png)

![image-20210513220220299](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513220220299.png)

### 性能问题分析

```java
// 普通方式调用
public static void test01() {
  User user = new User();
  long startTime = System.currentTimeMillis();
  for (int i = 0; i < 1000000000; i++) {
    user.getName();
  }
  long endTime = System.currentTimeMillis();
  System.out.println("普通方式执行10亿次：" + (endTime - startTime) + "ms");
}

// 反射调用方式
public static void test02() {
  User user = new User();
  Class c1 = user.getClass();
  Method getName = c1.getDeclaredMethod("getName", null);
  long startTime = System.currentTimeMillis();
  for (int i = 0; i < 1000000000; i++) {
    getName.invoke(user, null);
  }
  long endTime = System.currentTimeMillis();
  System.out.println("反射调用方式执行10亿次：" + (endTime - startTime) + "ms");
}

// 反射调用方式--关闭检测
public static void test02() {
  User user = new User();
  Class c1 = user.getClass();
  Method getName = c1.getDeclaredMethod("getName", null);
  getName.setAccessible(true);
  long startTime = System.currentTimeMillis();
  for (int i = 0; i < 1000000000; i++) {
    getName.invoke(user, null);
  }
  long endTime = System.currentTimeMillis();
  System.out.println("反射调用方式--关闭检测执行10亿次：" + (endTime - startTime) + "ms");
}

// 普通方式执行10亿次：9ms
// 反射调用方式执行10亿次：5699ms
// 反射调用方式--关闭检测执行10亿次：1959ms
```



### 获取泛型信息

![image-20210513221845469](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513221845469.png)

![image-20210513222628751](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513222628751.png)

![image-20210513222447293](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513222447293.png)

![image-20210513222604707](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513222604707.png)

### 反射获取注解信息

![image-20210513222748225](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210513222748225.png)

```java
public class Demo06 {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException {
        Class c1 = Class.forName("com.allen.reflection.Student2");

        // 通过反射获取注解
        Annotation[] annotations = c1.getAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println(annotation);
        }

        // 获取注解的value值
        TableAllen tableAllen = (TableAllen)c1.getAnnotation(TableAllen.class);
        String value = tableAllen.value();
        System.out.println(value);
        
        // 获得类指定的注解
        Field name = c1.getDeclaredField("name");
        fieldAllen annotation = name.getAnnotation(fieldAllen.class);
        System.out.println(annotation.columnName());
        System.out.println(annotation.type());
        System.out.println(annotation.length());

    }
}

@TableAllen("db_student")
class Student2 {

    @fieldAllen(columnName = "db_id", type = "int", length = 10)
    private int id;

    @fieldAllen(columnName = "db_age", type = "int", length = 10)
    private int age;

    @fieldAllen(columnName = "db_name", type = "varchar", length = 10)
    private String name;

    public Student2() {
    }

    public Student2(int id, int age, String name) {
        this.id = id;
        this.age = age;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

// 类名的注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface TableAllen{
    String value();
}

// 属性的注解
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface fieldAllen{
    String columnName();
    String type();
    int length();
}
```



















