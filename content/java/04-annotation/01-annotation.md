---
title: 注解
date: 2021-04-24
categories: [Annotation]
toc: true
---

## 1、注解入门

- Annotation 是从 JDK5.0 开始引入的新技术
- Annotation 的作用：
	- 不是程序本身，可以对程序作出解释（这一点和注释（comment）没有什么区别
	- 可以被其他程序（比如：编译器）读取
- Annotation 的格式：
	- 注解是以“@注解名”在代码中存在的，还可以添加一些参数值，例如：@SuppressWarnings(value="unchecked")
- Annotation 在哪里使用？
	- 可以附加在 package，class，method，field 等上面，相当于给他们添加了额外的辅助信息，我们可以通过反射机制编程实现对这些元数据的访问



## 2、内置注解

![image-20210424095304838](../../../../../../../../Library/Application Support/typora-user-images/image-20210424095304838.png)



## 3、自定义注解，元注解

### 元注解

![image-20210424100010974](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210424100010974.png)

```java
// 测试元注解
public class Demo01 {

    @MyAnnotation
    public void test() {

    }
}

// 定义一个注解
// Target：表示我们的注解可以用在哪些地方
@Target(value = {ElementType.METHOD, ElementType.TYPE})
// Retention：表示我们的注解在什么地方有效
// runtime > class > source
@Retention(RetentionPolicy.RUNTIME)
// Documented：表示是否将我们的注解生成在 JavaDoc 中
@Documented
// Inherited：子类可以继承父类的注解
@Inherited
@interface MyAnnotation {}

```

### 自定义注解

![image-20210424101103179](../../../../../../../../Library/Application Support/typora-user-images/image-20210424101103179.png)

```java
// 自定义注解
public class Demo02 {
    // 注解可以显示赋值，如果没有默认值，就必须给注解赋值
    @MyAnnotation(name = "Allen")
    public void test() {}
    
    @MyAnnotation2("Allen")
    public void test2(){}
}

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation {
    // 注解的参数：参数类型 + 参数名（）；
    String name() default "";
    int age() default 0;
    // 如果默认值为 -1，代表不存在
    int id() default -1;
    String[] schools() default {"北京大学", "清华大学"};
}

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation2 {
    String value();
}
```

