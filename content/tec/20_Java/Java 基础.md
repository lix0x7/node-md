# 参考


- Java Core I & II



# 第三章


- java的元类（metaclass）模型
- java中int永远为32位数
- 数据类型
   - 8中基本数据类型:
   - 4整形: int(4), short(2), long(8), byte(1)
   - 2浮点类型: double(8), float(4)
   - 1字符类型: char(2)
   - 1真值类型: boolean
   - Double
      - Double类的三个常量
         - Double.POSITIVE_INFINITY
         - Double.NEGATIVE_INFINITY
         - Double.NaN
   - Double的精度问题
      - 2.0-1.1=0.8999999999999999
      - 如需无误差的计算，使用BigDecimal类
- import static java.lang.Math.*
静态导入，导入静态方法或静态变量，可以将Math.pow(x, a)简化为pow(x, a)
- java运算符优先级
- String为不可变字符串，只可改变引用而不能更改字符串中某一特定值，如果需要频繁更改，使用StringBuilder
StringBuilder的使用场景：频繁的连接短字符串来构建长字符串，使用StringBuilder，最后使用toString()方法返回字符串，避免每次连接字符串都生成新的String对象。
- 使用*.equals(Object)判断对象是否相等，==运算符判断的是对象是否是内存中的同一个对象
- break语句可以带标签，但只能跳出语句块，而不能跳入语句块
- 数组
声明方式可以使用int[] a或者int a[]，初始化操作中new int[n]，此处n既可以为常量也可以为变量



# 面向对象


- 类间关系
   - 依赖（use-a）：A依赖B则称A/B有耦合关系的。
   - 聚合（has-a）：例如汽车和零件的关系
   - 继承（is-a）
- 类、类文件、类构造方法同名，一个源文件只能包含一个**公有**类。
- jvm会将某些简洁、经常被调用的、未重载、可优化的方法设置为内联方法，以提高运行效率。内联方法在java中不能手动设置
- 不要编写引用可变对象的访问器方法，例如对某一getter方法返回了一个Date对象，但是Date对象可以被直接修改而不需要经过setter方法。解决该问题的方法是在getter中返回要返回对象的副本，即使用clone()
- 静态方法不能访问实例域，但可以访问自身类中的静态域
- 每一个类都可以用有一个static main方法，可以通过该特性来对某个类单独进行测试。
例如有Employee类和Company类，此时想单独测试Employee类中的某些方法，只需直接运行java Employee即可
- java中方法参数采用按值调用
   - 对于基本数据类型，传递数据值；对于对象，传递该对象的引用值的clone
   - [!]注意，此处传递的是引用的克隆，可以理解为复制了一个指针，但该指针和原指针都指向了同一个对象。因而无法修改参数中指针的指向，但是可以通过该指针修改其所指对象的一些属性
   - 一个助于理解的例子
有Employee对象x与y，如下swap方法并不能交换这两个对象
```java
swap(Employee x, Employee y){
	Employee tmp = x;
	x = y;
	y = tmp;
}
```

- 在包中定义类是编译器的任务，字节码始终以完整的包名引用其他类。
- 初始化块
   - 初始化块在构造函数之前执行。
   - 除了一般初始化块，还有静态初始化块，静态初始化块只能访问静态域，且只在类首次加载时执行一次，在构造函数之前执行。
   - 初始化块一般最常用于内部匿名类的初始化等操作。
   - 注意执行顺序，静态初始化块>初始化块>构造函数，示例如下：
```java
public class InitBlock {
	// constructor
	public InitBlock(){
		System.out.println("Constructor");
	}

	// init block
	{
		System.out.println("Init Block");
	}

	// static init block
	static{
		System.out.println("Static Init Block");
	}
}

// output
// Static Init Block	Init Block	Constructor
```

- finalize()
在对象即将被GC回收时调用，极其不建议使用，因为Java中对象何时被回收并不确定。
- 如果想将类放入包中，就必须将包的名字放在源文件的开头。例如：
```java
package com.buaa.lix
public class Employee{
	// …
}
```

   - 如果没有在源文件中放置package语句，这个源文件就被放置在默认包（default package）中。
- public/private/protected/无
   - public范围为所有
   - private范围为定义他们的类（子类都不可以）
   - protected范围为本包和所有子类
   - 如无该修饰符，则范围为同一个包中
- javadoc
   - 在要描述的方法前使用`/** */`包含一段自由格式文本，可以使用HTML修饰符（但不要使用`<h1>`这类，会与生成的文档格式冲突）
   - 如果需要使用图片等，则将资源文件放置在子目录doc-files中，然后使用形如`<img src="doc-files/uml.png" alt="UML diagram">`的标签进行引用
   - 类注释语句放在import语句之后，类定义之前
   - 方法注释，必须放在要描述方法之前
      - [@param ]() 
一个方法的所有@param必须连续，可以换行
      - [@return ]()
      - [@throws ]()
   - 包注释
在包内添加单独的package.html或package-info.java文件
   - 通用注释
@author，@version，@since，[@deprecated ]()
- 类设计技巧
   - 保证数据私有
   - 保证成员变量显式初始化
   - 不要在类中使用过多的基本类型
例如，使用Address类替换street, city, state, zipCode四个String成员变量
   - 将职责过多的类进行分解



# 继承


- 一个对象变量可以指示多种实际类型的现象称为多态（polymorphism），在运行时能够自动选择调用哪个方法的现象称为动态绑定（dynamic binding）
- 一个方法拥有不同的参数列表的现象称为重载（overload）
- 方法名和方法的参数列表构成了方法的签名
- 如果使用了超类的构造方法，那么super超类构造方法必须在子类构造方法中的第一行
- 阻止继承 final
修饰类、方法、成员变量（域）
- 使用instanceof方法判断实例是否是某种特定类型，例如`staff instanceof Manager`
- Object类
   - Object.getClass()返回对象所属的类
   - `hashCode()`/`equals()`

这两个方法一般用于容器类调用，例如HashSet/Hashmap。
HashSet的内部实现实际是Map<E, Object>，hashCode()在这其中起到的作用是：Map根据hashcode计算元素E应该存储的位置（区域，同一区域可以存储多个元素），再根据equals判断同位置元素是否为相同元素，如若不同，则存入该区域；如相同，则不存储，并返回false。
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100544612-e59d4a1d-55e6-49f1-b37a-395d3b33012d.png#align=left&display=inline&height=158&margin=%5Bobject%20Object%5D&originHeight=158&originWidth=551&size=0&status=done&style=none&width=551)
所以hashCode()和equals必须同时重写才能保证程序运行的正确性。其逻辑如下：

   - hashCode不同，equals一定返回false
   - hashCode相同，equals可以返回false
   - equals返回true，则hashCode一定相同
   - equals返回false，则hashCode可以相同也可以不同
      - 但对于equals返回false的两个对象，不同的hashcode可以保证更好的程序性能
   - 默认的Object.hashCode()方法返回对象存储地址；String.hashCode()是由字符串内容导出的，所以相同内容的字符串具有相同的hashCode
- 自动拆箱、自动装箱
   - 基本类型及其包装类的转换是由编译器进行的自动装箱(autoboxing)/自动拆箱实现的。此过程没有虚拟机参加，编译器在字节码中就已经加入了拆箱装箱的方法，具体如下：
   - 自动装箱
```java
ArrayList<Integer> l = new ArrayList<>();
l.add(3);
```

   - 
其中add方法将由编译器变换为
```java
l.add(Integer.valueOf(3));
```

   - 自动拆箱
```java
Integer i01 = 4;
int i02 = i01;
```

   - 
其中i02赋值将由编译器变换为
```java
int i02 = i01.intValue();
```

- Integer、Short、Byte、Character、Long这几个类的valueOf方法的实现是类似的，有限可列举，共享`[-128,127]`；Boolean的valueOf同样只有两个值，有限可列举，共享。
此处详情参见`Integer.valueof(int i)`的源码，可以看到在[-128, 127]范围内的值，是有IntegerCache返回的一个已经缓存的对象，只有超出这个范围的，才会调用new Integer(int)构建新的对象。
补充说明，此处的上限127其实是从Java配置项中读取的，应该是可以更改的；而下限-128是硬编码在Integer库中的。详情见源码。
- 枚举类型
   - 下述枚举类型演示代码实际上是声明了一个类（Suffix），该类继承自Enum类，并实例化了三个对象（EXE，DOC，DOCX），lowercase/Suffix(String)/getLowercase()分别为Suffix类的属性、构造方法和getter方法。`EXE("exe")`, `DOC("doc")`, `DOCX("docx")`就是在调用构造方法构造了三个Suffix类实例。
   - 枚举类型使用==比较即可，因为每一个枚举值都是一个单例对象。
   - 使用枚举变量的toString()方法可以获取到枚举变量名。
```java
public class Main {
	public static void main(String[] args) {
	Suffix s1 = Suffix.EXE;
	System.out.println(s1.getLowercase());
	}
}

enum Suffix {
 EXE("exe"), DOC("doc"), DOCX("docx");

 private String lowercase;

 private Suffix(String lowercase){
	this.lowercase = lowercase;
 }
 public String getLowercase(){
	return lowercase;
 }
}
```

- 反射 com.core.Reflection
   - Class类（和class关键字不同，注意大小写）
      - 一个Class对象实际上表示的是一个类型
      - 虚拟机为每个类型管理一个Class对象
   - getClass()
      - 实例通过getClass()方法获取类信息，本质上获取到的是一个Class实例
   - Class.forName(String str)
      - 静态方法，返回参数str给定的类名所对应的类实例
      - 如果不存在该类，则抛出checked exception异常，所以无论何时使用该方法，都必须try/catch
         - 异常分为已检查异常和未检查异常，其中已检查异常必须由程序员处理（throw或try/catch）。
   - Class#newInstance()
      - 类对象的方法，调用默认构造函数创建一个对象，如果没有默认构造器则抛出异常。
      - 无法通过该方法调用有参数的构造函数。
   - Filed、Method、Constructor
      - 分别代表类中的域、方法、构造器的类
      - 分别通过Class#getFileds()|getMethods()|getConstructors()方法获得对应的私有域、方法、构造器的数组
      - 分别通过Class#getDeclaredFileds()|getDeclaredMethods()|getDeclaredConstructors()方法获得对应的全部域、方法、构造器的数组，但不包含超类成员
      - Field
         - 通过Field#get(Obj o)可获得对象o在指定Filed域上的值，但是该访问受限于Java的安全机制而无法访问private域。同样可以通过Filed#set(Obj o,			value)设置新值。
可以通过Filed#setAccessibleObject(boolean			 flag)设置该值为true屏蔽Java语言的访问检查，使得私有对象也可以被查询和设置。
         - 通过遍历类中的上述三个类可以实现对类结构的分析，比较实际的应用是遍历Class#Fileds[]实现对所有类的通用toString()方法。
      - Method
         - Method#invoke(Object obj, Object… args)



# 接口及内部类


- 接口
   - 接口方法可以不声明为public，接口中的方法都是public的，但是在实现的时候要声明为public，否则将按照默认访问权限处理。
   - 接口中不能包含实例域或静态方法，但是可以包含常量，其会被自动设置为public static final
   - Comparable接口
      - 要实现compareTo(E e)方法，如当前实例小于e，则返回负数；等于则返回；大于则返回正数。
   - Cloneable接口
该接口没有规定任何方法，clone()方法是Object类的protected方法而非Cloneable接口规定的，在实现中应该将其访问限制重置为public。在此处，Cloneable接口称之为标记接口，他的目的是确保类实现了某个特性的方法或者一组特定的方法，并使用instanceof进行类型检查。如果一个对象需要克隆，但没有实现Cloneable接口，就会产生已检查异常（checked exception）
在自己编写程序时，不应该使用上述技术。
- 内部类
内部类可以直接访问自己外部类的域，可以视作有内部类的this指针同时包含内部类实例自身和外部类实例的所有域，甚至是**私有域**。
```java
package com.core.InnerClass;

public class InnerClass {
	public static void main(String[] args){
	 Outter outter = new Outter();
	 outter.print();
	}
}

class Outter{
	private String val = "Outter";
	private String notConflict = "Outter Not Conflict Part";

	public void print(){
	 Inner inner = new Inner();
	 inner.print();
	}

	class Inner{
	 private String val = "Inner";
	 void print(){
		System.out.println(Outter.this.val + " " + this.val + ", " + notConflict);
	 }
	}
}
```

- 
注意上述代码中，内部类使用Outter.this访问到了外部类实例，通过this.val访问到了自身实例。如果域名称不冲突，则可以省略Outter.this，例如内部类访问notConflict实例域时。
内部类是一个编译概念，在虚拟机中不存在内部类，虚拟机中的内部类会以`Outter$Inner`（`$`是内部类外部类的分隔符）的名称作为一般类来看待。[?]至于内部类拥有不同于外部类的域访问权限这种现象是一个复杂的问题，具体机制需要以后再详细了解。
- 局部内部类
在块内部声明的类，不能用public或者private访问说明符进行声明，他的作用域仅限于这个局部类的块中。
局部内部类可以访问局部变量，但该局部变量必须为final。这是为了防止在局部块中创建的内部类对象在局部块后继续使用，未被回收，而局部变量被回收的状况考虑的，此时，内部类对象使用的局部变量实际为该变量的备份。
- 匿名内部类
对于某些只使用一次的类，多数来说是实现了某些适配器接口的对象，可以使用匿名内部类，直接创建一个匿名类的对象。
由于匿名类没有名字，所以也没有构造器，取而代之的是填入超类的参数，使用超类的构造器。
下面的代码展示了匿名内部类java.awt.event.ActionListener（接口），其逻辑是补全需要重写的方法后构建了类，随后new一个新的实例。
```java
new ActionListener() {
	@Override
	public void actionPerformed(ActionEvent e) {
	 // action
	}
}
```

- 对于继承下来的子类而言，与以下代码不同的是，子匿名类可以在小括号中加入参数，由超类构造器调用。
```java
abstract class Super{
	private int arg;
	Super(int arg){
	 this.arg = arg;
	}
	abstract void m();
}
 
new Super(0x12345678) {
	@Override
	void m() {
	 // code
	}
}
```

- Proxy 代理
通过构建代理类，可以实现AOP，在对象上调用任何方法时，可以通过代理类插入切面。见下面这个小例子，Outer代理类代理了Inner对象，并且截获了每一个方法，在Inner对象方法调用前打印了Outer.val
```java
public interface Base {
	void print();
}
```
```java
public class Inner implements Base {
	private String val = "Inner Val";
	public void print(){
		System.out.println(val);
	}
}
```
```java
public class Outer implements Base {
	private String val = "Outer Val";
	private Inner inner;

	Outer(Inner inner){
		this.inner = inner;
	}

	public Base getProxy(){
		return (Base)(Proxy.newProxyInstance(Outer.class.getClassLoader(),
				new Class[]{Base.class},
				new InvocationHandler() {
					@Override
					public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
						print();
						return method.invoke(inner, args);
					}
				}));
	}

	public void print(){
		System.out.println(val);
	}
}
```
```java
public class ProxyPratice {
	public static void main(String[] args) {

		Inner inner = new Inner();
		Outer outer = new Outer(inner);
		Base base = outer.getProxy();

		base.print();
	}
}
```


# 异常、断言、日志和调试


- 异常
   - Java中的异常层次结构
Error类层次结构描述了JRE的内部错误和资源耗尽错误，应用程序不应该抛出这类异常。
Exception中的蓝色部分，即Runtime Exception，属于未检查异常（unchecked exception），应该避免发生。如果出现了此类异常，一定是程序本身出现了问题。
Exception中的粉色部分属于检查异常（checked exception），如果有出现这类异常的可能，则一定要进行处理（使用try/catch或者throws抛出）
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100544686-d37b4193-f7d3-4238-b58c-98ccc28c3d19.png#align=left&display=inline&height=1079&margin=%5Bobject%20Object%5D&originHeight=1438&originWidth=700&size=0&status=done&style=shadow&width=525)
- try/catch/finally
   - 如果try块中有return语句，也会在真正返回之前执行finally块中的语句，如果此时finally块中也有return语句，那么其结果将会覆盖try块中的返回结果。
   - 还有一种try-with-resources，语法为try(){}，该语法可以在小括号中打开某些资源，这些资源将会按照其打开顺序的逆序依次调用close()方法关闭而无需在finally块中手动编码。
- 只在异常状态下使用异常机制，因为异常处理的时间消耗远大于正常程序执行。
- 异常应“早抛出，晚捕获”，让更高层级的方法通知用户发生了异常。
- 断言 assert
   - 两种形式
   - 

      - assert 条件;
         - assert 条件: 表达式;
其中，表达式的作用是生成AssertionError异常的消息字符串
   - 不满足条件的断言会抛出AssertionError异常
   - 断言默认是关闭的
   - 断言只应该用于开发和测试阶段
- 记录日志
此处不记录java自身的日志系统，实际开发中应当使用slf4j这类统一接口以保证各种依赖库及编码过程中的日志统一。
- 调试技巧
   - 调用Threa.dumpStack()可以打印当前线程的堆栈信息
   - 启动JVM时加入-verbose参数查看类加载信息，这有助于诊断由于类路径引发的问题



# 泛型程序设计


## 泛型类、泛型方法


- 泛型方法可以独立于泛型类而定义在普通类中
定义与调用，其中T称为类型参数
```java
public class Core {
 public static void main(String args[]){
	String res = Core.<String>genericMethod("generic method");
 }

 private static <T> T genericMethod(T argu){
	return argu;
 }
}
```

- 可以对类型变量T设置限定，限定类型变量T必须继承自某些类或实现了某些接口
```java
private static <T extends Comparable & Serializable> T genericMethod(T argu){
	return argu;
}
```

   - 使用extends关键字，无论是超类还是接口
   - 使用&连接多个限定
   - 限定中只能有一个超类，可以有多个接口。如果同时存在，则超类排在限定列表的第一个位置
- JVM中没有泛型的概念，泛型类和方法会在编译阶段通过擦除（erase）的方法转换为普通的类和方法，所有的类型参数都会被他们的限定类型替换，如果没有限定类型，则会被Object类替换；并且编译器会为访问添加相应的强制类型转换
- 约束与局限性
   - 不能使用基本类型作为泛型的实例化类型参数，例如：如果需要int类型，则需要使用其包装类Integer作为泛型的实例化类型参数。
其原因是基本型不能在编译阶段进行擦除并寻找超类的操作。例如，Integer类有超类Object，但是int则不属于任何类的子类
   - instanceof和getClass()返回的都是原始类型，例如：任何ArrayList实例在JVM中的原始类型均为java.util.ArrayList；如下条件表达式始终为真。
```java
// always true
new ArrayList<Integer>().getClass() ==	new ArrayList<Double>().getClass()
```

- 不能创建参数化类型的数组
例如：`Pair<String>[] p = new Pair<String>[] //Error`
因为该数组在擦除后会变为`Object[] p = new Object[]`，由于Object数组可以存储任意类，所以对于我们希望的p（只存储String类型），有可能存入其他类型，导致程序出现不可预料的错误。所以，编译器不允许创建参数化类型数组。
- 不能实例化类型变量
例如：`T t = new T();`
上述代码在擦除后会变为`Object t = new Object();`，这与原始代码中表达的意思完全不同
可以通过反射的trick来构造泛型对象，此处不做讨论
- 泛型类在静态上下文中的类型变量无效，例如静态域
因为无论泛型类型如何，他们的原始类型都是相同的，所以实际上不同泛型类型的静态域是同一个，这就导致了他们之间的冲突。例如下面的代码，实际上是不能编译通过的。但如果能够编译并运行，我们调用`GenericInStatic<String>.arrayList<String>`与`GenericInStatic<Integer>.arrayList<Integer>`从理论上来说都指向了同一个`GenericInStatic<Object>.arrayList<Object>`对象。
```java
static class GenericInStatic<T>{
 public static ArrayList<T> arrayList;
}
```

- 通配符类型，一般用于方法参数中的泛型类型定义
   - `? extends ClassType`

任何ClassType的子类

   - `? super ClassType`

任何ClassType的超类

   - `？`

无限定通配符


# 集合


- 集合框架整体类图
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100544690-3e85663e-cb2f-493e-ad27-70f937bbc9b6.png#align=left&display=inline&height=736&margin=%5Bobject%20Object%5D&originHeight=736&originWidth=1080&size=0&status=done&style=none&width=1080)
- Iterator
迭代器删除的是上一次next()方法取到的对象；如果使用ListIterator的previous()方法，那么被删除的将是previous()刚刚取到的对象。
- 散列表 HashMap/HashSet
Java中的散列表使用链表数组实现，每个链表被称为桶（bucket），要想查找表中对象的位置，就要先计算它的散列码，然后与桶的总数取余，得到保存这个对象的桶的索引，然后再在链表中一次访问比较，最终查找到指定元素
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100544988-382848dd-2fbb-4dd2-b74d-2eff34537e21.png#align=left&display=inline&height=320&margin=%5Bobject%20Object%5D&originHeight=320&originWidth=361&size=0&status=done&style=none&width=361)
   - 散列冲突（hash collision）
      - 在Java中表现为对应桶的链表已满
   - 再散列（Rehashed）
当散列表太满，进行再散列增加散列表的空间。此处详情参看源码。
      - 散列因子
默认0.75，散列因子为0.75的意思是当散列表超过75%的位置已经填入元素了，那么就将桶数目翻倍，进行再散列
- TreeSet/TreeMap
红黑树实现
可以在构造TreeSet时传入比较器以自定义排序
```java
TreeSet<Integer> ts = new TreeSet<>(new Comparator<Integer>() {
	@Override
	public int compare(Integer o1, Integer o2) {
		return 0;
	}
});
```

- 优先队列（Priority Queue）
   - 堆实现
   - 典型示例为任务调度
- 视图
   - 通过某些方法可以返回满足特定条件的集合的视图，例如subList等等
   - 某些视图，例如subList返回的视图，可能是匿名类实例，这些匿名类实例通常实现的是List或其他接口，这就导致了他们可能支持接口中声明的操作，而不具备一般集合类所具有的操作。例如有些视图不支持add操作，有些视图不支持set操作。



# 多线程


多线程内容参看单独的笔记。


# JavaBean构件


- JavaBean
   - 方法命名遵循了某些特定的模式，方便开发工具、框架通过反射的手段分析其结构，方便其他程序员使用的可重用组件，称为Java Bean。
   - JavaBean分为有界面JavaBean（Swing中的控件）和无界面JavaBean（数据库操作等）
   - 命名规则
      - 属性
         - 简单属性
set、get方法
         - 索引属性
索引属性指定了一个数组，需要提供两队get和set方法，分别对应数据整体和数据内的单个元素，注意方法名一定要遵循规则，只使用重载来实现不同的功能
```java
Type[] property;
Type[] getProperty()
void setProperty(Type[] newValue)
Type getProperty(int i)
void setProperty(int i, Type newValue)
```

- 绑定属性
   - 绑定属性会在对象值发生变化时，通知所有相关监听器
   - 必须实现两个机制
      1. 一但值发生变化，该bean必须发送一个PropertyChange时间给所有已注册监听器
      1. 为了使其他监听器能够注册，bean必须实现以下两个方法：
```java
void addPropertyChangeListener(PropertyChangeListener listener);
void removePropertyChangeListener(PropertyChangeListener listener);
```

      1. 
还推荐实现以下方法，但不强制：
```java
PropertyChangeListener[] getPropertyChangeListeners();
```

   - java.beans包实现了PropertyChangeSupport类，它能够管理监听器。要使用这个类，需要添加一个类型为该类的实例域成员：
```java
private PropertyChangeSupport propertyChangeSupport = new PropertyChangeSupport(this);
```

   - 
随后，可以将上述的三个监听器管理方法委托给该成员对象。在属性值发生变化时，使用fire相关的方法触发所有的监听器，具体的两个方法如下，分别针对简单属性和索引属性：
```java
propertyChangeSupport.firePropertyChange("propertyName", oldValue, newValue)
propertyChangeSupport.firePropertyChange("propertyName", index, oldValue, newValue)
```

   - 相对应的，如果其他bean想要收到通知，那么必须实现Property Listener接口，该接口只有一个方法：
```java
void propertyChange(PropertyChangeEvent event)
```

   - 
其中PropertyChangeEvent分别存储了上述参数中对应的propertyName、oldValue、newValue，所以说，该类只是参数的简单包装
- 约束属性
约束属性指的是，任何监听器都有权拒绝对该属性的修改，强迫其使用原本的旧值（通过抛出异常使方法跳出以达成该目的）。
要构建约束属性，bean必须包含以下两个方法：
```java
void addVetoableChangeListener(PropertyChangeListener listener);
void removeVetoableChangeListener(PropertyChangeListener listener);
```

- 
还应有一个方法用来获取所有的监听器：
```java
VetoableChangeListener[] getVetoableChangeListeners();
```

- 
java.beans实现了VetoableChangeSupport类，管理用以否决的监听器。bean可以将管理监听器的功能委托给该类的实例。其使用方法与PropertyChangeSupport类基本相同，不再赘述。
- 事件类型
   - 所有的事件类的名字必须以Event结尾，且必须继承自EventObject类
   - 假设自定义了事件类**Something**Event，则其监听器接口必须命名为**Something**EventListener，且添加和删除监听器的方法必须命名为
      - add**Something**EventListener(**Something**EventListener e)
      - remove**Something**EventListener(**Something**EventListener e)
- BeanInfo类
当Bean变得复杂的时候，单纯地依赖于命名规则可能不能很好地反应Bean的特征。例如对于某些Bean，我们不希望系统可以反射出某些对象的getter/setter方法，这些对象只属于我们自用的，而非Bean的属性。
此时，使用一个实现了BeanInfo接口的对象来描述有关Bean的信息，系统就会根据它来识别bean具有的特性
bean信息类的类名必须为他所对应的bean的类名加上BeanInfo而成，bean信息类也必须和bean在同一个包中。例如，有ImageViewerBean，那么他的bean信息类类名应该为ImageViewerBeanBeanInfo
通常情况下不直接实现接口，因为内容太多，一一实现意义不大且工作量太大。实际应用中，继承SimpleBeanInfo即可，它提供了BeanInfo中所有接口的默认实现
bean信息类中存储了诸多PropertyDescriptor，其创建如下：
```java
PropertyDescriptor descriptor = new PropertyDescriptor("fileName", filePickerBean.class);
```

- 
BeanInfo类中有getPropertyDescriptors方法，可以获取所有的PropertyDescriptor，SimpleBeanInfo对于这个方法直接返回null
- JavaBean持久化
   - bean的持久化依赖于通过将bean的非默认状态数据导出为XML文件存储到外部介质，恢复时根据XML文档中的数据构建新对象并对其进行初始化
   - 其中，只导出非默认值的行为称为消除冗余，对于状态属性极多的bean（例如Swing部件），可以极大的节省持久化后的空间占用



# 脚本、编译与注解


## 脚本


可以通过调用脚本引擎执行指定语言，例如通过JavaScript脚本引擎执行脚本代码，形如jsEngine.eval("n += 1")。除了最基本的调用执行环境以外，还可以将java中的变量绑定到执行引擎中，使得脚本可以操作java代码中的变量


## 编译器API


一些需要调用编译器API的情况，一般为代码分析、动态编译：


- 开发环境
- 自动化构建和测试工具
- 处理Java代码段的程序，例如JavaServerPages（JSP）



## 注解


- 注解的常见用法：
   - 附属文件的自动生成，例如bean信息类
   - 测试、日志、事务语义代码的自动生成
- 注解是一种修饰符，以@开头，用于被注解项之前，无分号，例如
```java
@Test(timeout="1000")
public void compute(int x, int y){};
```

- 
注解中包含的`timeout="1000"`这些元素可以被读取**注解**的工具获取并处理，详情见后文
- 注解本身不做任何事情，它需要其他工具的支持才会有用
- 每个注解通过注解接口定义，这些接口中的方法与注解中的元素相对应。例如JUnit的注解Test通过下面这个注解定义：
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test{
	long timeout() default 0l;
	//...
}
```

- 
处理注解的工具将接收到实现了Test的注解接口的对象，并通过timeout()方法获取到注解中的timeout元素
上文中的@Target和@Retention是元注解，他们可以注解注解（呃，有点绕口，前边是动词，后边是名词）
- 标准注解
   - java.lang.annotation和javax.annotation中定义了大量的注解，其中有四种元注解
   - 元注解，一般用于注解注解（Annotate Annotation）
      - Target，指明该注解可以应用于哪些注解，例如：包、类、域、方法、构造器、参数、局部变量
      - Retention，指明该注解可以保留多久。
可选SOURCE、CLASS和RUNTIME，分别对应着注解存在至源码、类文件、JVM级别。上文的@ActionListenerFor注解声明为Retention为RUNTIME，就是为了可以通过反射获取注解
      - Documented，指明该注解应该包含在注解项文档中
      - Inherited，指明当前注解应用于一个类时，能够自动被子类继承
- 注解语法
   - 声明注解接口
```java
modifier @interface AnnotationName{
	elementDeclaration1
	elementDeclaration2
	// ...
}
```

   - 每个元素声明具有如下形式
      - _type elementType_()
      - _type elementType_() default _value_
   - 使用注解，元素的位置无关紧要，未出现的元素将采用默认值
      - @AnnotationName(b=1, a=0)
      - 默认值和注解语句并不是存在一起的，如果注解接口中声明元素a的默认值为`“[none]”`，但随后将注解接口中a的默认值修改为`“none”`并重新编译，则所有的该注解中的a的默认值都会变为`“none”`，即使他们所在的类已经编译完成
      - 简化注解
当只有一个元素时，可以不写出元素名，例如`@Annotation("[Marked]")`
当不使用小括号时，所有的元素都使用默认值，这类注解又称为标记注解，例如`@Annotation`
- 无法扩展注解接口，所有的注解都直接扩展自java.lang.annotation.Annotation
- 注解元素只能为以下类型，且不能为null
   - 基本类型
   - String
   - Class
对应的空为Void.class
   - enum类型
   - 注解类型
例如，@BugReport(ref=@Ref(id="12345"))
这意味着注解元素也可以是另外一个注解，注解是可以嵌套的，但不能循环依赖。例如不能在@BugReport中包含一个@Ref元素，但@Ref中又包含了@BugReport元素
   - 由前面类型组成的数组
      - 使用大括号括起来，`@BugReport(by={"lix", "leaf"})`
      - 数组的数组是非法的
- 运行期注解处理示例，通过注解向组件上添加监听器
   - 常规监听器
```java
myButton.addActionListener(new ActionListener(){
	public void actionPerformed(ActionEvent event) {
		doSomething();
	}
}
```

   - 上述监听器添加可以转化为如下注解：
```java
@ActionListenFor(source="myButton")
void doSomething(){
	// ...
}
```

   - 注解实现如下，核心思路是通过反射的手段读取到对象、类和注解，然后通过反射的机制实现期望的注解想要实现的行为
      - 声明注解接口
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME) 
public @internface ActionListenerFor{
	String source(); 
}
```

      - 实现期望的注解功能
```java
public class ActionListenerInstaller
{
	/**
	* Processes all ActionListenerFor annotations in the given object.
	* @param obj an object whose methods may have ActionListenerFor annotations
	*/
	public static void processAnnotations(Object obj)
	{
		try
		{
		 Class<?> cl = obj.getClass();
		 // 遍历指定对象所属的类声明的方法
		 for (Method m : cl.getDeclaredMethods())
		 {
			// 取方法中的ActionListenerFor注解对象
			ActionListenerFor a = m.getAnnotation(ActionListenerFor.class);
			if (a != null)
			{
				// 读取注解对象中的域source
				Field f = cl.getDeclaredField(a.source());
				f.setAccessible(true);
				// 将方法m作为监听器添加到对象source上
				// 其中f.get(obj)指的是获取注解对象中的域的值（此处是source值）
				addListener(f.get(obj), obj, m);
			}
		 }
		}
		catch (ReflectiveOperationException e)
		{
		 e.printStackTrace();
		}
	}

	/**
	* Adds an action listener that calls a given method.
	* @param source the event source to which an action listener is added
	* @param param the implicit parameter of the method that the listener calls
	* @param m the method that the listener calls
	*/
	public static void addListener(Object source, final Object param, final Method m)
		 throws ReflectiveOperationException
	{
			// 创建反射Handler，在param对象上执行m方法
		InvocationHandler handler = new InvocationHandler()
		 {
			public Object invoke(Object proxy, Method m, Object[] args) throws Throwable
			{
				return m.invoke(param);
			}
		 };
		// 创建ActionListener接口的代理对象listener（可以理解为实例化了一个实现了ActionListener接口的匿名类），反射对象行为通过handler处理（即调用指定的监听器方法m）
		Object listener = Proxy.newProxyInstance(null,
			new Class[] { java.awt.event.ActionListener.class }, handler);
		// 获取要添加监听器对象的类上的addActionListener(ActionListener)方法
		Method adder = source.getClass().getMethod("addActionListener", ActionListener.class);
		// 在source上调用addActionListener方法，传入的参数（监听器）为listener，为source添加了listener监听器
		adder.invoke(source, listener);
	}
}
```

- 源码级注解处理
由于源码级注解不将注解信息保留到虚拟机中，所以无法通过反射的方式来处理
从Java SE6开始， 可以将注解处理器添加到Java编译器中。编译器会定位源代码中的注解，随后依次调用注解处理器；如果注解产生了新的文件，则在新文件上重复该过程直到不再有新文件产生
```bash
javac -processor ProcessorClassName1,ProcessorClassName2... sourceFiles
```

- 
具体操作及API请看Java Core II第十章10.6 源码级注解处理，后附代码清单详细写明了调用过程及逻辑
- 类文件级注解处理 字节码工程
   - 直接修改类文件字节码
   - 在类文件加载过程时修改字节码
使用Java SE5后添加的instrument API中提供的安装字节码转化器钩子，具体请看Java Core II第十章10.7 字节码工程，后附代码清单详细写明了调用过程及逻辑



# 流与文件


## 流


- 在Java API中，可以从中读出一个字节序列的对象称作**输入流**，可以向其中写入一个字节对象的称为**输出流**。这些字节序列的来源和目的地可以为文件、网络连接甚至是内存块。抽象类InputStream和OutputStream构成了输入/输出（I/O）类层次的基础
- 由于面向字节的流不方便处理Unicode信息，所以从抽象类Reader和Writer中继承出了专门用于处理Unicode字符的单独的类层次结构，这些类的读写操作都是基于两字节Unicode码元的，而不是基于单字节字符。
- 读写字节
   - InputStream和OutputStream分别有read和write方法，这两个方法在执行时将会阻塞，直到字节被读出或写入。
   - 输入流的available方法可以用于获取当前可读的字节数量，配合read(Array)方法可以一次直接读书所有可读数据
   - 在使用完毕流后应当关闭，否则会占用系统资源
- Java中的流家族
   - InputStream和OutputStream的类结构见下图，有点糊，但已经是能找到的最高清晰度了
![](https://cdn.nlark.com/yuque/0/2019/gif/657413/1577100544692-ee95dd91-2ac0-44a2-b29d-1088866c33c4.gif#align=left&display=inline&height=633&margin=%5Bobject%20Object%5D&originHeight=633&originWidth=500&size=0&status=done&style=none&width=500)
- 相对于Stream处理字节，抽象类Reader和Writer的子类可以处理Unicode文本，其中类结构如下
![](https://cdn.nlark.com/yuque/0/2019/gif/657413/1577100544718-449dee87-75dd-48cc-a8ea-4124154ee6ec.gif#align=left&display=inline&height=449&margin=%5Bobject%20Object%5D&originHeight=449&originWidth=500&size=0&status=done&style=none&width=500)
- 由于InputStream只能读取出字节，而DataInputStream只能读入数值类型，所以Java使用了类似责任链模式的方法，使得多个Stream可以协同工作。通过将InputStream传入DataInputStream作为参数，使得DataInputStream可以从字节流中读取数值类型。
例如，从文件中读取数值，需要使用以下代码：
```java
FileInputStream fis = new FileInputStream("val.dat");
DataInputStream dis = new DataInputStream(fis);
dis.readDouble();
```

- 
上述的依次读取字节或数值，每次read都会调用I/O阻塞当前线程，所以效率很低。使用BufferedInputStream包装FileInputStream会一次性读取一个数据块到缓存中，提升连续读写效率。最后使用DataInputStream包装BufferedInputStream获取数值型数据，代码如下
```java
DataInputStream dis = 
	new DataInputStream(
	new BufferedInputStream(
		new FileInputStream("file.dat")));
```

- 文本输入与输出

# Unsafe

概括的来说，Unsafe 类实现功能可以被分为下面 8 类：

- 内存操作
- 内存屏障
- 对象操作
- 数据操作
- CAS 操作
- 线程调度
- Class 操作
- 系统信息

ref: [Java 魔法类 Unsafe 详解 | JavaGuide](https://javaguide.cn/java/basis/unsafe.html#unsafe-%E5%88%9B%E5%BB%BA)

