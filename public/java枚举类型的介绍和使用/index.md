# Java枚举类型介绍和使用

### 1. 枚举类型的介绍

枚举类型是一种特殊的数据类型，它使得变量成为一组预定义常量。所以在需要表示一组固定常量时应尽量使用枚举类型。通过关键字enum来定义枚举类，它和普通类一样可以有构造器、成员变量、方法。

####  1.1 枚举类的特性

- 所有的枚举类都隐式的继承`java.lang.Enum`，Java不允许多继承，所以枚举类不能再继承其他任何类，但可以实现接口
- 枚举类被隐式地声明为final，所以也不能被其他任何类继承
- 枚举类型的构造函数修饰符必须是private。定义枚举常量时会自动调用，不能自己调用枚举的构造函数
- 枚举类的实例必须在第一行列出，并且枚举值默认被public static final修饰
- 编译时编译器会自动帮我们添加两个静态方法`values()`和`valueOf()`

#### 1.2 枚举类的原理

下面我们定义了一个枚举类并让它实现Info接口，这样可以让枚举值提供不同的实现，当然也可以在枚举类里面定义一个抽象方法，这样枚举值也必须实现此抽象方法才可，效果都一样。

```Java
public enum Season implements Info{
    SPRING("spring","春暖花开"){
        public void show(){
            System.out.println("春天在哪里");
        }
    },
    SUMMER("summer","夏日炎炎"){
        public void show(){
            System.out.println("生如夏花");
        }
    },
    AUTUMN("autumn","秋高气爽"){
        public void show(){
            System.out.println("秋");
        }
    },
    WINTER("winter","白雪茫茫"){
        public void show(){
            System.out.println("冷");
        }
    };
    private final String seasonName;
    private final String seasonDesc;


    Season(String seasonName,String seasonDesc){
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }

    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }

    @Override
    public String toString() {
        return "Season{" +
                "seasonName='" + seasonName + '\'' +
                ", seasonDesc='" + seasonDesc + '\'' +
                '}';
    }

}

```

可以看到该枚举类编译后的class文件，其中还包括了四个枚举值对应的class文件，而且其后还带有序号。在枚举实例创建时会给每个枚举值指定一个整形常量值（序号），若没有显示指定，则 整形常量值从0开始递增。这其实是与父类Enum有关，后面会介绍。

{{< image src="https://cdn.heysen.xyz/201907222345_571.png" >}}


下面是利用javap工具查看Season.java经过编译后的字节码（请忽略乱码--.--）:

```Java
"C:\Program Files\Java\jdk1.8.0_201\bin\javap.exe" -c pre.chl.enums.Season
Compiled from "Season.java"
public abstract class pre.chl.enums.Season extends java.lang.Enum<pre.chl.enums.Season> implements pre.chl.enums.Info {
  public static final pre.chl.enums.Season SPRING;

  public static final pre.chl.enums.Season SUMMER;

  public static final pre.chl.enums.Season AUTUMN;

  public static final pre.chl.enums.Season WINTER;

  public static pre.chl.enums.Season[] values();
    Code:
       0: getstatic     #2                  // Field $VALUES:[Lpre/chl/enums/Season;
       3: invokevirtual #3                  // Method "[Lpre/chl/enums/Season;".clone:()Ljava/lang/Object;
       6: checkcast     #4                  // class "[Lpre/chl/enums/Season;"
       9: areturn

  public static pre.chl.enums.Season valueOf(java.lang.String);
    Code:
       0: ldc           #5                  // class pre/chl/enums/Season
       2: aload_0
       3: invokestatic  #6                  // Method java/lang/Enum.valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
       6: checkcast     #5                  // class pre/chl/enums/Season
       9: areturn

  public java.lang.String getSeasonName();
    Code:
       0: aload_0
       1: getfield      #8                  // Field seasonName:Ljava/lang/String;
       4: areturn

  public java.lang.String getSeasonDesc();
    Code:
       0: aload_0
       1: getfield      #9                  // Field seasonDesc:Ljava/lang/String;
       4: areturn

  public java.lang.String toString();
    Code:
       0: new           #10                 // class java/lang/StringBuilder
       3: dup
       4: invokespecial #11                 // Method java/lang/StringBuilder."<init>":()V
       7: ldc           #12                 // String Season{seasonName='
       9: invokevirtual #13                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      12: aload_0
      13: getfield      #8                  // Field seasonName:Ljava/lang/String;
      16: invokevirtual #13                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      19: bipush        39
      21: invokevirtual #14                 // Method java/lang/StringBuilder.append:(C)Ljava/lang/StringBuilder;
      24: ldc           #15                 // String , seasonDesc='
      26: invokevirtual #13                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      29: aload_0
      30: getfield      #9                  // Field seasonDesc:Ljava/lang/String;
      33: invokevirtual #13                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      36: bipush        39
      38: invokevirtual #14                 // Method java/lang/StringBuilder.append:(C)Ljava/lang/StringBuilder;
      41: bipush        125
      43: invokevirtual #14                 // Method java/lang/StringBuilder.append:(C)Ljava/lang/StringBuilder;
      46: invokevirtual #16                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      49: areturn

  pre.chl.enums.Season(java.lang.String, int, java.lang.String, java.lang.String, pre.chl.enums.Season$1);
    Code:
       0: aload_0
       1: aload_1
       2: iload_2
       3: aload_3
       4: aload         4
       6: invokespecial #1                  // Method "<init>":(Ljava/lang/String;ILjava/lang/String;Ljava/lang/String;)V
       9: return

  static {};
    Code:
       0: new           #17                 // class pre/chl/enums/Season$1
       3: dup
       4: ldc           #18                 // String SPRING
       6: iconst_0
       7: ldc           #19                 // String spring
       9: ldc           #20                 // String ��ů����
      11: invokespecial #21                 // Method pre/chl/enums/Season$1."<init>":(Ljava/lang/String;ILjava/lang/String;Ljava/lang/String;)V
      14: putstatic     #22                 // Field SPRING:Lpre/chl/enums/Season;
      17: new           #23                 // class pre/chl/enums/Season$2
      20: dup
      21: ldc           #24                 // String SUMMER
      23: iconst_1
      24: ldc           #25                 // String summer
      26: ldc           #26                 // String ��������
      28: invokespecial #27                 // Method pre/chl/enums/Season$2."<init>":(Ljava/lang/String;ILjava/lang/String;Ljava/lang/String;)V
      31: putstatic     #28                 // Field SUMMER:Lpre/chl/enums/Season;
      34: new           #29                 // class pre/chl/enums/Season$3
      37: dup
      38: ldc           #30                 // String AUTUMN
      40: iconst_2
      41: ldc           #31                 // String autumn
      43: ldc           #32                 // String �����ˬ
      45: invokespecial #33                 // Method pre/chl/enums/Season$3."<init>":(Ljava/lang/String;ILjava/lang/String;Ljava/lang/String;)V
      48: putstatic     #34                 // Field AUTUMN:Lpre/chl/enums/Season;
      51: new           #35                 // class pre/chl/enums/Season$4
      54: dup
      55: ldc           #36                 // String WINTER
      57: iconst_3
      58: ldc           #37                 // String winter
      60: ldc           #38                 // String ��ѩãã
      62: invokespecial #39                 // Method pre/chl/enums/Season$4."<init>":(Ljava/lang/String;ILjava/lang/String;Ljava/lang/String;)V
      65: putstatic     #40                 // Field WINTER:Lpre/chl/enums/Season;
      68: iconst_4
      69: anewarray     #5                  // class pre/chl/enums/Season
      72: dup
      73: iconst_0
      74: getstatic     #22                 // Field SPRING:Lpre/chl/enums/Season;
      77: aastore
      78: dup
      79: iconst_1
      80: getstatic     #28                 // Field SUMMER:Lpre/chl/enums/Season;
      83: aastore
      84: dup
      85: iconst_2
      86: getstatic     #34                 // Field AUTUMN:Lpre/chl/enums/Season;
      89: aastore
      90: dup
      91: iconst_3
      92: getstatic     #40                 // Field WINTER:Lpre/chl/enums/Season;
      95: aastore
      96: putstatic     #2                  // Field $VALUES:[Lpre/chl/enums/Season;
      99: return
}

Process finished with exit code 0

```

可以发现，一个枚举类在经过编译后变成了一个抽象类，同时添加了两个方法`values()`和`valueOf(String)`。这里还可以看到字节码中的构造函数多了两个参数，一个int型一个String型，这两个参数其实是Enum里面的。

#### 1.3 java.lang.Enum

下面是JDK1.8的实现，注释做了部分翻译。

```Java

package java.lang;

import java.io.Serializable;
import java.io.IOException;
import java.io.InvalidObjectException;
import java.io.ObjectInputStream;
import java.io.ObjectStreamException;

/**
 * 这是所有Java语言枚举类型的公共基类。
 *
 * 有关枚举的更多信息，包括对枚举的描述、隐式声明由编译器合成的方法，
 * 可以见于<cite> Java＆trade;Language</ cite>的第8、9节。
 * 另外，使用枚举类型作为set的类型或作为Map中keys的类型时，java.util.EnumSet
 * 和java.util.EnumMap更加高效
 *
 * @param <E> The enum type subclass
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @see     Class#getEnumConstants()
 * @see     java.util.EnumSet
 * @see     java.util.EnumMap
 * @since   1.5
 */
public abstract class Enum<E extends Enum<E>>
        implements Comparable<E>, Serializable {
    /**
     * 枚举常量的名称，和枚举声明中声明的一致。
     * 大多数程序员应该使用{@link #toString}方法而不是直接访问此字段。
     */
    private final String name;

    /**
     * Returns the name of this enum constant, exactly as declared in its
     * enum declaration.
     *
     * <b>Most programmers should use the {@link #toString} method in
     * preference to this one, as the toString method may return
     * a more user-friendly name.</b>  This method is designed primarily for
     * use in specialized situations where correctness depends on getting the
     * exact name, which will not vary from release to release.
     *
     * @return the name of this enum constant
     */
    public final String name() {
        return name;
    }

    /**
     * 这个枚举常量的序数（它在枚举声明中的位置，分配给初始常量的序数为零）。
     * 大多数程序员对此字段没有使用。它是专门设计的供复杂的基于枚举的数据结构使用	 * 的，例如{@link java.util.EnumSet}和{@link java.util.EnumMap}.
     */
    private final int ordinal;

    /**
     * Returns the ordinal of this enumeration constant (its position
     * in its enum declaration, where the initial constant is assigned
     * an ordinal of zero).
     *
     * Most programmers will have no use for this method.  It is
     * designed for use by sophisticated enum-based data structures, such
     * as {@link java.util.EnumSet} and {@link java.util.EnumMap}.
     *
     * @return the ordinal of this enumeration constant
     */
    public final int ordinal() {
        return ordinal;
    }

    /**
     * 唯一的构造函数，程序员无法调用次构造函数。它由编译器在进行枚举类型声明时调用。
     *
     * @param name - The name of this enum constant, which is the identifier
     *               used to declare it.
     * @param ordinal - The ordinal of this enumeration constant (its position
     *         in the enum declaration, where the initial constant is assigned
     *         an ordinal of zero).
     */
    protected Enum(String name, int ordinal) {
        this.name = name;
        this.ordinal = ordinal;
    }

    /**
     * Returns the name of this enum constant, as contained in the
     * declaration.  This method may be overridden, though it typically
     * isn't necessary or desirable.  An enum type should override this
     * method when a more "programmer-friendly" string form exists.
     *
     * @return the name of this enum constant
     */
    public String toString() {
        return name;
    }

    /**
     * Returns true if the specified object is equal to this
     * enum constant.
     *
     * @param other the object to be compared for equality with this object.
     * @return  true if the specified object is equal to this
     *          enum constant.
     */
    public final boolean equals(Object other) {
        return this==other;
    }

    /**
     * Returns a hash code for this enum constant.
     *
     * @return a hash code for this enum constant.
     */
    public final int hashCode() {
        return super.hashCode();
    }

    /**
     * 抛出CloneNotSupportedException。这保证了枚举永远不会被克隆，这保证它们的“单例状态”。
     *
     * @return (never returns)
     */
    protected final Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
    }

    /**
     * Compares this enum with the specified object for order.  Returns a
     * negative integer, zero, or a positive integer as this object is less
     * than, equal to, or greater than the specified object.
     *
     * Enum constants are only comparable to other enum constants of the
     * same enum type.  The natural order implemented by this
     * method is the order in which the constants are declared.
     */
    public final int compareTo(E o) {
        Enum<?> other = (Enum<?>)o;
        Enum<E> self = this;
        if (self.getClass() != other.getClass() && // optimization
            self.getDeclaringClass() != other.getDeclaringClass())
            throw new ClassCastException();
        return self.ordinal - other.ordinal;
    }

    /**
     * Returns the Class object corresponding to this enum constant's
     * enum type.  Two enum constants e1 and  e2 are of the
     * same enum type if and only if
     *   e1.getDeclaringClass() == e2.getDeclaringClass().
     * (The value returned by this method may differ from the one returned
     * by the {@link Object#getClass} method for enum constants with
     * constant-specific class bodies.)
     *
     * @return the Class object corresponding to this enum constant's
     *     enum type
     */
    @SuppressWarnings("unchecked")
    public final Class<E> getDeclaringClass() {
        Class<?> clazz = getClass();
        Class<?> zuper = clazz.getSuperclass();
        return (zuper == Enum.class) ? (Class<E>)clazz : (Class<E>)zuper;
    }

    /**
     * 返回指定枚举类型和名称的枚举常量，名称必须与声明的完全匹配。
     * 无关的空白字符是不被允许的。
     * <p>Note that for a particular enum type {@code T}, the
     * implicitly declared {@code public static T valueOf(String)}
     * method on that enum may be used instead of this method to map
     * from a name to the corresponding enum constant. 所有的枚举常量值可以通过调用隐式方法{@code public static T[] values()}获得。
     *
     * @param <T> The enum type whose constant is to be returned
     * @param enumType the {@code Class} object of the enum type from which
     *      to return a constant
     * @param name the name of the constant to return
     * @return the enum constant of the specified enum type with the
     *      specified name
     * @throws IllegalArgumentException if the specified enum type has
     *         no constant with the specified name, or the specified
     *         class object does not represent an enum type
     * @throws NullPointerException if {@code enumType} or {@code name}
     *         is null
     * @since 1.5
     */
    public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                                String name) {
        T result = enumType.enumConstantDirectory().get(name);
        if (result != null)
            return result;
        if (name == null)
            throw new NullPointerException("Name is null");
        throw new IllegalArgumentException(
            "No enum constant " + enumType.getCanonicalName() + "." + name);
    }

    /**
     * enum classes cannot have finalize methods.
     */
    protected final void finalize() { }

    /**
     * 防止默认反序列化。
     */
    private void readObject(ObjectInputStream in) throws IOException,
        ClassNotFoundException {
        throw new InvalidObjectException("can't deserialize enum");
    }

    private void readObjectNoData() throws ObjectStreamException {
        throw new InvalidObjectException("can't deserialize enum");
    }
}

```

引用API文档对Enum中一些方法（不包含继承Object的）的解释:

| protected Object           |          clone()抛出CloneNotSupportedException异常           |
| :------------------------- | :----------------------------------------------------------: |
| int                        |       compareTo(E o)将此枚举与指定的对象比较以进行排序       |
| boolean                    | equals(Object other)如果指定的对象和此枚举常量相等就返回true |
| protected void             |              finalize()枚举类不能有finalize方法              |
| Class<E>                   | getDeclaringClass()返回和此枚举常量的枚举类型相对应的Class对象 |
| int                        |               hashCode()返回此枚举常量的哈希值               |
| String                     |     name()返回此枚举常量的名称,与其枚举声明中声明的一致      |
| int                        | ordinal()返回此枚举常量的序号(它在枚举声明中的位置,初始常量的序号默认为0) |
| String                     |           toString()返回声明中包含的枚举常量的名称           |
| static <T extends Enum<T>> | valueOf(Class<T> enumType,String name)返回指定名称和类型的枚举常量 |

这里有一点需要注意的是，我们不能在定义的枚举类型的构造器里调用父类Enum的构造器，这是不被允许的，在编译器编译时会自动帮我们调用去初始化枚举值的声明。虽然Enum是一个抽象类，但是自定义枚举类型仍然会在编译阶段使用Enum的构造函数，不过在创建子类对象时到底有没有创建父类对象还存在分歧，具体可以看看知乎上的这篇[文章](https://www.zhihu.com/question/51920553/answer/128610039?utm_source=qq&utm_medium=social)。



### 2. 枚举类的使用

#### 2.1 定义

```java
public enum Day {
    SUNDAY, MONDAY, TUESDAY, WEDNESDAY,
    THURSDAY, FRIDAY, SATURDAY 
}
```

枚举值之间使用","分隔，如果后面没有属性或方法，最后加不加";"都行。但如果有就一定要加";"

```java
public enum PurOrderSplitEnum {
    NORELEASEERPBILLCOUNT("1", "待发布"),
    RELEASEERPBILLCOUNT("2", "待买方确认"),
    PURCONFIRMINGCOUNT("3", "买方变更中");
    
    private String code;
    private String name;
    public static Map<String, String> code2name = new HashMap();
    public static Map<String, String> code2Muname = new HashMap();

    private PurOrderSplitEnum(String code, String name) {
        this.code = code;
        this.name = name;
    }

    public String getCode() {
        return this.code;
    }

    public String getName() {
        return this.name;
    }

    static {
        code2name.put(NORELEASEERPBILLCOUNT.getCode(), NORELEASEERPBILLCOUNT.getName());
        code2name.put(RELEASEERPBILLCOUNT.getCode(), RELEASEERPBILLCOUNT.getName());
        code2name.put(PURCONFIRMINGCOUNT.getCode(), PURCONFIRMINGCOUNT.getName());
   
        code2Muname.put(NORELEASEERPBILLCOUNT.getCode(), "NORELEASEERPBILLCOUNT");
        code2Muname.put(RELEASEERPBILLCOUNT.getCode(), "RELEASEERPBILLCOUNT");
        code2Muname.put(PURCONFIRMINGCOUNT.getCode(), "PURCONFIRMINGCOUNT");
     
    }
}

```

#### 2.2 结合Switch

在switch中使用枚举，可以让我们的代码可读性更好

```java
public class EnumTest {
    Day day;
    
    public EnumTest(Day day) {
        this.day = day;
    }
    
    public void tellItLikeItIs() {
        switch (day) {
            case MONDAY:
                System.out.println("周一还行吧");
                break;
                    
            case FRIDAY:
                System.out.println("周五很nice");
                break;
                         
            case SATURDAY: 
            case SUNDAY:
                System.out.println("周末超级棒");
                break;
                        
            default:
                System.out.println("额....");
                break;
        }
    }
    
    public static void main(String[] args) {
        EnumTest firstDay = new EnumTest(Day.MONDAY);
        firstDay.tellItLikeItIs();
        EnumTest thirdDay = new EnumTest(Day.WEDNESDAY);
        thirdDay.tellItLikeItIs();
        EnumTest fifthDay = new EnumTest(Day.FRIDAY);
        fifthDay.tellItLikeItIs();
        EnumTest sixthDay = new EnumTest(Day.SATURDAY);
        sixthDay.tellItLikeItIs();
        EnumTest seventhDay = new EnumTest(Day.SUNDAY);
        seventhDay.tellItLikeItIs();
    }
    //输出结果
    /*
    周一还行吧
	额....
	周五很nice
	周末超级棒
	周末超级棒
	*/
```

#### 2.3 valueOf方法

把字符串转成对应类型的枚举值，如：

```Java
 String str = "SPRING";
 Season season2 = Season.valueOf(str);
 //Season season2 = Season.valueOf(Season.class,str);跟上面是等价的
 System.out.println(season2 == season1);//true
```

#### 2.4 values()和其他方法

values()方法可以返回所有定义的枚举值，name()方法返回的是枚举的名称，不是枚举值里面的属性名称，这里需要注意一下。

```Java
 Season season1 = Season.SPRING;
 for(Season season:Season.values()){
     System.out.println(season);
 }
 season1.show();
 System.out.println(season1.name());
 System.out.println("SPIRNG的序号:" + season1.ordinal());
//输出结果
/*
Season{seasonName='spring', seasonDesc='春暖花开'}
Season{seasonName='summer', seasonDesc='夏日炎炎'}
Season{seasonName='autumn', seasonDesc='秋高气爽'}
Season{seasonName='winter', seasonDesc='白雪茫茫'}
春天在哪里
SPRING
0
*/
```

#### 2.5 使用接口组织枚举

在接口中定义枚举类，可以将数据分组。

```Java
public interface Animal {  
    enum Dog implements Animal {  
         HUSKY,GOLDEN_RETRIEVER,ALASKAN_MALAMUTE
    }  
       
    enum Cat implements Animal {  
         PERSIAN,RAGDOLL,BIRMAN  
    }  
}
// Animal dog = Animal.Dog.ALASKAN_MALAMUTE;
// Animal cat = Animal.Cat.BIRMAN;
```

#### 2.6 EnumSet和EnumMap

Set类型是枚举集或Map的key是枚举值时，使用这两者将会更加高效。这里只摘取Java API文档的部分说明，详细的方法大家可以自己查找文档。

EnumMap

> 用于枚举类型键的专用Map实现。 枚举映射中的所有键必须来自创建映射时显式或隐式指定的单个枚举类型。 枚举映射在内部表示为数组。 这种表现非常紧凑和高效。
>
> 枚举映射按其键的自然顺序（枚举常量的声明顺序）维护。 这反映在集合视图（keySet（），entrySet（）和values（））返回的迭代器中。
>
> 集合视图返回的迭代器非常一致：它们永远不会抛出ConcurrentModificationException，它们可能会也可能不会显示迭代进行过程中对映射所做的任何修改的影响。
>
> Key不允许为null。 尝试插入null将抛出NullPointerException。 但是，尝试测试是否存在空键或删除空键将正常运行。 value允许为null。
>
> 与大多数集合实现一样，EnumMap不同步。 如果多个线程同时访问枚举映射，并且至少有一个线程修改了映射，则应该在外部进行同步。 这通常通过在自然封装枚举映射的某个对象上同步来完成。 如果不存在此类对象，则应使用Collections.synchronizedMap（java.util.Map <K，V>）方法“包装”映射。 这最好在创建时完成，以防止意外的不同步访问：
>
>    `Map <EnumKey，V> m= Collections.synchronizedMap（new EnumMap <EnumKey，V>（...））;`
>
> 说明：所有基本操作都在恒定时间内执行。 它们很可能（虽然不能保证）比它们对应的HashMap更快。

EnumSet

> 用于枚举类型的专用Set实现。 枚举集中的所有元素必须来自单个枚举类型，该类型在创建集时显式或隐式指定。 枚举集在内部表示为位向量。 这种表现非常紧凑和高效。 这个类的空间和时间性能应该足够好，以允许它作为传统的基于int的“位标志”的高性能、类型安全的替代品。 如果它们的参数也是枚举集，即使批量操作（例如containsAll和retainAll）也应该非常快速地运行。
>
> 迭代器方法返回的迭代器以其自然顺序（枚举常量声明的顺序）遍历元素。 返回的迭代器是弱一致的：它永远不会抛出ConcurrentModificationException，它可能会也可能不会显示迭代进行过程中对集合所做的任何修改的影响。
>
> 不允许使用空元素。 尝试插入null元素将抛出NullPointerException。 但是，尝试测试null元素的存在或删除一个元素将正常工作。
>
> 与大多数集合实现一样，EnumSet不同步。 如果多个线程同时访问枚举集，并且至少有一个线程修改了该集，则应该在外部进行同步。 这通常通过在自然封装枚举集的某个对象上进行同步来实现。 如果不存在此类对象，则应使用`Collections.synchronizedSet（java.util.Set <T>）`方法“包装”该集合。 这最好在创建时完成，以防止意外的不同步访问：
>
> `Set <MyEnum> s = Collections.synchronizedSet（EnumSet.noneOf（MyEnum.class））;`
>
> 说明：所有基本操作都在恒定时间内执行。 它们很可能（虽然不能保证）比它们对应的的HashSet快得多。 如果它们的参数也是枚举集，即使批量操作也会在恒定时间内执行。

#### 2.7 使用枚举实现单例模式

基于枚举类型的特性，Effective Java建议使用枚举来实现单例模式。

```Java
public enum Singleton{
    INSTANCE;
    public void doSomething{
        //...
    }
}
```
