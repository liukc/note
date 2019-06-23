# Java编程思想

## 第三章 操作符

### 1.赋值

#### 别名现象：

```java
	    Cl cl = new Cl(10);
        Cl cl1 = new Cl(20);
        System.out.println("cl: "+cl+"cl1: "+cl1);
        cl = cl1;
        cl.setLevel(33);
        System.out.println("cl: "+cl+"cl1: "+cl1);
```

```
cl: Cl{level=10}   cl1: Cl{level=20}
cl: Cl{level=33}   cl1: Cl{level=33}
```

### 2.逻辑操作符

Java不允许我们将一个数字作为布尔值(boolean)使用

#### 短路：

当使用逻辑操作符时，一旦能够明确无误地确定整个表达式的值，就不再计算表达式余下部分

### 3.垃圾回收机制

Java的垃圾回收器只知道释放那些经由new分配的内存。其他只能用

```java
finalize()
```

释放。

1. 垃圾回收只与内存有关
2. 对象可能不被垃圾回收
3. 垃圾回收不等于“析构”

