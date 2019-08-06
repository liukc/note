# Java编程思想

## 第三章、操作符

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

### 3.垃圾回收机制

Java的垃圾回收器只知道释放那些经由new分配的内存。其他只能用

```java
finalize()
```

释放。

1. 垃圾回收只与内存有关
2. 对象可能不被垃圾回收
3. 垃圾回收不等于“析构”



### 4. 自动递增和自动递减

```java

++a;  // 前缀递增
a++;  // 后缀递增
```

前缀递增表示 `++` 操作符位于变量或表达式的前面， 后缀递减表示 `++` 操作符位于变量或表达式后面

对于前缀递增，会先执行运算，再生成值；

对于后缀递增，会先生成值，再执行运算。

### 5. 短路

一旦能够明确无误地确定整个表达式的值，就不再计算表达式余下的内容

“&&”  和  “||” 会造成短路现象，“&” 和 “|”不会



### 6. 直接常量

指定常量的类型，与大小写无关

```java
int a = 0X2f; // 十六进制常量
int b = 0177; // 八进制常量
long c = 200L; // 指明为long 类型
float d = 1F; // 指定为float 类型
double e = 1d; // 指定为double类型
```

### 7. 指数计数法

```java
float e = 1.39e-47f;
double e = 47e47D; // d 可有可无
```

## 第五章、初始化与清理

### 1. 方法重载

规则：**每个重载的方法都必须有独一无二的参数类型列表**（参数顺序不同也可以，但会导致代码难以维护）

### 2. 构造器

如果创建一个类，但没有声明构造方法，Java会自动生成一个无参默认构造器

如果有声明构造方法（无论有参还是无参），Java都不会再自动生成一个默认构造器

构造方法没有返回值

### 3. this 关键字

在构造器中，如果为 `this` 添加参数列表， 将是调用其他构造器

```java
public class InitClass{
    InitClass(){}
    
    InitClass(int a){}
    
    InitClass(float f){
        this(10)			// 调用另一个构造器
    }
}
```

在构造器中：

- 只能用this 调用一个构造器， 不能调用两个
- 必须将构造器调用置于最起始处，否则编译会报错

### 4. static 关键字

1. 不可以从static方法内部发出对非static方法的调用。
2. 在没有任何对象的前提下，我们可针对类本身发出对一个static方法的调用
3. 一个类之中，先执行static方法，如果有static代码块，先执行static代码块

### 5. 包：库单元

`package my-package; `

package语句必须作为文件的第一个非注释语句出现，该语句的作用是指出这个编译单元属于名为my-package的一个库的一部分

## 第七章、复用类

1. 编译器并不是简单地为每一个引用都创建默认对象，如果想初始化这些引用，可以在代码的以下位置进行

   - 在定义对象的地方。这意味着他们总能在构造器被调用之前被初始化
   - 在类的构造器中
   - 就在正要使用这些对象之前，这种方法被称为惰性初始化
   - 使用实例初始化

   ```java
   public class GoodMorning{
       private String str = "再定义对象的地方初始化";
       private String str2;
       private String str3;
       private String str4;
       
       public GoodMorning(){
           str2 = "在类的构造器中初始化";
       }
       
       public String toString(){
           if(str3 == null){
               str3 = "惰性初始化";
           }
       }
       {
           str4 = "实例初始化";
       }
   }
   ```

2. 继承并不只是复制基类的接口，当创建一个导出类的对象时，该对象包含了一个基类的子对象。这个子对象和用基类直接创建的对象是一样的。二者的区别在于，后者来自于外部，而基类的子对象被包装在导出类对象的内部

3. 继承的构建过程是从基类“向外”扩散的，所以基类在导出类构造器可以访问它之前，就已经完成了初始化

4. 代理使类能达到复用的目的

   ```java
   class Fly{
       public void forward(); //向前方飞
       public void down(); //向下飞
   }
   
   
   public class Bird{
       private Fly fly;
       
       public Bird(){
           fly = new Fly();
       }
       // 使用代理完成飞这个具体动作
       public void forward(){
           fly.forward();
       }
       public void down(){
           fly.down();
       }
   }
   ```

   

5. 该用组合还是继承：当需要从新类向基类进行向上转型，就用继承，否则就要考虑组合的情况





## 第八章、多态

### 1. 向上转型

```java
package com.forlkc;

class Upper{
    private int id;

    public void say(){
        System.out.println("I am Upper");
    }

    public Upper(){
        System.out.println("Upper was initialized");
    }
}

public class ConvertUpper extends Upper{
    private int id;

    public ConvertUpper(){
        System.out.println("Convert was initialized");
    }
    @Override
    public void say() {
        System.out.println("I am convertUpper");
    }

    public void sayHi(){
        System.out.println("convert say hi");
    }

    public static void main(String[] args) throws Throwable {
        Upper convertUpper = new ConvertUpper();

        convertUpper.say();
        // convertUpper.sayHi();

    }
}
```

```cmd
Upper was initialized
Convert was initialized
I am convertUpper

Process finished with exit code 0

```

从运行结果来看，向上转型之后，这个类还是属于子类，他会先初始化父类，再初始化子类，但是只能用父类已有的能继承的方法，如果子类重写了该方法，用的是子类已经重写的方法，而不是父类的方法。所以，向上转型相当于缩小了子类的方法。

## 第九章、接口

1. 接口的方法默认为`public`  如果里面有域的话，默认为 `final` 和 `static`

```java
package com.forlkc.infa;

public interface Fish {
    public static final String name = "fish";
	int a = 2;  // pulic static final
    void f();// public

    public void m();
}
```

## 第十章、内部类

### 1. 内部类创建

内部类无法直接初始化，需要借助外部类的方法创建内部类的实例

```java
package com.forlkc;

import java.util.List;

public class OuterClassInnerClass {
    private String name;
    class Inner{
        private String name;
        public Inner(){
            ;
        }
        public Inner(String s){
            this.name = s;
        }

        public void outputName(){
            System.out.println("Inner class name is : " + name);
        }
    }
    public Inner getInner(String args){
        return new Inner(args);
    }
    public void outputName(){
        System.out.println("outer name is : " + name);
    }

    public static void main(String[] args) {
        OuterClassInnerClass outer = new OuterClassInnerClass();
        OuterClassInnerClass.Inner inner = outer.getInner("inner");
        //Inner inner1 = new Inner();
        // 在拥有外部类对象之前是不能创建内部类对象的。因为内部类会连接到创建它的外部类对象上
        // 如果，创建的是嵌套类（静态内部类），就不需要对外部类对象的引用
        OuterClassInnerClass.Inner inner1 = outer.new Inner("inner2");

        outer.outputName();
        inner.outputName();
        inner1.outputName();
    }
}
```

## 第十一章、持有对象

### 1. List

List 承诺可以将元素维护在特定的序列之中

有两种类型的List：

- ArrayList：长于随机访问元素，但在List的中间插入和移除元素比较慢
- LinkedList：它在List中间插入和删除元素的代价比较低，提供了优化的顺序访问。LinkedList 在随机访问方面相对比较慢，但是它的特性集较ArrayList更大

### 2. Iterator

Java的iterator只能单向移动，且它只能用来：

1. 使用方法iterator() 要求容器返回一个iterator。iterator将准备好返回序列的第一个元素
2. 使用next() 获得序列中的下一个元素
3. 使用hasNext() 检查序列中是否还有元素
4. 使用remove() 将迭代器新近返回的元素删除
5. 