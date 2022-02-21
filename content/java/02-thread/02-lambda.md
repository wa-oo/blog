---
title: Lambda 表达式
date: 2021-04-07
categories: [Thread]
toc: true
---



![Λ](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/th?id=AMMS_5a547dbc53b14716111b4a28a93eb0c4&w=110&h=110&c=7&rs=1&qlt=80&pcl=f9f9f9&cdv=1&pid=16.1)

> 避免匿名内部类定义过多
>
> 其实质属于函数式编程的概念

`(params) -> expression [表达式]`

`(params) -> statement [语句]`

`(params) -> {statements}`

```java
a -> System.out.println("i like lambda -->" + a);

new Thread(() -> System.out.println("多线程...")).start();
```

### 为什么要使用 lambda 表达式

- 避免匿名内部类定义过多
- 可以让你的代码看起来很简洁
- 去掉了一堆没有意义的代码，只留下核心的逻辑

>  理解 Functional Interface（函数式接口）是学习 Java8 lambda 表达式的关键

### 函数式接口的定义：

- 任何接口，如果只包含唯一一个抽象方法，那么它就是一个函数式接口
- 对于函数式接口，我们可以通过 lambda 表达式来创建该接口的对象

```java
public class TestLambda {

    // 3.静态内部类
    static class Like2 implements ILike {
        @Override
        public void lambda(int a) {
            System.out.println("I like lambda2" + a);
        }
    }

    public static void main(String[] args) {
        ILike like = new Like();
        like.lambda(1);

        like = new Like2();
        like.lambda(2);

        // 4.局部内部类
        class Like3 implements ILike {
            @Override
            public void lambda(int a) {
                System.out.println("I like lambda3" + a);
            }
        }

        like = new Like3();
        like.lambda(3);

        // 5.匿名内部类，没有类名称，必须借助接口或者父类
        like = new ILike() {
            @Override
            public void lambda(int a) {
                System.out.println("I like lambda4" + a);
            }
        };
        like.lambda(4);

        // 6.用 lambda 简化
        like = (int a) -> {
            System.out.println("I like lambda5" + a);
        };
        like.lambda(5);

        // 7.简化 lambda 参数类型
        like = (a) -> {
            System.out.println("I like lambda6" + a);
        };
        like.lambda(6);

        // 8.简化 lambda 括号
        like = a -> {
            System.out.println("I like lambda7" + a);
        };
        like.lambda(7);

        // 9.简化 lambda 花括号
        like = a -> System.out.println("I like lambda8" + a);
        like.lambda(8);
    }
}

// 1.定义一个函数式接口
interface ILike {
    void lambda(int a);
}

// 2.实现类
class Like implements ILike {

    @Override
    public void lambda(int a) {
        System.out.println("I like lambda" + a);
    }
}
```

### 总结

1. lambda 表达式只能有一行代码的情况下才能简化成为一行，如果有多行，那么就用代码块包裹。
2. 前提是接口为函数式接口。
3. 多个参数也可以去掉参数类型，要去掉就都去掉，必须加上括号