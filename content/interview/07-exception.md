## throw 和 throws 的区别？

**throw**：是真实抛出一个异常。

**throws**：是声明可能会抛出一个异常。

## final、finally、finalize 有什么区别？

**final**：是修饰符，如果修饰类，此类不能被继承；如果修饰方法和变量，则表示此方法和此变量不能在被改变，只能使用。

**finally**：是 try{} catch{} finally{} 最后一部分，表示不论发生任何情况都会执行，finally 部分可以省略，但如果 finally 部分存在，则一定会执行 finally 里面的代码。

**finalize**： 是 Object 类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法。

## try-catch-finally 中哪个部分可以省略？

try-catch-finally 其中 catch 和 finally 都可以被省略，但是不能同时省略，也就是说有 try 的时候，必须后面跟一个 catch 或者 finally。

## try-catch-finally 中，如果 catch 中 return 了，finally 还会执行吗？

finally 一定会执行，即使是 catch 中 return 了，catch 中的 return 会等 finally 中的代码执行完之后，才会执行。

## 常见的异常类有哪些？

**NullPointerException**：空指针异常

**ClassNotFoundException**：指定类不存在

**NumberFormatException**：字符串转换为数字异常

**IndexOutOfBoundsException**：数组下标越界异常

**ClassCastException**：数据类型转换异常

**FileNotFoundException**：文件未找到异常

**NoSuchMethodException**：方法不存在异常

**IOException IO**：异常

**SocketException Socket**：异常
