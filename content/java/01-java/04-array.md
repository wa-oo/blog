## 1、数组概述

### 数组的定义：

- 数组是相同类型数据的有序集合
- 数组描述的是相同类型的若干个数据，按照一定的先后次序排列组合而成
- 其中，有一个数据称作一个数据元素，每个数据元素可以通过一个小标来访问它们

## 2、数组声明创建

首先必须声明数组变量，才能在程序中使用数组。

- 声明数组变量的语法：

  ```java
  dataType[] arrayRefVar;  // 首选的方法
  dataType arrayRefVar[];  // 效果相同，但不是首选方法
  ```

- Java 语言使用 new 操作符来创建数组，语法：

  ```java
  dataType[] arrayRefVar = new dataType[arraySize];
  ```

- 数组的元素是通过索引访问的，数组索引从 0 开始

- 获取数组长度：`arrays.length`

```java
// 声明一个数组
int[] nums;

// 创建一个数组
nums = new int[3];

// 给数组赋值
nums[0] = 1;
nums[1] = 2;
nums[3] = 2;
```

### 内存分析：

![image-20210326184648764](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210326184648764.png)

### 三种初始化方式：

- 静态初始化

  ```java
  int[] a = {1, 2, 3};
  Man[] mans = {new Man(1, 1), new Man(2, 2)};
  ```

- 动态初始化

  ```java
  int[] a = new int[2];
  a[0] = 1;
  a[1] = 2;
  ```

- 默认初始化

  - 数组是引用类型，它的元素相当于类的实例变量，因此数组一经分配空间，其中的每个元素也被按照实例变量同样的方式被隐式初始化

### 四个基本特点：

- 其长度是确定的，数组一旦被创建，它的大小就是不可以改变的。
- 其元素必须是相同的类型，不允许出现混合类型。
- 数组中的元素可以是任何数据类型，包括基本类型和引用类型。
- 数组变量属于引用类型，数组也可以看成是对象，数组中的每个元素相当于该对象的成员变量。数组本身就是对象，Java 中对象是在堆中的，因此数组无论保存原始类型还是其他对象类型，**数组对象本身是在堆中的。**

### 数组边界：

- 下标的合法区间：`[0, length - 1]`，如果越界就会报错；

	```java
	public static void main(String[] args) {
	  int[] a = new int[2];
	  System.out.println(a[2]);
	}
	```

- `ArrayIndexOutOfBoundsException`：数组下标越界异常！

### 小结：

- 数组事相同数据类型（数据类型可以为任意类型）的有序集合
- 数组是对象，数组元素相当于对象的成员变量
- 数组长度是确定的，不可变的。如果越界，则报：ArrayIndexOutOfBoudsException

## 3、数组使用

```java
int[] arrays = {1, 2, 3};

// 输出数组全部元素
for (int array : arrays) {
  System.out.println(array);
}

// 计算所有元素的和
int sum = 0;
for (int array : arrays) {
  sum += array;
}

// 查找最大元素
int max = arrays[0];
for (int i = 1; i < arrays.length; i++) {
  if (arrays[i] > max) {
    max = arrays[i];
  }
}

// 反转数组
public int[] reverse(int[] arrays) {
  int[] result = new int[arrays.length];
  
  // 反转操作
  for (int i = 0, j = result.length - 1; i < arrays.length; i++) {
    result[j] = arrays[i];
  }
  
  return result;
}
```

## 4、多维数组

- 多维数组可以看成数组的数组，比如二维数组就是一个特殊的一堆数组，其每一个元素都是一个一维数组。
- 二维数组：`int a[][] = new int[2][5];`
- 解析：二维数组 a 可以看成一个两行三列的数组

![image-20210326202257591](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210326202257591.png)

## 5、Arrays 类

- 数组的工具类 java.util.Arrays
- 由于数组对象本身并没有什么方法可以供我们调用，但 API 中提供了一个工具类 Arrays 供我们使用，从而可以对数据对象进行一些基本的操作
- Arrays 类中的方法都是 static 修饰的静态方法，在使用的时候可以直接使用类名进行调用，而“不用”使用对象来调用（是“不用”不是“不能”）
- 具有以下常用功能：
	- 给数组赋值：通过 fill 方法
	- 对数组排序：通过 sort 方法，按升序
	- 比较数组：通过 equals 方法比较数组中元素值是否相等
	- 查找数组元素：通过 binarySearch 方法能对排序好的数组进行二次查找法操作

```java
int[] a = {1, 3, 2, 8, 5};

// 打印数组元素
System.out.println(Arrays.toString(a));
```

### 冒泡排序：

- 两层循环，外层冒泡轮数，里层依次比较

```java
    // 冒泡排序
    // 1.比较数组中，两个相邻的元素，如果第一个数比第二个数大，就交换它们的位置
    // 2.每一次比较，都会产生出一个最大或者最小的数字
    // 3.下一轮则可以少一次排序
    // 4.一次循环，直到结束！
    public static int[] sort(int[] array) {
        // 临时变量
        int temp = 0;
        // 外层循环，判断循环需要走多少次
        for (int i = 0; i < array.length - 1; i++) {

            // 通过 flag 标识减少没有意义的比较
            boolean flag = false;

            // 内层循环，比较判断两个数，如果第一个数比第二个数大，则交换位置
            for (int j = 0; j < array.length - 1 - i; j++) {
                // > 由大到小，< 由小到大
                if (array[j + 1] > array[j]) {
                    temp = array[j];
                    array[j] = array[j + 1];
                    array[j + 1] = temp;
                    flag = true;
                }
            }

            if (flag == false) {
                break;
            }
        }
        return array;
    }
```

## 6、稀疏数组

### 介绍：

- 当一个数组中大部分元素为0，或者为同一值的数组时，可以使用稀疏数组来保存该数组
- 稀疏数组的处理方式是：
	- 记录数组一共有几行几列，有多少个不同值
	- 把具有不同值的元素和行列及值记录在一个小规模的数组中，从而缩小程序的规模
- 如图：左边是原始数组，右边是稀疏数组

![image-20210326212459606](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210326212459606.png)

```java
// 1.创建一个二维数组 11*11，0：没有棋子，1：黑棋，2：白棋
int[] array = new int[11][11];
array[1][2] = 1;
array[2][3] = 2;

// 转换为稀疏数组保存
// 获取有效值的个数
int sum = 0;
for (int i = 0; i < array.length; i ++){
  for (int j = 0; j < array[i].length; j ++){
  	if (array[i][j] != 0) {
      sum++;
    }
	}
}

// 2.创建一个稀疏数组的数组
int[][] arr = new int[sum + 1][3];
arr[0][0] = 11;
arr[0][1] = 11;
arr[0][2] = sum;

// 遍历二维数组，将非零的值，存放稀疏数组中
int count = 0;
for (int i = 0; i < array.length; i ++){
  for (int j = 0; j < array[i].length; j ++){
  	if (array[i][j] != 0) {
      count++;
      arr[count][0] = i;
      arr[count][1] = i;
      arr[count][2] = array[i][j];
    }
	}
}

// 还原稀疏数组
// 1.读取稀疏数组
int[][] arrs = new int[arr[0][0]][arr[0][1]];
// 2.给元素还原值
for (int i = 1; i < arr.length; i ++){
  arrs[arr[i][0]][arr[i][1]] = arr[i][2];
}
```

