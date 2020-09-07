# OnJava8

## 函数式编程

### lambda表达式

Lambda 表达式是使用**最小可能**语法编写的函数定义：

1. Lambda 表达式产生函数，而不是类。 在 JVM（Java Virtual Machine，Java 虚拟机）上，一切都是一个类，因此在幕后执行各种操作使 Lambda 看起来像函数 —— 但作为程序员，你可以高兴地假装它们“只是函数”。
2. Lambda 语法尽可能少，这正是为了使 Lambda 易于编写和使用。

#### 基本语法

- 参数
  - 当只用一个参数，可以不用（）
  - 正常情况使用（）包裹参数。为了保持一致性，也可以用括号（）包裹单个参数
  - 如果没有参数，则必须使用（）表示空参数列表
  - 对于多个参数，则将参数列表放在括号（）中
- ->  ， 可视为产出
- ->  之后的内容都是方法体
- 当方法体为单行时，该表达式的结果自动成为Lambda表达式的返回值，这时候使用 **return** 是非法的
- 如果在 Lambda 表达式中确实需要多行，则必须将这些行放在花括号中。 在这种情况下，就需要使用 **return**

#### 递归

递归函数是一个自我调用的函数。可以编写lambda表达式，但需要注意的是，递归方法必须是实例变量或静态变量，否则会出现编译时错误

示例1：

```java 
public class Lambda {
    interface InCall {
        int call(int n);
    }

    static InCall fact;
    
    public static void main(String[] args) {
        fact = n -> n == 0 ? 1 : n * fact.call(n - 1);
        System.out.println(fact.call(10));
    }
}
```

示例2：

```java 
public class Lambda {
    interface InCall {
        int call(int n);
    }

    static InCall fact;
    
    public static void main(String[] args) {
        fact = n -> n == 0 ? 0 : n == 1 ? 1 : fact.call(n-1) + fact.call(n-2);
        System.out.println(fact.call(10));
    }
}
```

### 方法引用

#### Runnable接口

```java 
class Describe {
    static void show() {
        System.out.println("hello MethodReferences");
    }
}

public class MethodReferences {

    public static void main(String[] args) {
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(5,5,10, TimeUnit.DAYS,
                new LinkedBlockingDeque<>(10));
        threadPoolExecutor.execute(Describe::show);
        threadPoolExecutor.shutdown();
    }
}
```

#### 未绑定的方法引用

使用未绑定的引用时，函数式方法的签名（接口中的单个方法）不再与方法引用的签名完全匹配。 原因是：你需要一个对象来调用方法。

#### 构造函数引用

```java 
class Dog {
  String name;
  int age = -1; // For "unknown"
  Dog() { name = "stray"; }
  Dog(String nm) { name = nm; }
  Dog(String nm, int yrs) { name = nm; age = yrs; }
}

interface MakeNoArgs {
  Dog make();
}

interface Make1Arg {
  Dog make(String nm);
}

interface Make2Args {
  Dog make(String nm, int age);
}

public class CtorReference {
  public static void main(String[] args) {
    MakeNoArgs mna = Dog::new; // [1]
    Make1Arg m1a = Dog::new;   // [2]
    Make2Args m2a = Dog::new;  // [3]

    Dog dn = mna.make();
    Dog d1 = m1a.make("Comet");
    Dog d2 = m2a.make("Ralph", 4);
  }
}
```

**Dog** 有三个构造函数，函数接口内的 `make()` 方法反映了构造函数参数列表（ `make()` 方法名称可以不同）。

**注意**我们如何对 **[1]**，**[2]** 和 **[3]** 中的每一个使用 `Dog :: new`。 这 3 个构造函数只有一个相同名称：`:: new`，但在每种情况下都赋值给不同的接口。编译器可以检测并知道从哪个构造函数引用。

编译器知道调用函数式方法（本例中为 `make()`）就相当于调用构造函数

### 函数式接口

```java
public class Functional {
    @FunctionalInterface
    interface FunctionalOne{
        String onePeace(String msg);
    }
    public String msg(String msg){
        return "hello " + msg;
    }

    public static void main(String[] args) {
        Functional functional = new Functional();
        FunctionalOne functionalOne = functional::msg;
        System.out.println(functionalOne.onePeace("world"));
    }
}
```

`@FunctionalInterface` 的作用：接口中如果有多个方法则会产生编译期错误

#### 基本命名准则：

1. 如果只处理对象而非基本类型，名称则为 `Function`，`Consumer`，`Predicate` 等。参数类型通过泛型添加。
2. 如果接收的参数是基本类型，则由名称的第一部分表示，如 `LongConsumer`，`DoubleFunction`，`IntPredicate` 等，但基本 `Supplier` 类型例外。
3. 如果返回值为基本类型，则用 `To` 表示，如 `ToLongFunction <T>` 和 `IntToLongFunction`。
4. 如果返回值类型与参数类型一致，则是一个运算符：单个参数使用 `UnaryOperator`，两个参数使用 `BinaryOperator`。
5. 如果接收两个参数且返回值为布尔值，则是一个谓词（Predicate）。
6. 如果接收的两个参数类型不同，则名称中有一个 `Bi`。

下表描述了 `java.util.function` 中的目标类型（包括例外情况）：

| **特征**                                  | **函数式方法名**                                             | **示例**                                                     |
| ----------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 无参数； 无返回值                         | **Runnable** (java.lang) `run()`                             | **Runnable**                                                 |
| 无参数； 返回类型任意                     | **Supplier** `get()` `getAs类型()`                           | **Supplier`<T>` BooleanSupplier IntSupplier LongSupplier DoubleSupplier** |
| 无参数； 返回类型任意                     | **Callable** (java.util.concurrent) `call()`                 | **Callable`<V>`**                                            |
| 1 参数； 无返回值                         | **Consumer** `accept()`                                      | **`Consumer<T>` IntConsumer LongConsumer DoubleConsumer**    |
| 2 参数 **Consumer**                       | **BiConsumer** `accept()`                                    | **`BiConsumer<T,U>`**                                        |
| 2 参数 **Consumer**； 1 引用； 1 基本类型 | **Obj类型Consumer** `accept()`                               | **`ObjIntConsumer<T>` `ObjLongConsumer<T>` `ObjDoubleConsumer<T>`** |
| 1 参数； 返回类型不同                     | **Function** `apply()` **To类型** 和 **类型To类型** `applyAs类型()` | **Function`<T,R>` IntFunction`<R>` `LongFunction<R>` DoubleFunction`<R>` ToIntFunction`<T>` `ToLongFunction<T>` `ToDoubleFunction<T>` IntToLongFunction IntToDoubleFunction LongToIntFunction LongToDoubleFunction DoubleToIntFunction DoubleToLongFunction** |
| 1 参数； 返回类型相同                     | **UnaryOperator** `apply()`                                  | **`UnaryOperator<T>` IntUnaryOperator LongUnaryOperator DoubleUnaryOperator** |
| 2 参数类型相同； 返回类型相同             | **BinaryOperator** `apply()`                                 | **`BinaryOperator<T>` IntBinaryOperator LongBinaryOperator DoubleBinaryOperator** |
| 2 参数类型相同; 返回整型                  | Comparator (java.util) `compare()`                           | **`Comparator<T>`**                                          |
| 2 参数； 返回布尔型                       | **Predicate** `test()`                                       | **`Predicate<T>` `BiPredicate<T,U>` IntPredicate LongPredicate DoublePredicate** |
| 参数基本类型； 返回基本类型               | **类型To类型Function** `applyAs类型()`                       | **IntToLongFunction IntToDoubleFunction LongToIntFunction LongToDoubleFunction DoubleToIntFunction DoubleToLongFunction** |
| 2 参数类型不同                            | **Bi操作** (不同方法名)                                      | **`BiFunction<T,U,R>` `BiConsumer<T,U>` `BiPredicate<T,U>` `ToIntBiFunction<T,U>` `ToLongBiFunction<T,U>` `ToDoubleBiFunction<T>`** |

此表仅提供些常规方案。通过上表，能自行推导出更多行的函数式接口。

### 高阶函数

高阶函数（Higher-order Function）：一个消费或产生函数的函数

```java 
interface Funcc extends Function<String, String> {
}
public class ProduceFunction {
    static Funcc produce() {
        return String::toLowerCase;
    }

    public static void main(String[] args) {
        Funcc funcc = produce();
        System.out.println(funcc.apply("YELLOW"));
    }
}
```

### 闭包

考虑一个更复杂的lambda，它使用函数作用域之外的变量。返回该函数会发发生什么？也就是说，当你调用函数时，它对那些外部变量引用了什么？如果语言不能自动解决这个问题，那将变得非常具有挑战性。能够解决这个问题的语言被称为**支持闭包**，或者叫做在词法上限定范围（变量捕获）。Java8提供了有限但合理的闭包支持。

### 函数组合

函数组合（Function Composition）意为“多个函数组合成新函数”。它通常是函数式编程的基本组成部分。在前面的 `TransformFunction.java` 类中，有一个使用 `andThen()` 的函数组合示例。一些 `java.util.function` 接口中包含支持函数组合的方法。

| 组合方法                                         | 支持接口                                                     |
| ------------------------------------------------ | ------------------------------------------------------------ |
| `andThen(argument)` 根据参数执行原始操作         | **Function BiFunction Consumer BiConsumer IntConsumer LongConsumer DoubleConsumer UnaryOperator IntUnaryOperator LongUnaryOperator DoubleUnaryOperator BinaryOperator** |
| `compose(argument)` 根据参数执行原始操作         | **Function UnaryOperator IntUnaryOperator LongUnaryOperator DoubleUnaryOperator** |
| `and(argument)` 短路**逻辑与**原始谓词和参数谓词 | **Predicate BiPredicate IntPredicate LongPredicate DoublePredicate** |
| `or(argument)` 短路**逻辑或**原始谓词和参数谓词  | **Predicate BiPredicate IntPredicate LongPredicate DoublePredicate** |
| `negate()` 该谓词的**逻辑否**谓词                | **Predicate BiPredicate IntPredicate LongPredicate DoublePredicate** |

## 流式编程

集合优化了对象的存储，而流和对象的处理有关

流是一系列与特定存储机制无关的元素——实际上，流并没有存储之说

利用流，我们无需迭代集合中的元素，就可以提取和操作它们。这些管道通常被组合在一起，在流上形成一条操作管道

在大多数情况下，将对象存储在集合中是为了处理他们，因此你将会发现你将把编程的主要焦点从集合转移到了流上。流的一个核心好处是，它使得程序更加短小并且更易理解。当 Lambda 表达式和方法引用（method references）和流一起使用的时候会让人感觉自成一体。流使得 Java 8 更具吸引力。

```java 
public class Randoms {
    public static void main(String[] args) {
        new Random().ints(2, 50).distinct().limit(7).sorted().forEach(System.out::println);
    }
}
```

流式编程采用内部迭代，这是流式编程的核心特性之一。这种机制使得编写的代码可读性更强，也更能利用多核处理器的优势。通过放弃对迭代过程的控制，我们把控制权交给并行化机制

流是懒加载的。这代表着它只在绝对必要时才计算。你可以将流看作“延迟列表”。由于计算延迟，流使我们能够表示非常大（甚至无限）的序列，而不需要考虑内存问题。

### 流创建

通过 `Stream.of()` 很容易地将一组元素转化成为流，除此之外，每个集合都可以通过调用 `stream()` 方法来产生一个流

```java 
public class Randoms {
    public static void main(String[] args) {
        int[] array = new Random().ints(2, 50).distinct().limit(7).sorted().toArray();
        Stream.of(array).forEach(System.out::println);
        Map<String, String> map = new HashMap<>();
        map.put("a", "A");
        map.put("b", "B");
        map.entrySet().stream().map(entry->entry.getKey() + ":" + entry.getValue()).forEach(System.out::println);
    }
}
```

中间操作 `map()` 会获取流中的所有元素，并且对流中元素应用操作从而产生新的元素，并将其传递到后续的流中。通常 `map()` 会获取对象并产生新的对象

#### 随机数流

```java 
public class RandomGenerators {
    public static <T> void show(Stream<T> stream) {
        stream
        .limit(4)
        .forEach(System.out::println);
        System.out.println("++++++++");
    }

    public static void main(String[] args) {
        Random rand = new Random(47);
        show(rand.ints().boxed());
        show(rand.longs().boxed());
        show(rand.doubles().boxed());
        // 控制上限和下限：
        show(rand.ints(10, 20).boxed());
        show(rand.longs(50, 100).boxed());
        show(rand.doubles(20, 30).boxed());
        // 控制流大小：
        show(rand.ints(2).boxed());
        show(rand.longs(2).boxed());
        show(rand.doubles(2).boxed());
        // 控制流的大小和界限
        show(rand.ints(3, 3, 9).boxed());
        show(rand.longs(3, 12, 22).boxed());
        show(rand.doubles(3, 11.5, 12.3).boxed());
    }
}
```

`boxed()` 流操作将会自动地把基本类型包装成为对应的装箱类型

#### int 类型的范围

`IntStream` 类提供了 `range()` 方法用于生成整型序列的流。编写循环时，这个方法会更加便利

```java
public class Ranges {
    public static void main(String[] args) {
        System.out.println(IntStream.range(10, 20).sum());
    }
}
```

#### generate()

```java
public class Ranges {
    public static void main(String[] args) {
        Stream.generate(()->"lkc").limit(3).map((a)-> a.charAt(new Random().nextInt(3))).forEach(System.out::println);
    }
}
```

#### iterate()

**Stream.**`iterate()` 以种子（第一个参数）开头，并将其传给方法（第二个参数）。方法的结果将添加到流，并存储作为第一个参数用于下次调用 `iterate()`，依次类推

```java 
public class Fibonacci {
    int x = 1;

    Stream<Integer> numbers() {
        return Stream.iterate(0, i -> {
            int result = x + i;
            x = i;
            return result;
        });
    }

    public static void main(String[] args) {
        new Fibonacci().numbers()
                       .skip(20) // 过滤前 20 个
                       .limit(10) // 然后取 10 个
                       .forEach(System.out::println);
    }
}
```

`skip()` ：根据参数丢弃指定数量的流元素

#### 流的建造者模式

在建造者设计模式（也称构造器模式）中，首先创建一个 `builder` 对象，传递给它多个构造器信息，最后执行“构造”。**Stream** 库提供了这样的 `Builder`

```java 
public class FileToWordsBuilder {
    Stream.Builder<String> builder = Stream.builder();

    public FileToWordsBuilder(String filePath) throws Exception {
        Files.lines(Paths.get(filePath))
             .skip(1) // 略过开头的注释行
             .forEach(line -> {
                  for (String w : line.split("[ .?,]+"))
                      builder.add(w);
              });
    }

    Stream<String> stream() {
        return builder.build();
    }

    public static void main(String[] args) throws Exception {
        new FileToWordsBuilder("Cheese.dat")
            .stream()
            .limit(7)
            .map(w -> w + " ")
            .forEach(System.out::print);
    }
}
```

**注意**，构造器会添加文件中的所有单词（除了第一行，它是包含文件路径信息的注释），但是其并没有调用 `build()`。只要你不调用 `stream()` 方法，就可以继续向 `builder` 对象中添加单词。

在该类的更完整形式中，你可以添加一个标志位用于查看 `build()` 是否被调用，并且可能的话增加一个可以添加更多单词的方法。在 `Stream.Builder` 调用 `build()` 方法后继续尝试添加单词会产生一个异常。

#### Arrays

`Arrays` 类中含有一个名为 `stream()` 的静态方法用于把数组转换成为流。

#### 正则表达式

ava 8 在 `java.util.regex.Pattern` 中增加了一个新的方法 `splitAsStream()`。这个方法可以根据传入的公式将字符序列转化为流。但是有一个限制，输入只能是 **CharSequence**，因此不能将流作为 `splitAsStream()` 的参数。

### 中间操作

中间操作用于从一个流中获取对象，并将对象作为另一个流从后端输出，以连接到其他操作。

#### 跟踪与调试

`peek()` 操作的目的是帮助调试。它允许你无修改地查看流中的元素

因为 `peek()` 符合无返回值的 **Consumer** 函数式接口，所以我们只能观察，无法使用不同的元素来替换流中的对象

#### 流元素排序

```java
Stream<T> sorted(Comparator<? super T> comparator);
```

#### 移除元素

- `distinct()`：在 `Randoms.java` 类中的 `distinct()` 可用于消除流中的重复元素。相比创建一个 **Set** 集合，该方法的工作量要少得多。
- `filter(Predicate)`：过滤操作会保留与传递进去的过滤器函数计算结果为 `true` 元素。

#### 应用函数到元素

- `map(Function)`：将函数操作应用在输入流的元素中，并将返回值传递到输出流中。
- `mapToInt(ToIntFunction)`：操作同上，但结果是 **IntStream**。
- `mapToLong(ToLongFunction)`：操作同上，但结果是 **LongStream**。
- `mapToDouble(ToDoubleFunction)`：操作同上，但结果是 **DoubleStream**。

#### 在 map() 中组合流

假设我们现在有了一个传入的元素流，并且打算对流元素使用 `map()` 函数。现在你已经找到了一些可爱并独一无二的函数功能，但是问题来了：这个函数功能是产生一个流。我们想要产生一个元素流，而实际却产生了一个元素流的流。

`flatMap()` 做了两件事：将产生流的函数应用在每个元素上（与 `map()` 所做的相同），然后将每个流都扁平化为元素，因而最终产生的仅仅是元素。

`flatMap(Function)`：当 `Function` 产生流时使用。

`flatMapToInt(Function)`：当 `Function` 产生 `IntStream` 时使用。

`flatMapToLong(Function)`：当 `Function` 产生 `LongStream` 时使用。

`flatMapToDouble(Function)`：当 `Function` 产生 `DoubleStream` 时使用。

