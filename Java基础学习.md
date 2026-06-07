# 1.Java基础

## 1.数据类型:

## 2.运算符

1.++i 与 i++ 在表达式中的区别。
	2.== 比较包装类型时（如 Integer 在 -128~127 有缓存，之外会 new 新对象），容易误解。
	3.>> 与 >>> 对负数的行为。
	  >>对负数没有影响，高位是补1的
	  >>>对附属有影响，高位补0
	4.移位取模：1 << 32 在 int 下等于 1 << 0 = 1。
	5.浮点数取余：% 也能用于浮点数（5.0 % 2.2 = 0.6），但结果不精确。
	6.& 和 | 不会短路，可能导致不必要的计算或异常。
	7.赋值运算的隐含转型：short s = 1; s = s + 1; 报错；s += 1; 正确。
	8.布尔运算符的优先级：& 高于 | 高于 && 高于 ||，但建议用括号。

## 3.流程控制

### 1.条件语句 if-else switch-case

​	switch 可以支持byte short char int  不支持 long float double boolean，java5 新增 枚举类enum ,java7 新增 String
​		switch 没有break 会一直执行下去

### 2.循环语句 for while do-while

​	增强for循环 只能用于遍历数组和集合 不能修改元素值

### 3.跳转语句 break continue return

### 4.异常处理 try-catch-finally throw throws

​	finally块中的return会覆盖try块中的return

### 复习建议

1. **亲自手写每个例子**，观察输出，理解执行路径。
2. **刷选择题**：牛客网、LeetCode 上 Java 语言基础题部分。
3. **注意陷阱**：`switch` 穿透、循环条件边界、`break` 与 `continue` 配合标签、增强 for 的只读性。
4. **结合 `return` 与 `finally`**：理解方法执行流程。

## 4.关键字

### 1.static(静态)

​	静态变量：类变量，所有实例共享，可以通过类名直接访问
​		静态方法：只能访问静态成员，不能使用this,super
​		静态快：类加载时执行一次，常用于初始化静态资源
​		静态内部类：不需要不外类实例即可闯将
​		静态导入：import static：导入类的静态成员，可以直接使用

### 2.final（不可变）

​	final变量：基本类型值不可变；引用类型不能指向新对象，但对象内部状态可变
​		final方法：不能被子类重写（但可被重载）
​		final类：不能被继承
​		空白final：声明时未赋值，需要在构造函数中赋值

### 3.this 和 super

​	this:指向当前对象实例，用于区分成员变量和参数，调用本类其他构造器
​		super:指向分类对象，用于访问父类成员，调用父类构造器
​		两者必须在构造器的第一行（不能同时出现）

### 4.abstract(抽象)

​	抽象类：不能实例话，可包含抽象方法和具体方法
​		抽象方法：只能声明，没有方法体，必须由子类实现(除非子类也是抽象类)

### **5.synchronized(同步锁)**

​	**实例方法：锁住当前对象this**
​		**静态方法：锁住class对象**
​		**代码块：可指定任意对象作为锁**
​		**底层：依赖对象头的monitor（监视器锁），存在锁升级过程**

### 6.volatile(可见，有序性)

​	**保证可见性：修改后立即写回主内存，其他线程读取最新值**
​		**禁止指令重排序：通过内存屏障实现**
​		**不保证原子性：volatile int count 和 count++线程不安全**

### 7.transient

​	**修饰实例变量，表示该字段不参与序列化(Serializable接口)**
​		**反序列化后，transient字段被初始化为默认值(对象为null,基本类型为0/false)**

### 8.native

​	修饰方法，表示方法由本地代码(C/C++)实现，不提供方法体
​		常用于JNI(Java Native Interface),如 Object.getClass、System.arraycopy()

### 9.strictfp(浮点严格模式)

​	修饰类、接口、方法，保证浮点运算在不同平台上结果一致(不省略中间精度)

### 10.assert(断言)

​	开发调式工具，运行需-ea开始。条件为false时抛出AssertionError。
​		生产环境一般不使用

### 11.enum(枚举)

​	隐式继承java.lang.Enum,不能手动继承其他类
​		可以有构造器(私有)、字段、方法、常用于单例、状态机



### 12.面试问题：

**1. `static` 成员变量和实例变量的区别？**

- 存储位置：静态变量在方法区（JDK 8 后元空间），实例变量在堆内存。
- 生命周期：静态变量随类加载而存在，类卸载才销毁；实例变量随对象创建/回收。
- 访问方式：静态变量可通过类名直接访问，实例变量必须通过对象引用。
- 共享性：静态变量被所有实例共享，实例变量每个对象独立。

------

**2. `final`、`finally`、`finalize()` 的区别？**

- **final**：关键字，修饰类/方法/变量。
- **finally**：异常处理块，无论是否异常都会执行（除非 `System.exit`）。
- **finalize()**：Object 的方法，GC 回收对象前可能被调用（已废弃，不推荐使用）。

------

**3. 简述 `transient` 的使用场景，并举例。**

- 场景：字段不需要被持久化（如密码、敏感信息），或字段可以根据其他字段计算得出（缓存字段）。
- 例：实现 `Serializable` 的 `User` 类中，`password` 字段用 `transient` 修饰，反序列化后为 `null`，需重新获取。

------

**4. `assert` 关键字的作用是什么？什么时候用它？**

- 作用：断言条件为真，为假时抛出 `AssertionError`。
- 使用场景：开发测试阶段检查内部不变量（如参数非空、状态合法），生产环境一般禁用。

------

**5. `native` 方法如何调用？**

- 通过 JNI（Java Native Interface），编写 C/C++ 代码并编译成动态链接库（`.dll` / `.so`）。
- 使用 `System.loadLibrary()` 加载库，Java 声明 `native` 方法后可直接调用。

------

### 附加进阶题（常考）

**6. `volatile` 能否保证线程安全？为什么？**
不能完全保证。它只保证单个读/写的原子性（如 `volatile long` 赋值），但不保证复合操作的原子性（如 `i++`）。如果要保证线程安全，仍需配合 `synchronized` 或 `Lock`。

**7. `synchronized` 和 `volatile` 的区别？**

| 特性     | `synchronized`     | `volatile`          |
| :------- | :----------------- | :------------------ |
| 原子性   | 保证（代码块整体） | 不保证              |
| 可见性   | 保证               | 保证                |
| 有序性   | 保证               | 保证（禁止重排序）  |
| 阻塞     | 可能阻塞线程       | 不阻塞              |
| 适用场景 | 复合操作、互斥     | 状态标志、单次读/写 |

### 复习建议

1. **分主题记忆**：把 `static`、`final`、`volatile`、`transient` 等对比记忆。
2. **手写代码验证**：例如写一个包含 `transient` 字段的类进行序列化/反序列化测试。
3. **熟悉反例**：`final` 修饰集合时内容可变；`volatile` 不能替代锁。
4. **结合 JVM**：理解 `static` 方法区、`synchronized` 对象头、`volatile` 内存屏障。
5. ***private`、`protected`、`public*** ：这三个字段需要研究一下



## 5.面向对象（OOP）

### 1.三大特性：

#### 1.封装：

​	保护数据完成性，隐藏内部实现，降低模块间耦合

#### 	2.继承：

​	extends关键字，方法重写（@Override）
​		先调用父类的无参构造器，再执行子类构造器

```
	class Parent {
   		Parent() { System.out.print("P "); }
	}
	class Child extends Parent {
   		Child() { System.out.print("C "); }
	}
	public class Test {
   		public static void main(String[] args) {
       		new Child();
   		}
	}
输出：P C

```

#### 3.多态：

​	方法重载 编译看左边（检查类型是否有该方法，运行看右边(实际执行子类重写后的方法)）

```
	class Animal { void shout() { System.out.println("Animal"); } }
	class Dog extends Animal { void shout() { System.out.println("Dog"); } }
	Animal a = new Dog();
	a.shout();
输出：B
```

### 2.抽象类vs接口

#### 抽象类：

关键字 abstract,不能实例化，可包含抽象方法和具体方法
	可以有构造器(供子类调用)、成员变量、静态方法等

#### 接口：

Java8之前：只能有public static final 常量和public abstract方法
	java8:允许default 和static方法(有方法体)
	java9:允许private方法
	接口支持多继承(一个类可实现多个接口)

### 3.内部类

成员内部类：可以访问外部类的所有成员变量(包括私有的)，需要外部类实例来创建

静态内部类：用static修饰，不依赖外部类实例

局部内部类：定义在方法内，只能访问方法的final或者effectively final 变量

匿名内部类：没有名字，必须继承一个类或者实现一个接口

### 4.构造器与初始化块

构造器：与类同名，无返回值，可重载，默认有无参数构造(如果未显示定义)

初始化块：{}实例块，在构造器之前执行；static{}静态块，类加载时执行一次

执行顺序：父类静态块->子类静态块->父类实例块->父类构造器->子类实例块->子类构造器

### 5.向上转型 和 向下转型

向上转型：子类->父类(自动，安全)，丢失子类的特有方法

向下转型：父类->子类(需强制转换)，可能抛出classCastException，建议用instanceof判断

### 面试问题：

```
class Base {
    public Base() { print(); }
    void print() { System.out.println("Base"); }
}
class Derived extends Base {
    int x = 2;
    Derived() { print(); }
    void print() { System.out.println("Derived x=" + x); }
}
public class Test {
    public static void main(String[] args) {
        new Derived();
    }
}
输出：
Derived x=0
Derived x=2
解释：
调用 Derived 构造器前先调用 Base 构造器，Base 构造器中调用了 print()，由于多态，实际执行 Derived 的 print()，此时子类实例变量 x 还未初始化（默认为 0）。

然后回到 Derived 构造器，先完成实例变量初始化（x=2），再执行构造器体内的 print()，输出 Derived x=2。
```

### 复习建议：

1. **动手编码**：亲自写继承、多态、内部类的例子，观察执行顺序。
2. **对比记忆**：抽象类 vs 接口、重载 vs 重写、向上转型 vs 向下转型。
3. **刷题**：牛客网、LeetCode 上的 Java 面向对象选择题很有帮助。
4. **理解 JVM 层面**：多态依赖动态绑定（虚方法表）、`super` 和 `this` 的字节码。

## 6.核心API 和常用类

```
1.Object
2.String：final修饰 不可修改，线程安全
  StringBUilder:可变，线程不安全，推荐单线程中字符串拼接
  StringBuffer：可变，线程安全（synchronized），性能略低
  intern()：手动写入常量池
  例题:
  	String a = "ab";
	String b = "a" + "b";
	System.out.println(a == b);        // true 常量折叠，编译时变成 "ab"，与 a 指向同一常量池
	String c = "a";
	String d = c + "b";
	System.out.println(a == d);        // false 变量参与拼接，会在堆上创建新对象
3.包装类（Wrapper Classes）
	Integer、Long、Short、Byte、Character 对部分值（默认 -128~127）使用缓存，valueOf() 返回缓存对象。
	Integer i1 = 100;
	Integer i2 = 100;
	Integer i3 = 200;
	Integer i4 = 200;
	System.out.println(i1 == i2);//true 100 在缓存范围内，i1 和 i2 指向同一对象
	System.out.println(i3 == i4);//false 200 超出范围，i3 和 i4 分别是新对象
4.Math类
	round()：返回 long 或 int，四舍五入（Math.round(3.5) = 4，Math.round(-3.5) = -3）
5.日期时间	
	ava.util.Date：表示特定时刻，大部分方法已废弃。toString() 格式不友好。
	java.util.Calendar：抽象类，常用子类 GregorianCalendar，用于日期计算。但存在线程不安全、月份从 0 开始等问题。
	Java 8+ java.time：
		LocalDate、LocalTime、LocalDateTime：不可变，线程安全。
		DateTimeFormatter：线程安全，用于解析和格式化。
		Instant：时间戳。
		Duration / Period：时间间隔。
6.scanner 和 System 
	Scanner：从输入流（如 System.in）解析基本类型和字符串。常用：nextInt()、nextLine()、hasNext()。
	System：
		System.out / System.err / System.in。
		System.currentTimeMillis()：毫秒级时间戳。
		System.nanoTime()：纳秒级，用于测量时间间隔（不受系统时间调整影响）。
7.Random 和 ThreadLocalRandom
	Random：伪随机数生成器，线程安全（但竞争时性能差）。
	ThreadLocalRandom：JDK 7+，每个线程独立随机数生成器，性能高，推荐多线程环境。
8.Arrays
	排序：Arrays.sort(arr)（基本类型使用双轴快速排序，对象使用 TimSort）。
	二分查找：Arrays.binarySearch(arr, key)（必须先排序）。
	拷贝：Arrays.copyOf(arr, newLength)、copyOfRange()。
	填充：Arrays.fill(arr, val)。
	比较：Arrays.equals(arr1, arr2)（比较内容而非引用）。
	转字符串：Arrays.toString(arr) 用于一维数组；Arrays.deepToString(arr2D) 用于多维数组。
	并行操作（Java 8+）：parallelSort()、parallelPrefix()、parallelSetAll()。
9.Collections
	排序：Collections.sort(list)（要求 List 元素实现 Comparable），或传入 Comparator。
	反转：Collections.reverse(list)。
	随机打乱：Collections.shuffle(list)。
	不可变集合：unmodifiableList()、unmodifiableMap() 等（只读视图，修改会抛异常）。
	线程安全包装：synchronizedList()、synchronizedMap() 等（返回线程安全的委托对象）。
	单元素集合：singletonList()、singletonMap()。
	频率：frequency(collection, element)。
	最值：min()、max()。
	问题：
	Collections.synchronizedList(list) 和 CopyOnWriteArrayList 的区别。
		答：synchronizedList 通过同步方法包装，所有操作都加锁，迭代时需手动同步；CopyOnWriteArrayList 采用写时复制策		略，读操作无锁，适合读多写少的场景。本题属于基础集合工具类，了解即可。
```

## 7.集合框架

### 1.整体框架

- **两大根接口**：`Collection` 和 `Map`。
	- `Collection` 存储单个对象，包括 `List`（有序可重复）、`Set`（无序不重复）、`Queue`（队列）。
	- `Map` 存储键值对，键唯一。
- **Java 8 新增**：`Stream`、`Lambda` 对集合的操作支持。
- **迭代器**：`Iterator`（`hasNext()` + `next()`）和 `ListIterator`（双向迭代）。
- **`Comparable` vs `Comparator`**：自然排序 vs 定制排序。

### 2.list

```
ArrayList:
	底层：动态数组（默认容量10，1.5倍扩容）
	特点：随机访问快（O(1)）,中间插入、删除慢（O（n））
	线程不安全：多线程环境需手动同步或使用CopyOnWriteArrayList
	fail-fast:迭代时修改集合（结构性修改）会抛出ConcurrentModifcationException(通过modCount实现)
LinkList:
	底层：双向列表
	特点:插入/删除快（O（1））,中间插入/删除慢（O（n））
	实现了Deque,可做双端队列使用
Vector/Stack(旧版，了解即可)
	Vector 线程安全（方法synchronized）,性能较差
	stack继承Vector，栈操作，推荐使用Deque替代
题目：
	List<String> list = new ArrayList<>();
	list.add("A");
	list.add("B");
	for (String s : list) {
    	if ("A".equals(s)) list.remove(s);
	}
	答：抛出 ConcurrentModificationException。增强 for 循环底层使用迭代器，迭代过程中直接调用 list.remove() 会修改 modCount，导致迭代器检测失败。解决：使用迭代器的 remove() 方法，或 Java 8 removeIf()。
```

### 3.Set

```
HashSet:
	底层：HashMap(Value用一个固定的PRESENT对象)
	特点：无序、不重复、依赖hashCode()和equals()
	初始容量16，负载因子0.75，扩容为2倍
LinkedHashSet
	继承HashSet,底层使用LinkedHashMap,维护双向链表记录插入顺序
	有序（插入顺序）
TreeSet
	底层：红黑树
	特点：元素有序（自然排序或定制Comparator）,add/remove复杂度O（log N）
	元素必须实现Comparable,否则构造时需传入Comparator
题目：
	Set<Student> set = new HashSet<>();
	set.add(new Student(1, "A"));
	set.add(new Student(1, "A"));
	System.out.println(set.size());
	答：2。因为默认 hashCode 返回内存地址，两个对象不同，所以算两个元素。需要重写 hashCode 和 equals 基于 id 和 name 来判断重复。
```

### 4.Map

```
HashMap
	底层：JDk1.8前：数组+链表；jdk1.8：数组+链表/红黑树
		链表长度>8且数组长度>=64时，链表转为红黑树；树节点数<6时退化为链表
	初始容量16，负载因子0.75，扩容为2倍
	hash算法：(key == null)?0:(h=key.hashcode())^(h>>>16)(扰动函数，高位参与运算)
	put流程：计算hash->定位索引->若为空则插入->若存在则比较hash和equals->替换旧值或插入链表/树->检查是否需要树化或扩容
	线程不安全：jdk1.7扩容时头插法可能死循环；jdk1.8使用尾插法，但数据任然可能丢失，依旧需要并发控制
	允许null key和null value (null value 的hash 为0，放在table[0])
LinkedHashMap
	继承HashMap，内部维护双向链表记录插入顺序或访问顺序（accessOrder）
	可用于实现LRU缓存（重写removeEldestEntery）
HashTable(旧版)
	线程安全（方法synchronized),不允许null key/value
	初始容量11，扩容为2倍+1
	已淘汰，被ConcurrentHashMap替代
TreeMap
	底层：红黑树，键有序（自然排序或Comparator）
	操作复杂度O（log n）,不允许 null key(除非Comparator支持)
ConcurrentHashMap
	线程安全：jdk1.7分段锁（Segment）,Jdk1.8使用CAS+synchronized锁住链表/树头节点
	不允许null key/value
	读操作不锁，写操作仅锁定特定锁，并发度高
```

### 5.队列与双端队列

```
Queue:offer()(添加)、poll()(移除并返回头)、peek()(返回头不移)
Deque：双端队列，支持addFirst/addList、offerFirst/OffLast、pollFirst/pollLast等
实现类：ArrayDeque(循环数组，推荐)、LinkedList(也实现了Deque)、PriorityQueue(优先级队列，基于堆，非FIFO)

问题：ArrayDeque 和 LinkedList 作为队列时，哪个性能更好？为什么？
答：ArrayDeque 通常更好，因为它基于循环数组，内存连续，CPU 缓存友好，且没有链表节点的额外开销。LinkedList 每次操作需要分配节点对象，GC 压力更大。
```

### 6集合与排序、比较

```fa
1.Comparable:
	实现compareTo(T o)方法，定义自然顺序
	例如：Integer、String等都实现了该接口
2.Comparator:
	实现了compare(T o1,T o2)方法，可定义多个不同的排序规则
	使用了Collections.sot(list,comparator)或TreeSet(comparator)
```

### 7.集合框架中的算法与遍历

```
迭代器遍历、增强for循环（语法糖，底层还是迭代器）、forEach+Lambda(list.foreach(System.out::printIn))
removeIf():java8引入，使用Predicate条件删除元素，避免ConCurrentMOdificationException
list排序：list.sort(null)或list.sort(comparate)
```

### 8.复习建议

1. **手画结构图**：画出 `Collection` 和 `Map` 的继承体系，标注各实现类的底层数据结构。
2. **源码阅读**：重点关注 `HashMap` 的 `put`、`resize`、`treeifyBin` 方法。
3. **对比记忆**：
	- `ArrayList` vs `LinkedList`
	- `HashSet` vs `TreeSet`
	- `HashMap` vs `LinkedHashMap` vs `TreeMap` vs `ConcurrentHashMap`
	- `HashMap` vs `Hashtable`
4. **刷题练习**：牛客网、LeetCode 上的集合相关选择题和代码题。

## 8.JVM

### 1.内存模型

| 区域       | 存储内容                                 | 线程共享           |
| ---------- | ---------------------------------------- | ------------------ |
| 堆         | 对象实例、数组                           | 是                 |
| 方法区     | 类信息、常量、静态变量、JIT编译后的代码  | 是                 |
| 虚拟机栈   | 局部变量表、操作数栈、动态链接、方法出口 | 否（每个方法私有） |
| 本地方法栈 | native方法执行环境                       | 否                 |
| 程序计数器 | 当前线程执行的字节码行号                 | 否                 |

jdk1.8变化：方法区被元空间替代，使用本地内存（不在虚拟机内存中），默认无上限

### 2.对象创建与内存布局（堆的逻辑）

```
对象创建过程
	1.类加载检查（常量池是否定位到类符号引用，检查类是否已经加载/初始化）
	2.分配内存（指针碰撞/空闲列表，取决于堆是否规整）
	3.初始化零值（实例变量赋默认值）
	4.设置对像头（Mark Word，类型指针、数组长度）
	5.执行<init>方法（构造函数）
对象内存布局
	对象头：MarkWord(哈希码、GC分代年龄、锁状态标志)、类型指针（指向类元数据）
	实例数据：实例字段（包含父类继承）
	对其填充：保证对象大小为8字节的整数倍
```

### 3.垃圾回收

```
1.对象存活判断
	引用计数法：（Java不用，无法解决循环引用）
	可达性分析（Java使用）：从GC roots出发，无法到达的对象视为可回收。
		GC roots:虚拟机栈引用的对象、静态属性引用的对象、常量引用的对象、JNI引用的对象等。
2.四种引用类型
	强引用：永不回收（除非无引用），普通new
	软引用：内存不足时回收 ，缓存
	软引用：下次GC即回收，WeakHashMap
	虚引用：任何时候都可能回收，无法通过它获取对象 ，跟踪GC回收（DirectByteBuffer）
3.GC算法
	标记-清除：标记存活对象，标记未清楚的。缺点：生产碎片
	复制：将存活对象复制到另一块区域，适合新生代（Eden+S0/S1）.缺点：浪费内存
	标记-整理：标记存活对象，向着一端移动，然后清理边界外内存。适合老年代
4.分代收集
	新生代：Eden(占8/10)、S0（1/10）、S1（1/10），使用复制法
		对象在Eden分配，Minor GC后存活的对象复制到S0/S1,年龄+1，达到阈值（默认15）晋升老年代
	老年代：存放长期存活对象，使用标记-整理或标记-清除
	Full GC：堆整个堆（包括元空间）进行回收，代价大
5.常见的垃圾收集器
问题：	
	如何判断一个对象是否“已死”（可回收）？
	答：通过可达性分析，从 GC Roots 出发不可达。但还要考虑对象的 finalize() 方法（已废弃），以及软/弱/虚引用。实际生产中，只要对象不可达，基本就会被回收。

```

### 4.类加载机制

```f
类加载过程：	加载->验证->准备->解析->初始化->使用->卸载
	加载：获取二进制流，生成class对象
	验证：确保字节码安全
	准备：为静态变量分配内存空间并赋零值（final Static在此直接赋值）
	解析；将符号引用替换成直接引用
	初始化：执行<clinit>类构造器（静态变量赋值和静态块）
类加载器
	Bootstrap ClassLoader （C++实现，加载rt.jar）
	Extension ClassLoader（加载ext目录）
	Application ClassLoader（加载classpath下的类）
	双亲委派模型：类加载收到加载请求，先委派给父类，父类无法加载才自己加载。
		好处：防止核心类被篡改（如自定义 Java.lang.String不会被加载）
```

### 5.JVM 的调优和工具

```
常用参数：
	-Xms:初始堆大小
	-Xmx:最大堆大小
	-Xss:栈大小
	-XX:MetaspaceSize/-XX；MaxMetaspaceSize
	-XX:+PrintGCDetails：打印GC日志
	-XX：+UseG1GC>:使用G1垃圾回收器
常用工具：
	jps、jstack（线程快照）、jmap（堆转储）、jstat（GC监控）、VisualVM、Arthas
问题：
	CPU 飙升 100%，如何用 JVM 命令排查？
答：top 找到 Java 进程 PID。
	top -Hp PID 查看线程 CPU 占用，找到高占用线程 ID（十进制）。
	转换为十六进制。
	jstack PID | grep 十六进制 查看线程栈，定位代码行。
```

### 6.综合问题

```
题目：请说明 String s = new String("hello") 创建了几个对象？
答案：1 或 2。如果常量池中已有 "hello"，则在堆中创建一个对象；如果常量池中没有，则在常量池中创建 "hello" 对象，再在堆中创建新对象（共 2 个）。

简答题：什么是 OutOfMemoryError？举例几种可能导致 OOM 的场景。
答：JVM 内存不足无法分配对象时抛出。常见场景：

堆 OOM：不断创建对象且被引用无法回收。

栈 OOM（或 SOF）：递归过深。

元空间 OOM：大量动态生成的类（如 CGLib 代理）。

直接内存 OOM：ByteBuffer.allocateDirect 使用不当。

选择题：以下关于 finalize() 方法的描述，正确的是？
A. 在任何情况下，对象被回收前都会调用一次 finalize()
B. 可以手动调用 finalize()，效果与 GC 调用相同
C. 已废弃，不推荐使用
D. finalize() 中可以使对象重新被引用

答案：C、D（A 错误：GC 只调用一次，若对象在 finalize 中复活，下次回收不会再调用；B 错误：手动调用不触发 GC 动作；C 正确，JDK 9 标记为废弃；D 正确，在 finalize 中将 this 赋值给某个可达引用可复活对象，但不推荐）。
```

### 7.复习建议

```
理解 JMM（Java 内存模型） 与本节的 JVM 内存区域不要混淆，JMM 是并发相关的规范。
重点背诵：GC Roots 包括哪些、垃圾回收算法、CMS 和 G1 的区别、双亲委派模型。
实验验证：写代码制造 StackOverflowError / OutOfMemoryError，观察不同参数下的 JVM 行为。
结合工具：熟悉 jps/jmap/jstack 基本用法。
```



## 9.并发编程

### 1.线程基础

```
进程vs线程：进程是资源分配单元，线程是CPU调度单元；线程共享是进程堆和方法区，拥有独立的栈和PC
线程创建方式：
	1.继承Thread类，重写run()
	2.实现Runnable接口,传给Thread
	3.实现Callable接口+FutureTask(可返回结果、抛异常)。
	4.使用线程池（ExecutorService）
线程状态：
	new：创建未启动
	runnable：就绪+运行中（JVM视角）
	blocked：等待锁（synchronized）
	waiting：无期限等待（wait/join/park）
	time_wating:限时等待（sleep/wait(time)/join(time)）
	terminated:结束
常用方法：
	start():启动线程
	join():等待该线程结束
	sleep():休眠(不释放锁)
	yield():让出CPU(不一定生效)
	interrupt():设置中断标志，配合isInterrupted()或 InterruptedException响应
问题：
	run() 和 start() 的区别？
答：start() 会创建新线程并执行 run()；直接调用 run() 只是在当前线程中执行方法，不会启动新线程。
```

### 2.共享性 和 可见性（JMM基础）

```
Java内存模型（JMM）:规定线程间如何通过主内存和工作内存交互
	所有变量存于主内存
	每个线程有自己的工作内存（缓存，寄存器），线程堆变量的操作必须在工作内存中进行，不能直接读写主内存
	可见性：一个线程修改共享变量，其他线程能立即看到
volative 关键字：
	保证可见性（修改后立即写回主内存，其他线程读取时强制从内存读）
	禁止指令重排序（内存屏障）
	不保证原子性（如 volative int count,count++ 依旧是非原子性）
synchronized:即能保证原子性，又能保证原子性（互斥）
happens-before规则：JMM堆程序员承诺的可见性保障，如解锁happen-before加锁、volative写，happen-before读等
问题一：
	解释 Java 内存模型中的“工作内存”是什么？与 JVM 内存区域的关系？
答：工作内存是 JMM 的抽象概念，对应 CPU 缓存、寄存器等，不是 JVM 的运行时数据区。每个线程的工作内存独立，存储变量的副本。主内存对应堆中的共享变量。
问题二：下面代码可能死循环吗？为什么？
	boolean flag = true;
	// 线程A
	while (flag) { }  // 忙循环
	// 线程B
	flag = false;
答案：可能死循环，因为 flag 没有用 volatile 修饰，线程A 可能一直读取自己工作内存中的缓存值，看不到线程B 的修改。加上 volatile 可解决。
```



### 3.锁机制

```
1.synchronized
	用法：修饰实例方法（锁当前对象）、静态方法（锁Class对象）、代码块（锁指定对象）。
	底层原理：每一个对象有一个monitor（监视器锁），通过monitorenter/monitorexit指令实现
	锁升级（JDK1.6优化）：无锁->偏向锁->轻量级锁->（CAS自旋->重量级锁（OS互斥锁）
	可重入:同一线程可以多次获取同一个锁
2.Lock接口（ReentrantLock）
	与synchronized区别：
		可中断等待（lockInterruptibly()）
		可尝试非阻塞获取锁（tryLock()）
		可设置公平锁（new ReentrantLock(true)）
		支持多个条件变量（Condition）
		必须手动释放锁（unlcok()）
	底层依赖AQS(AbstractQueuedSynchronizer)
3.乐观锁 & CAS
	CAS：原子操作，比较并变化，有CPU指令支持（cmpxchg）
	ABA问题：值从A->B->A,CAS会误认为没变。通过版本号/时间戳解决（AtomicStampedReference）
	java中的CAS:Unsafe类，AtomicInteger等原子类底层
```

### 4.并发工具类

```
原子类（java.util.concurrent.atomic）：
	AtomicInteger、AtomicLong、AtomicReference、AtomicBoolean
	通过CAS实现原子操作，如incrementAndGet()
	数组原子类：AtomicIntegerArray
	字段更新器：AtomicIntegerFieldUpdater(基于反射，对volatile字段进行原子更新)
显示锁与条件
	ReentrantLoock+Condition(类似wait/notify,更灵活，支持多条件队列)
	ReentrantReadWriteLock:读写锁，读锁共享，写锁互斥
	StampedLock(java 8):乐观读锁，提交并发
同步辅助类
	CountDownLatch:计数器，一个或多个线程等待其他线程完成操作。不可重用
	CyclicBarrier:可重用屏障，一组线程相互等待达到屏障后继续执行
	Semaphore:信号量，控制同时访问资源的线程数
	Exchanger:两个线程交换数据
并发集合：
	ConcurrentHashMap(线程安全，高并发)
	CopyOnWriteArrayList(读多写少场景，写时复制)
	BlockingQueue(阻塞队列，用于生产者-消费者模式)：
		ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue、PriorityBlockingQueue。
		实现put()/take()阻塞操作
```

### 5.ThreadLoacl

```
每个线程拥有自己的变量副本，避免共享
原理：每个线程维护一个ThreadLoaclMap（Entry的key是ThreadLoacl弱引用，value是值）
内存泄漏问题：key是弱引用，CG回收后会变成null,value无法访问；使用后应调用remove()
```

### 6.线程池（Execute框架）

```
七大核心参数：
	corePoolSize:核心线程数
	maximumPoolSize:最大线程数
	keepAliveTime:空闲线程存活时间（超过corePoolSize的线程）
	unit:时间单位
	workQueue:阻塞队列（ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue）
	threadFactory:创建线程的工厂
	handler：拒绝策略（AbortPolicy、CallerRunsPolicy、DiscardPolicy、DiscardOldestPolicy）
执行流程：
	1.提交任务，当前线程数<corePoolSize,创建新线程执行
	2.当前线程数>corePoolSize,任务放入阻塞队列
	3.队列已满，且当前线程数<maximumPoolSize ,创建新线程执行
	4.队列已满且线程数达到maximumPoolSize，执行拒绝策略
常用工厂方法（Executeors,但不推荐）
	newFixedThreadPool:固定线程数，不界队列（可能OOM）
	newCachedThreadPool:核心为0，最大为Integer.MAX_VALUE，同步队列（线程无限创建，风险高）
	newSimpleThreadPool:单线程，无界队列
	newScheduledThreePool:支持定时/周期任务
关闭线程池：
	shutdown():不再接受新任务，等待已有任务完成
	shutdownNow():尝试中断正在执行的任务，返回未执行的任务列表
问题：
	程池的拒绝策略有哪些？分别说明。
答：
	AbortPolicy（默认）：抛出 RejectedExecutionException。
	CallerRunsPolicy：由提交任务的线程自己执行该任务（如果线程池已关闭则丢弃）。
	DiscardPolicy：直接丢弃任务，不抛异常。
	DiscardOldestPolicy：丢弃队列头部的任务（最旧的），然后重新提交当前任务。
```

### 7.进阶主题

```
1. wait() / notify() / notifyAll()
	必须在 synchronized 块中调用（否则抛 IllegalMonitorStateException）。
	wait() 释放锁并进入 WAITING 状态，直到被 notify / notifyAll 唤醒或超时。
	notify() 随机唤醒一个等待线程，notifyAll() 唤醒所有。
2. Thread.sleep() vs wait()
	sleep 属于 Thread 类，不释放锁；wait 属于 Object，释放锁。
	sleep 时间到自动唤醒；wait 需要被 notify。
3. 死锁
	产生条件：互斥、持有并等待、不可剥夺、循环等待。
	排查：jstack 查看线程堆栈，分析锁占用关系。
4. 活锁、饥饿
	活锁：线程不断重试但都无法进展（如互相谦让）。
	饥饿：线程长时间获取不到所需资源。
5. Fork/Join 框架（Java 7）
	分治任务模型，RecursiveTask / RecursiveAction。
	工作窃取算法：空闲线程从其他队列尾部偷任务。
```

### 8.综合题目

```
题目：实现一个单例模式，要求线程安全且懒加载。写出两种方式并说明原理。
答案：

双重检查锁（DCL）：

java
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
原因：volatile 禁止指令重排序，避免半初始化对象被其他线程看到。
2. 静态内部类：

java
public class Singleton {
    private Singleton() {}
    private static class Holder { static Singleton instance = new Singleton(); }
    public static Singleton getInstance() { return Holder.instance; }
}
原理：类加载机制保证线程安全，且懒加载（内部类在使用时才加载）。

简答题：synchronized 和 ReentrantLock 的区别（至少 4 点）。
答案：

特性	synchronized	ReentrantLock
实现	JVM 内置，通过 monitor	API 级别，AQS
锁释放	自动（同步块结束）	手动 unlock()
可中断	不支持	支持 lockInterruptibly()
超时获取	不支持	tryLock(time, unit)
公平锁	非公平	可指定公平/非公平
条件变量	wait/notify（单一）	多个 Condition
性能	优化后（锁升级）接近 ReentrantLock	可调优
```

### 9.复习建议

```
代码实验：编写多线程代码验证 volatile 可见性、死锁、线程池行为。
对比记忆：sleep/wait、synchronized/Lock、CountDownLatch/CyclicBarrier。
源码阅读：ThreadPoolExecutor 的 execute 方法、ReentrantLock 的 lock 方法（AQS 队列）。
刷题：牛客网、LeetCode 上的多线程选择题，以及手写单例、生产者-消费者。
```



## 10. java 8+新特性

### 1.lambda表达式

```
语法：（参数）->{方法体}
函数式接口：只有一个抽象方法的接口（如 Runable,Comparator,Consumer），可用@FunctionalInterface注解
变量捕获：可以捕获final 或 effectively final的局部变量
```

### 2.函数式接口与内置函数式接口

```
Java.util.function包：
	Predicate<T>:test(T)->boolean
	Consumer<T>:accept(T)->void
	Function<T,R>:apply(T)->R
	Supplier<T>:get()->T
	UnaryOperator<T>、BinaryOperator<T>等
```

### 3.Stream API

```
流操作：filter、map、flatMap、disinct、sorted、limit、skip
终端操作：forEach、collect、reduce、count、anyMath/allmath/noneMath、findFirst/findAny
特性：惰性求值、流水线、内部迭代
并行流：parallelStream(),注意线程安全
```

### 4.方法引用和构造器引用

```
静态方法引用：ClassName::staticMethod
实例方法引用（特定对象）：instance:method
实例方法引用（任意对象）：ClassName::instanceMethod
构造器引用：ClassName::new
```

### 5.Optional类

```
用来避免NullPointerException
常用方法：ofNUllable、orElse、orElseGet、orElseThrow、map、filter
```

### 6.新的时间API

```
核心类：LocalDate、LoaclTime、LocalDateTime、ZonedDateTime、Instant
格式化：DateTimeFormatter(线程安全)
时间调整：TemporalAdjuster(如：firstDayOfMoth)
时间段：Duation(时间差)、Period(日期差)
```

### 7.接口增强

```
默认方法（default）:接口中可以有方法体，实现可继承和重写
静态方法（static）:接口中定义静态方法，通过接口名调用
```

### 8.其他java8特性

```
Nashorn引擎：在JVM中运行JavaScript(后续版本中移除)
Base64编译码器（java.util.Base64）
数组并行排序：Arrays.parallelSort()
CompletableFuture(对Future的增强，异步编程)
java.util.consurrent.ConcurrentHashMap增强：forEach、reduce、search等并行操作
```

### 9.题目

```
简答题：map 和 flatMap 在 Stream 中的区别？
答：map 将每个元素转换成另一个对象（一对一）；flatMap 将每个元素转换成多个对象（一对多），并扁平化成一个流。例如，将字符串列表拆分为单词列表。

代码题：使用 Stream 实现求整数列表中偶数的平方和。
list.stream().filter(n -> n % 2 == 0).mapToInt(n -> n * n).sum()
```

### 10.java9新特性

```
1.模块化系统（Project Jigsaw）
	引用module-info.java，定义模块导出、依赖等
	命令：java --module-path
2.集合工厂方法
	快速创建不可变集合：List.of(1,2,3)、Set.of("a","b")、Map.of("k","v")
	返回的集合不可变，元素不能为null
3.私有接口方法
	接口中允许private 方法，用于在默认方法间的共享代码
4.try-with-resources增强
	可以使用effectively final变量：try(resource)无需在try重新声明
5.stream增强
	dropWhile、takeWhile、ofNUllable、iterate重载（支持Predicate）
6.其他
	多版本兼容JAR(MRJAR)
	改进的Optional(ifPressentOrElse、or、stream()方法)
	Process API改进
```

### 11.Java 10特性

```
1.局部变量类型推断（var）
	使用var 声明局部变量，编译器自动推断类型
	不能用于字段、方法参数、返回类型
	示例：var list =new ArrayList<String>()
2.不可变集合增强
	List.copyOf、Set.copyOf、Map.copyOf
3.其他：
	并行全垃圾回收（G1的并行Full GC）
	Optional.orElseThrow()改为无参（java 10中已存在，JAVA 9是 orElseThrow(Supplier)）
```

### 12.Java 11(LTS)主要特性

```
1.局部变量类型推断用于Lambda
	Lambda参数可以使用 var 并添加注释 （@Nullable var x）-> x.toUpperCase().
2.字符串增强
	isBlack(),lines(),strip()(去除后前后空白，识别Unicode)、repeat(int)
3.文件读写便捷方法
	Files.readString(path)、Files.writeString(path,content)。
4.运行单文件程序
	java Hello.java直接运行（无需显示编译）
5.其他
	移除 JAVA EE和 CORBA模块
	HttpClient标准化（Java 9 引入，Java 11正式成为标准，支持 HTTP/2)
	新的垃圾回收器：ZGC(实验性)、Epsilon
```

### 13.java 12-16值得关注的特性

```
Java 12
	Switch 表达式（预览）：switch 可以作为表达式，使用->,不需要break。Java14正式引入
	Collectors.teeing(将流收集为两个结果后合并)
Java 13
	文本块（预览）："""..."""解决多行字符串拼接。java15 正式引入
	增强的switch表达式引入 yield
java 14
	记录类（Record,预览）：简洁的数据载体类，自动生成构造器、equals/hashcode/toString。Java 16正式引用
	instanceof 模糊匹配（预览）：if(obj instanceof String s),直接使用 s.java 16正式引用
Java 15
	文本块正式版
	密封类（Sealed Classes,预览）：限制子类范围。Java17正式引用
	ZGC 和 Shenandoah 变为产品特性（不再实验）
Java 16
	记录类（Record）正式版
	instanceof 模糊匹配正式版
	默认强封装JDK内部类（此前可通过 --illegal-access绕过）
```

### 14.Java 17（LTS)主要特性

```
密封类（Sealed Classes）正式版
	使用 sealed修饰类/接口，permits指定允许的子类
	子类可以是final、sealed 或 non-sealed
模糊匹配for switch(预览)
	switch 可以直接匹配类型、null、支持when子句
其他
	增强的伪随机数生成器（RandomGenerator接口）
	remove了实验性的AOT和JIT编译器
```

### 15.Java 18-21(最新LTS为21)特性

```
Java 18
	简单的Web服务器：jwebserver
	UTF-8作为默认字符集
	支持Javadoc中嵌入代码片段
Java 19
	虚拟线程（预览）：轻量级线程，提升并发处理能力。java 21正式版
	向量API(第四次孵化)
Java 20
	虚拟线程（第二次预览）、作用域值（孵化）
Java 21（LTS,2023年9月发布）
	虚拟线程（正式版）：Thread.ofVirtual().start(()->{...}),Executors.newVirtualthreadPerTaskExecutor().
	记录模式（Record Patterns）:解构记录对象
	模糊匹配for switch(正式版)
	有序集合（Sequenced Collections）:增强getFirst、getLast、reversed等方法
	字符串模板（预览）：STR."Hello \{name}"
	分代ZGC:提高ZGC性能
```

