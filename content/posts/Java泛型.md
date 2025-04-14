---
date: "2025-04-14T21:06:15+08:00"
draft: false
title: "Java中的泛型"
tags: ["Java基础"]
categories: ["Java编程"]
lightgallery: true
---
# 概述

泛型可以使得在编译期发现bug,从而增加代码稳定性。泛型使得在定义类、接口和方法时类型（类和接口）可以被参数化。就像方法声明中使用的形参，类型参数对于不同的输入可以重用代码。不同点在于形参的输入是值，类型参数的输入是类型。

使用泛型有诸多益处：

- 编译器强类型检查：Java编译器对泛型代码运行强类型检查，如果代码类型不安全则报错。修复编译器错误总比修复运行时错误简单

- 消除转换：

  下面代码端需要转换

  ```java
  List list = new ArrayList();
  list.add("hello");
  String s = (String) list.get(0);
  ```

  使用泛型则不需要转换

  ```java
  List<String> list = new ArrayList<String>();
  list.add("hello");
  String s = list.get(0);   // no cast
  ```

- 提供实现通用算法的能力:通过泛型,程序员能实现集合上不同类型的通用算法,这些算法可以自定义,类型安全且更易读

# 泛型类型

泛型类型是类型参数化的泛型类或泛型接口。

以一个示例展示泛型的使用，下面非泛型的Box类适用于任何类型的对象

```java
public class Box {
    private Object object;

    public void set(Object object) { this.object = object; }
    public Object get() { return object; }
}
```

因为它的方法接受或返回一个Object对象,所以你可以传入除原始类型外的任何对象。但在运行时无法验证该类如何被使用。一部分代码可能使用Integer并期望输出Integer，同时另一部分代码可能错误地传入String,这将导致运行时错误。

泛型类的定义格式如下：

```java
class name<T1, T2, ..., Tn> { /* ... */ }
```

将泛型应用到Box类上

```java
/**
 * Generic version of the Box class.
 * @param <T> the type of the value being boxed
 */
public class Box<T> {
    // T stands for "Type"
    private T t;

    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```

类型变量可以是任何非原始类型:任意类、接口、数组甚至是其它类型变量。

**类型参数命名约定**

约定类型参数命名是单个大写字母，常见的类型参数名称如下：

- E-Element(在Java集合框架中常用)
- K-Key
- N-Number
- T-Type
- V-Value
- S,U,V等

**调用并实例化泛型类型**

要使用泛型类,必须以具体的类型值代替泛型定义。如使用Box类`Box<Integer> integerBox;`可以把泛型类调用类比普通方法调用,只不过传递的是类型参数（就像上面传递的是Integer）。

> type paramaeter与type argument术语:
>
> 编码时,type argument用来创建一个参数化的类型。如Foo\<T\>中的T是type paramaeter，Foo\<String\>中的String是type argument

泛型类型的调用也叫类型参数化。泛型类的实例化如下例子:

```java
Box<Integer> integerBox = new Box<Integer>();
//java 7之后,编译器可以从上下文中确定或推断类型参数
Box<Integer> integerBox = new Box<>();
```

**多类型参数**

```java
public interface Pair<K, V> {
    public K getKey();
    public V getValue();
}

public class OrderedPair<K, V> implements Pair<K, V> {

    private K key;
    private V value;

    public OrderedPair(K key, V value) {
	this.key = key;
	this.value = value;
    }

    public K getKey()	{ return key; }
    public V getValue() { return value; }
}

OrderedPair<String, Integer> p1 = new OrderedPair<>("Even", 8);//自动装箱使得可以传入原始类型
OrderedPair<String, String>  p2 = new OrderedPair<>("hello", "world");
```

**参数化类型**

可以将类型参数(如K,V)替换为参数化类型(如List\<String\>)

```java
OrderedPair<String, Box<Integer>> p = new OrderedPair<>("primes", new Box<Integer>(...));
```

## 原始类型

原始类型是不带任何类型参数的泛型类或泛型接口。

```java
public class Box<T> {
    public void set(T t) { /* ... */ }
    // ...
}
//参数化定义
Box<Integer> intBox = new Box<>();
//Box<T>的原始类型
Box rawBox = new Box();
```

Box是Box\<T\>的原始类型,但是非泛型类或接口不是原始类型。

原始类型在旧版代码中出现，因为那是许多API类（像Collections）不支持泛型。当使用原始类型时，实际上是一种预泛型行为:Box的f泛型参数被参数化为Object。为了向后兼容，将参数化类型赋值给原始类型是允许的：

```java
Box<String> stringBox = new Box<>();
Box rawBox = stringBox;               // OK
```

但是反过来你会得到一个警告:

```java
Box rawBox = new Box();           // rawBox is a raw type of Box<T>
Box<Integer> intBox = rawBox;     // warning: unchecked conversion
```

如果你用原始类型来调用泛型类的方法同样会得到警告:

```java
//这段代码不会报错,stringBox在运行时存储的值为<Integer>类型
Box<String> stringBox = new Box<>();
Box rawBox = stringBox;
rawBox.set(8);  // warning: unchecked invocation to set(T) 这表明原始类型绕过了泛型类型检查
```

**未受检错误信息**

当在旧版本代码中使用泛型代码时,可能出现以下警告信息:

```tex
Note: Example.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
```

这通常在旧版API中使用原始类型时出现：

```java
public class WarningDemo {
    public static void main(String[] args){
        Box<Integer> bi;
        bi = createBox();
    }

    static Box createBox(){
        return new Box();
    }
}
```

"unchecked"意味着编译器没有足够的类型信息执行所有的类型检查来确保类型安全。使用`-Xlint:unchecked`重新编译来查看所有'"unchecked"警告：

```tex
WarningDemo.java:4: warning: [unchecked] unchecked conversion
found   : Box
required: Box<java.lang.Integer>
        bi = createBox();
                      ^
1 warning
```

可以用`-Xlint:-unchecked`禁用未受检警告；`@SuppressWarnings("unchecked")`注解会抑制未受检警告。

# 泛型方法

泛型方法是引入自己的类型参数的方法。与声明泛型类型相似，但是其类型参数的作用域限制在方法内。静态、非静态泛型方法以及泛型类构造器都合法。

泛型方法的语法包含出现在方法返回类型前的类型参数列表。

```java
public class Util {
    public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
        return p1.getKey().equals(p2.getKey()) &&
               p1.getValue().equals(p2.getValue());
    }
}

public class Pair<K, V> {

    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public void setKey(K key) { this.key = key; }
    public void setValue(V value) { this.value = value; }
    public K getKey()   { return key; }
    public V getValue() { return value; }
}
```

泛型方法的调用如下:

```java
Pair<Integer, String> p1 = new Pair<>(1, "apple");
Pair<Integer, String> p2 = new Pair<>(2, "pear");
boolean same = Util.<Integer, String>compare(p1, p2);
```

当省略明确的类型时,编译器将推断具体的类型:

```java
Pair<Integer, String> p1 = new Pair<>(1, "apple");
Pair<Integer, String> p2 = new Pair<>(2, "pear");
boolean same = Util.compare(p1, p2);
```

# 边界类型参数

如果我们想限制作为类型参数的参数化类型,如一个针对数字操作的方法仅接受Number及其子类,可以使用有界类型参数。

要声明有界类型参数，列出类型参数名称，后面加上extends关键字，然后是它的上界。在这种情况下，extends是广义的意味着"extends(类)"或"implements(接口)"。

```java
public class Box<T> {

    private T t;          

    public void set(T t) {
        this.t = t;
    }

    public T get() {
        return t;
    }

    public <U extends Number> void inspect(U u){
        System.out.println("T: " + t.getClass().getName());
        System.out.println("U: " + u.getClass().getName());
    }

    public static void main(String[] args) {
        Box<Integer> integerBox = new Box<Integer>();
        integerBox.set(new Integer(10));
        //使用了有界泛型参数,编译器失败
        integerBox.inspect("some text"); // error: this is still String!
    }
}
```

有界泛型参数除了限制类型外,你还可以用起实例调用边界类型的方法:

```java
public class NaturalNumber<T extends Integer> {

    private T n;

    public NaturalNumber(T n)  { this.n = n; }

    public boolean isEven() {
        return n.intValue() % 2 == 0;
    }

    // ...
}
```

**多边界**

一个类型参数可以有多个边界:

```java
<T extends B1 & B2 & B3>
```

有多个边界的类型变量是所有边界类型的子类型。如果有一个边界是类，它必须先声明(否则编译不通过)，例如：

```java
Class A { /* ... */ }
interface B { /* ... */ }
interface C { /* ... */ }

class D <T extends A & B & C> { /* ... */ }
```

## 泛型方法中的边界类型参数

有界类型参数是实现泛型算法的关键。下面的泛型使用会出现编译错误：

```java
public static <T> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e > elem)  // compiler error
            ++count;
    return count;
}
```

因为">"操作符只能运用于原始类型,可以使用以Comparable\<T\>为上界的类型参数修复:

```java
public static <T extends Comparable<T>> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e.compareTo(elem) > 0)
            ++count;
    return count;
}
```

# 泛型中的继承和子类型

把一种类型的对象赋值给另一种类型的对象在某些情况下是可行的。例如，你可以把Integer的对象赋值给Object的，因为Object是Integer的超类：

```java
Object someObject = new Object();
Integer someInteger = new Integer(10);
someObject = someInteger;   // OK
```

因为Integer也是Number的子类型,所以以下代码也合法:

```java
public void someMethod(Number n) { /* ... */ }

someMethod(new Integer(10));   // OK
someMethod(new Double(10.1));   // OK
```

泛型也适用以上的赋值法则:

```java
Box<Number> box = new Box<Number>();
box.add(new Integer(10));   // OK
box.add(new Double(10.1));  // OK
```

考虑以下代码:

```java
public void boxTest(Box<Number> n) { /* ... */ }
```

该方法不能接受Box\<Integer\>或Box\<Double\>为入参,因为它们都不是Box\<Number\>的子类型。

> 给定类型A和B，MyClass\<A\>和MyClass\<B\>没有从属关系,它们的父类都是Object。

**泛型类和其子类**

你可以通过继承或实现来子类化一个泛型类或接口。

以Collections类举例,`ArrayList<E>`实现`List<E>`,`List<E>`继承`Collection<E>`。

自定义一个list接口`PayloadList`:

```java
interface PayloadList<E,P> extends List<E> {
  void setPayload(int index, P val);
  ...
}
```

下列参数化的PayloadList是List\<String\>的子类型:

- PayloadList<String,String>
- PayloadList<String,Integer>
- PayloadList<String,Exception>

# 类型推断

类型推断是Java编译器确定具体的类型参数来使方法调用和声明可用的能力。类型推断算法确定参数的类型，该类型赋值给返回值或直接返回。最后，推断算法找到适用于所有参数的最合适的类型。

下面的例子，推断确定了传递给pick方法的两个参数是`Serializable`类型。

```java
static <T> T pick(T a1, T a2) { return a2; }
Serializable s = pick("d", new ArrayList<String>());
```

## 泛型方法中的类型推断

类型推断可以让你像调用普通方法一样调用泛型方法。看看下面的例子：

```java
public class BoxDemo {

  public static <U> void addBox(U u, 
      java.util.List<Box<U>> boxes) {
    Box<U> box = new Box<>();
    box.set(u);
    boxes.add(box);
  }

  public static <U> void outputBoxes(java.util.List<Box<U>> boxes) {
    int counter = 0;
    for (Box<U> box: boxes) {
      U boxContents = box.get();
      System.out.println("Box #" + counter + " contains [" +
             boxContents.toString() + "]");
      counter++;
    }
  }

  public static void main(String[] args) {
    java.util.ArrayList<Box<Integer>> listOfIntegerBoxes =
      new java.util.ArrayList<>();
    BoxDemo.<Integer>addBox(Integer.valueOf(10), listOfIntegerBoxes);
    BoxDemo.addBox(Integer.valueOf(20), listOfIntegerBoxes);
    BoxDemo.addBox(Integer.valueOf(30), listOfIntegerBoxes);
    BoxDemo.outputBoxes(listOfIntegerBoxes);
  }
}

//输出
Box #0 contains [10]
Box #1 contains [20]
Box #2 contains [30]
```

addBox方法定义了一个名为U的类型参数,通常Java编译器能推断调用的泛型方法的类型参数。所以多数情况下不用具体说明。例如，调用addBox方法,你可以使用具体的类型参数:

```java
BoxDemo.<Integer>addBox(Integer.valueOf(10), listOfIntegerBoxes);
```

相应的,如果你省略类型参数,Java编译器从方法参数中自动推断类型参数的类型为Integer:

```java
BoxDemo.addBox(Integer.valueOf(20), listOfIntegerBoxes);
```

## 实例化泛型类中的类型推断

泛型类实例化时,你可以在实例化的`<>`符号中省略类型参数,但是你不能省略`<>`符号,否则会被认为是原始类型:

```java
Map<String, List<String>> myMap = new HashMap<String, List<String>>();
//省略类型参数
Map<String, List<String>> myMap = new HashMap<>();
//未受检警告
Map<String, List<String>> myMap = new HashMap(); // unchecked conversion warning
```

## 泛型构造器中的类型推断

泛型构造器可以存在于泛型类和非泛型类中，泛型构造器可以有自己的形式类型参数。如：

```java
class MyClass<X> {
  <T> MyClass(T t) {
    // ...
  }
}
```

考虑下面MyClass的实例化:

```java
MyClass<Integer> myObject1 = new MyClass<Integer>("")
//省略版
MyClass<Integer> myObject2 = new MyClass<>("");
```

该语句定义了参数化类型的MyClass\<Integer\>,它表明类型参数X的类型是Integer。同时，编译器推断类型参数T的类型是String。

## 目标类型

Java编译器利用目标输入来判断泛型方法调用的类型参数。表达式的目标类型是Java编译器期望的数据类型，具体取决于表达式出现的位置。考虑`Collections.emptyList`方法，其定义如下：

```java
static <T> List<T> emptyList();
```

考虑下面的赋值语句:

```java
List<String> listOne = Collections.emptyList();
```

该表达式期望一个`List<String>`的实例,这种数据类型是目标类型。因为emptyList方法返回List\<T\>,编译器推断类型参数T必须是String。这在Java7和Java8中都适用。

但以下例子存在不同：

```java
void processStringList(List<String> stringList) {
    // process stringList
}

//这在Java7中无法编译,因为List<T>在Java7中认为需要List<Object>
processStringList(Collections.emptyList());
//在Java7中可以编译通过
processStringList(Collections.<String>emptyList());
```

# 通配符

泛型代码中,"?"叫作通配符，代表着一种未知类型。通配符可以在众多场景中使用：作为从参数类型、字段或者本地变量；有时甚至可以是返回类型。通配符不能用在泛型方法调用的类型参数、泛型类实例创建或超类。

## 上界通配符

你可以使用上界通配符放大变量的限制。例如你想写一个能在List\<Integer>中运行的方法,就可以使用上界通配符实现。

使用如<? extends A>声明。在这种情况下，extends是广义的意味着"extends(类)"或"implements(接口)"。

例如：

```java
public static void process(List<? extends Foo> list) { /* ... */ }

public static void process(List<? extends Foo> list) {
    for (Foo elem : list) {
        // elem可以使用任何定义在Foo类中的方法。
    }
}


public static double sumOfList(List<? extends Number> list) {
    double s = 0.0;
    for (Number n : list)
        s += n.doubleValue();
    return s;
}

List<Double> ld = Arrays.asList(1.2, 2.3, 3.5);
System.out.println("sum = " + sumOfList(ld));//sum=7.0
```

## 无界通配符

无界通配符如:List<?>意为未知类型的列表。有两种场景适用无界通配符:

- 如果你在编写一个能用Object类提供的方法实现的方法
- 当代码使用了泛型类中不依赖于类型参数的方法，例如List.size或List.clear。Class<?>就经常被使用,因为Class\<T\>中的多数方法不依赖T。

考虑下面的printList方法:

```java
public static void printList(List<Object> list) {
    for (Object elem : list)
        System.out.println(elem + " ");
    System.out.println();
}
```

printList并不能打印任何类型的列表,它仅打印Object实例。因为像例如List\<Integer\>不是List\<Object\>的子类型。使用无界通配符可实现打印任意类型列表：

```java
public static void printList(List<?> list) {
    for (Object elem: list)
        System.out.print(elem + " ");
    System.out.println();
}

List<Integer> li = Arrays.asList(1, 2, 3);
List<String>  ls = Arrays.asList("one", "two", "three");
printList(li);
printList(ls);
```

List\<Object\>与List\<?\>是不同的,你可以往List\<Object\>插入Object或任意Object的子类型,但你仅能插入null到List\<?\>。

## 下界通配符

下界通配符将未知类型限定为特定类型或该类型的超类,使用<? super A>声明。下界通配符与上界通配符不能同时使用。

## 通配符和子类型

可以使用通配符创建泛型中的类型关系,尽管List\<Integer\>和List\<Number\>没有关系,但是它们的父类型都是List\<?\>。

```java
List<? extends Integer> intList = new ArrayList<>();
List<? extends Number>  numList = intList;  // OK. List<? extends Integer> is a subtype of List<? extends Number>

```

下图显示了几个List类之间的关系:

![](/images/Java/通配符子类型关系.png)

## 通配符捕获和辅助方法

有时编译器会推断通配符的类型,这种场景叫通配符捕获。

下面的WildcardError类在编译时会产生一个捕获错误:

```java
import java.util.List;

public class WildcardError {

    void foo(List<?> i) {
        i.set(0, i.get(0));
    }
}
//Java7
WildcardError.java:6: error: method set in interface List<E> cannot be applied to given types;
    i.set(0, i.get(0));
     ^
  required: int,CAP#1
  found: int,Object
  reason: actual argument Object cannot be converted to CAP#1 by method invocation conversion
  where E is a type-variable:
    E extends Object declared in interface List
  where CAP#1 is a fresh type-variable:
    CAP#1 extends Object from capture of ?
1 error
```

你可以通过编写一个私有的辅助方法来捕获通配符。

```java
public class WildcardFixed {

    void foo(List<?> i) {
        fooHelper(i);
    }


    // Helper method created so that the wildcard can be captured
    // through type inference.
    //通过该方法,编译器推断T就是被捕获的CAP#1变量的类型
    private <T> void fooHelper(List<T> l) {
        l.set(0, l.get(0));
    }

}
```

约定辅助方法命名为*originalMethodName*Helper（原方法名+Helper）。

再看一个更复杂的例子：

```java
import java.util.List;

public class WildcardErrorBad {

    void swapFirst(List<? extends Number> l1, List<? extends Number> l2) {
      Number temp = l1.get(0);
      l1.set(0, l2.get(0)); // expected a CAP#1 extends Number,
                            // got a CAP#2 extends Number;
                            // same bound, but different types
      l2.set(0, temp);	    // expected a CAP#1 extends Number,
                            // got a Number
    }
}

List<Integer> li = Arrays.asList(1, 2, 3);
List<Double>  ld = Arrays.asList(10.10, 20.20, 30.30);
//这种情况没有辅助方法可以解决,因为把Double值添加到Integer列表中就是语法错误的
swapFirst(li, ld);

//
WildcardErrorBad.java:7: error: method set in interface List<E> cannot be applied to given types;
      l1.set(0, l2.get(0)); // expected a CAP#1 extends Number,
        ^
  required: int,CAP#1
  found: int,Number
  reason: actual argument Number cannot be converted to CAP#1 by method invocation conversion
  where E is a type-variable:
    E extends Object declared in interface List
  where CAP#1 is a fresh type-variable:
    CAP#1 extends Number from capture of ? extends Number
WildcardErrorBad.java:10: error: method set in interface List<E> cannot be applied to given types;
      l2.set(0, temp);      // expected a CAP#1 extends Number,
        ^
  required: int,CAP#1
  found: int,Number
  reason: actual argument Number cannot be converted to CAP#1 by method invocation conversion
  where E is a type-variable:
    E extends Object declared in interface List
  where CAP#1 is a fresh type-variable:
    CAP#1 extends Number from capture of ? extends Number
WildcardErrorBad.java:15: error: method set in interface List<E> cannot be applied to given types;
        i.set(0, i.get(0));
         ^
  required: int,CAP#1
  found: int,Object
  reason: actual argument Object cannot be converted to CAP#1 by method invocation conversion
  where E is a type-variable:
    E extends Object declared in interface List
  where CAP#1 is a fresh type-variable:
    CAP#1 extends Object from capture of ?
3 errors
```

## 使用指南

**输入变量**

输入变量将数据提供给代码。例如copy(src,dest)方法src是输入参数

**输出变量**

输出变量接收在其他地方使用的数据。如copy(src,dest)方法dest是输出参数

当然有些变量既是输入也是输出,可以参考以下几点使用通配符:

- 输入变量使用上界通配符定义(使用extends关键字)
- 输出变量使用下界通配符定义(使用super关键字)
- 可以使用Object类中方法访问输入变量时,使用无界通配符
- 代码需要访问既是输入又是输出的变量时,不要使用通配符

这些指南不适用于方法返回类型。应避免使用通配符作为返回类型。

像List\<? extends ... \>这样的列表可以认为它仅是只读的(并非严格意义上)。

```java
class NaturalNumber {

    private int i;

    public NaturalNumber(int i) { this.i = i; }
    // ...
}

class EvenNumber extends NaturalNumber {

    public EvenNumber(int i) { super(i); }
    // ...
}

List<EvenNumber> le = new ArrayList<>();
List<? extends NaturalNumber> ln = le;
ln.add(new NaturalNumber(35));  // compile-time error
```

因为List\<EvenNumber\>是List\<? extends NaturalNumber\>的子类型,所以可以把le赋值给ln。但是你不能使用ln添加自然数,因为它实际上是偶数列表,下列操作是可以的(从以下几点可以看出List\<? extends NaturalNumber\>并不是严格语意上的只读):

- 添加null
- 调用clear
- 获取迭代器并调用remove
- 捕获通配符并写入列表中读取的元素

# 类型擦除

Java语言引入泛型以在编译期提供严格类型检查。Java编译器应用类型擦除来实现泛型：

- 如果类型参数是无界的，使用它们的边界或Object代替所有的类型参数
- 如有必要执行类型转换
- 生成桥接方法以保证在泛型类型继承中的多态性

类型擦除确保没有为参数化类型创建新类,因此,泛型没有运行时开销。

## 泛型类中的擦除

Java编译器擦除所有类型参数,如果类型参数是有界的,则使用首个边界代替;如果类型参数是无界的,则使用Object代替。

考虑下面的泛型类：

```java
public class Node<T> {

    private T data;
    private Node<T> next;

    public Node(T data, Node<T> next) {
        this.data = data;
        this.next = next;
    }

    public T getData() { return data; }
    // ...
}
```

因为类型参数是无界的,Java编译器使用Object代替:

```java
public class Node {

    private Object data;
    private Node next;

    public Node(Object data, Node next) {
        this.data = data;
        this.next = next;
    }

    public Object getData() { return data; }
    // ...
}
```

当泛型类Node使用有界类型参数时,Java编译器使用首个边界类代替类型参数:

```java
public class Node<T extends Comparable<T>> {

    private T data;
    private Node<T> next;

    public Node(T data, Node<T> next) {
        this.data = data;
        this.next = next;
    }

    public T getData() { return data; }
    // ...
}

//泛型擦除后
public class Node {

    private Comparable data;
    private Node next;

    public Node(Comparable data, Node next) {
        this.data = data;
        this.next = next;
    }

    public Comparable getData() { return data; }
    // ...
}
```

## 泛型方法中的擦除

与泛型类中的擦除类似:

```java
// Counts the number of occurrences of elem in anArray.
//
public static <T> int count(T[] anArray, T elem) {
    int cnt = 0;
    for (T e : anArray)
        if (e.equals(elem))
            ++cnt;
        return cnt;
}
//擦除后
public static int count(Object[] anArray, Object elem) {
    int cnt = 0;
    for (Object e : anArray)
        if (e.equals(elem))
            ++cnt;
        return cnt;
}


```

```java
class Shape { /* ... */ }
class Circle extends Shape { /* ... */ }
class Rectangle extends Shape { /* ... */ }

public static <T extends Shape> void draw(T shape) { /* ... */ }

//擦除后
public static void draw(Shape shape) { /* ... */ }
```

## 类型擦除的影响和桥接方法

以下例子展示了编译器在类型擦除的过程中如何创建桥接方法。

```java
public class Node<T> {

    public T data;

    public Node(T data) { this.data = data; }

    public void setData(T data) {
        System.out.println("Node.setData");
        this.data = data;
    }
}

public class MyNode extends Node<Integer> {
    public MyNode(Integer data) { super(data); }

    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
}

MyNode mn = new MyNode(5);
Node n = mn;            // A raw type - compiler throws an unchecked warning
n.setData("Hello");     // Causes a ClassCastException to be thrown.
Integer x = mn.data;   

//类型擦出后
MyNode mn = new MyNode(5);
Node n = mn;            // A raw type - compiler throws an unchecked warning
                        // Note: This statement could instead be the following:
                        //     Node n = (Node)mn;
                        // However, the compiler doesn't generate a cast because
                        // it isn't required.
n.setData("Hello");     // Causes a ClassCastException to be thrown.
Integer x = (Integer)mn.data; 
```

**桥接方法**

当编译一个继承于参数化类的类或接口,或实现一个参数化接口时,编译器会自动创建一个桥接方法。

类型擦除后，Node和MyNode类：

```java
public class Node {

    public Object data;

    public Node(Object data) { this.data = data; }

    public void setData(Object data) {
        System.out.println("Node.setData");
        this.data = data;
    }
}

public class MyNode extends Node {

    public MyNode(Integer data) { super(data); }

    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
}
```

可以注意到类型擦除后,方法签名不匹配了,Node.setData(T)方法变成了Node.setData(Object)。也就是说MyNode.setData(Integer)没有重写Node.setData(Object)。

为了保证泛型擦出后的多态性，编译器生成了一个桥接方法：

```java
class MyNode extends Node {

    // Bridge method generated by the compiler
    //
    public void setData(Object data) {
        setData((Integer) data);
    }

    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }

    // ...
}
```

桥接方法MyNode.setData(object)委托给了原始方法MyNode.setData(Integer)。所以n.setData("Hello");语句调用的是 `MyNode.setData(Object)`方法，所以才会抛出ClassCastException（因为"Hello"不能转换成Integer）。

## 非具体类型（Non-Reifiable Types）

具体类型指在类型信息在运行时可知道类型,包括:

- 原始类型
- 非泛型类型
- raw type
- 无界通配符调用

非具体类型指类型信息在编译器被擦除的类型:调用不是由无界通配符定义的泛型类型。其类型信息在运行时不完全可知,如List\<String\>和List\<Number\>，JVM在运行时无法分辨这两者点区别。

在某些场景下非具体类型是不能使用的，如`instanceof`语句或作为数组元素。

**堆污染**

当一个参数化类型的变量指向一个不是这些参数化类型的对象时，就会发生堆污染。如果该程序执行某些操作会引起编译时的未受检警告，则会发生这种情况。当包含一个参数化类型的操作(如转换或方法调用)在编译时或运行时无法被验证时,就会生成未受检警告。例如，混用原始类型（raw type）和参数化类型时,或执行未受检转换时就会出现堆污染。

**含非具体类型形参的可变参方法的漏洞**

包含可变长入参的泛型方法可能造成堆污染：

```java
public class ArrayBuilder {

  public static <T> void addToList (List<T> listArg, T... elements) {
    for (T x : elements) {
      listArg.add(x);
    }
  }

  public static void faultyMethod(List<String>... l) {
    Object[] objectArray = l;     // Valid但是可能产生堆污染
    objectArray[0] = Arrays.asList(42);
    String s = l[0].get(0);       // ClassCastException thrown here
  }

}
```

```java
public class HeapPollutionExample {

  public static void main(String[] args) {

    List<String> stringListA = new ArrayList<String>();
    List<String> stringListB = new ArrayList<String>();

    ArrayBuilder.addToList(stringListA, "Seven", "Eight", "Nine");
    ArrayBuilder.addToList(stringListB, "Ten", "Eleven", "Twelve");
    List<List<String>> listOfStringLists =
      new ArrayList<List<String>>();
    ArrayBuilder.addToList(listOfStringLists,
      stringListA, stringListB);

    ArrayBuilder.faultyMethod(Arrays.asList("Hello!"), Arrays.asList("World!"));
  }
}
//warning: [varargs] Possible heap pollution from parameterized vararg type T
```

当编译器进入一个可变长参数方法时，会将其转换成数组。但是Java语言不允许创建参数化类型的数组。Java编译器会将T...elements转换成Object[] elements，这可能产生堆污染。

**阻止带不可变类型的可变长参数方法的警告**

- @SafeVarargs
- @SuppressWarnings({"unchecked","varargs"}):注意这无法抑制从方法调用处产生的警告

# 泛型约束

## 不能使用原始类型实例化泛型类型

```java
class Pair<K, V> {

    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    // ...
}

Pair<int, char> p = new Pair<>(8, 'a');  // compile-time error
```

## 不能创建类型参数的实例

```java
public static <E> void append(List<E> list) {
    E elem = new E();  // compile-time error
    list.add(elem);
}
```

但是你可以通过反射创建类型参数的对象

```java
public static <E> void append(List<E> list, Class<E> cls) throws Exception {
    E elem = cls.newInstance();   // OK
    list.add(elem);
}
//调用
List<String> ls = new ArrayList<>();
append(ls, String.class);
```

## 不能声明为静态属性

## 不能转换或使用instanceof语句

```java
public static <E> void rtti(List<E> list) {
    if (list instanceof ArrayList<Integer>) {  // compile-time error
        // ...
    }
}

List<Integer> li = new ArrayList<>();
List<Number>  ln = (List<Number>) li;  // compile-time error
```

有时候转换是允许的：

```java
List<String> l1 = ...;
ArrayList<String> l2 = (ArrayList<String>)l1;  // OK
```

另外，无界通配符可以使用instanceof：

```java
public static void rtti(List<?> list) {
    if (list instanceof ArrayList<?>) {  // OK; instanceof requires a reifiable type
        // ...
    }
}
```

## 不能创建参数化类型的数组

```java
List<Integer>[] arrayOfLists = new List<Integer>[2];  // compile-time error
//如果不同的类型插入到同一个数组将会产生异常
Object[] strings = new String[2];
strings[0] = "hi";   // OK
strings[1] = 100;    // An ArrayStoreException is thrown.
//所以类似的，不允许这样创建参数化类型的数组
Object[] stringLists = new List<String>[2];  // compiler error, but pretend it's allowed
stringLists[0] = new ArrayList<String>();   // OK
stringLists[1] = new ArrayList<Integer>();  // An ArrayStoreException should be thrown,
                                            // but the runtime can't detect it.
```

## 不能创建，捕获或抛出参数化类型的对象

泛型类不能直接或间接地继承Throwable类。

```java
// Extends Throwable indirectly
class MathException<T> extends Exception { /* ... */ }    // compile-time error

// Extends Throwable directly
class QueueFullException<T> extends Throwable { /* ... */ // compile-time error
    
public static <T extends Exception, J> void execute(List<J> jobs) {
    try {
        for (J job : jobs)
            // ...
    } catch (T e) {   // compile-time error
        // ...
    }
}    
```

但是可以在throws语句中使用类型参数：

```java
class Parser<T extends Exception> {
    public void parse(File file) throws T {     // OK
        // ...
    }
}
```

## 参数在类型擦除后一眼的方法不能重载

```java
public class Example {
    //'print(Set<String>)' clashes with 'print(Set<Integer>)'; both methods have same erasure
    public void print(Set<String> strSet) { }
    public void print(Set<Integer> intSet) { }
}
```

