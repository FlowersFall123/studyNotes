#  Java 方向班笔记

## 一、环境变量配置

| 名称        | 作用                               | 示例                                          |
| ----------- | ---------------------------------- | --------------------------------------------- |
| `JAVA_HOME` | 指向 JDK 安装目录                  | `C:\Program Files\Java\jdk21`                 |
| `CLASSPATH` | 指定类加载路径                     | `.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar` |
| `Path`      | 使系统可以在任意目录运行 Java 命令 | `%JAVA_HOME%\bin`                             |

------

## 二、标识符命名规则

1. 不能以数字开头
2. 不能是关键字
3. 区分大小写
4. 只能由字母、数字、下划线 `_`、美元符 `$` 构成

✅ **示例：**

```
int studentAge;
String _name;
double $salary;
```

------

## 三、分隔符

空格、逗号 `,`、分号 `;`、冒号 `:`、大括号 `{}`、小括号 `()`、引号 `"" ''`、箭头 `->`、尖括号 `< >`

------

## 四、数据类型

### 1. 基本数据类型（Primitive）

| 类型    | 位数 | 默认值   | 示例                   |
| ------- | ---- | -------- | ---------------------- |
| byte    | 8位  | 0        | `byte a = 10;`         |
| short   | 16位 | 0        | `short s = 1000;`      |
| int     | 32位 | 0        | `int i = 10;`          |
| long    | 64位 | 0L       | `long l = 10000L;`     |
| float   | 32位 | 0.0f     | `float f = 3.14f;`     |
| double  | 64位 | 0.0      | `double d = 3.14;`     |
| boolean | 1位  | false    | `boolean flag = true;` |
| char    | 16位 | '\u0000' | `char c = 'A';`        |

### 2. 引用数据类型（Reference）

包括：**数组、包装类、类对象、接口对象、枚举类型**

**区别：**

1. 基本类型不是对象，只能调用 `.class`；包装类有丰富的方法。
2. 数组中默认值不同（如 `int[]` 默认 0，`String[]` 默认 `null`）。
3. 引用类型的本质是类。

------

## 五、类型转换

### 1. 向上转型（自动类型提升）

```
int a = 10;
double b = a; // 自动转换 int → double
```

### 2. 向下转型（强制类型转换，可能丢失精度）

```
double d = 9.8;
int i = (int) d; // 结果为9
```

------

## 六、装箱与拆箱

### 1. 装箱（基本 → 包装类）

```
int a = 10;
Integer b = Integer.valueOf(a); // 或直接 b = a;
```

### 2. 拆箱（包装类 → 基本）

```
Integer x = 20;
int y = x.intValue(); // 或直接 y = x;
```

------

## 七、比较方式

| 比较方式       | 作用                            | 示例                      |
| -------------- | ------------------------------- | ------------------------- |
| `==`           | 比较 **地址**（基本类型比较值） | `a == b`                  |
| `.equals()`    | 比较 **内容值**                 | `"abc".equals("abc")`     |
| `.compareTo()` | 比较大小/顺序                   | `"a".compareTo("b") → -1` |

------

## 八、变量分类

| 类型                     | 定义位置               | 生命周期         |
| ------------------------ | ---------------------- | ---------------- |
| **局部变量**             | 方法内、参数、代码块中 | 方法执行完即消失 |
| **全局变量（成员变量）** | 类中、方法外           | 随对象存在而存在 |

------

## 九、语法结构

| 结构       | 写法示例                    |
| ---------- | --------------------------- |
| 类         | `public class ClassName {}` |
| 属性       | `private int age;`          |
| 方法       | `public void show() {}`     |
| 静态代码块 | `static {}`                 |
| 普通代码块 | `{}`                        |

------

## 十、运算符分类

| 类型       | 符号               |
| ---------- | ------------------ |
| 算术运算符 | `+ - * / %`        |
| 逻辑运算符 | `&&                |
| 关系运算符 | `> < >= <= == !=`  |
| 位运算符   | `~ >> <<`          |
| 三目运算符 | `条件 ? 值1 : 值2` |
| 赋值运算符 | `=`                |
| 一元运算符 | `+ - ++ --`        |
| 小括号     | `()`（优先级最高） |

------

## 十一、访问修饰符作用域

| 修饰符           | 同类 | 同包 | 子类 | 其他包 |
| ---------------- | ---- | ---- | ---- | ------ |
| `private`        | ✅    | ❌    | ❌    | ❌      |
| （默认 default） | ✅    | ✅    | ❌    | ❌      |
| `protected`      | ✅    | ✅    | ✅    | ❌      |
| `public`         | ✅    | ✅    | ✅    | ✅      |

------

## 十二、程序执行结构

1. **顺序结构**：从上到下依次执行
2. **分支结构**：`if`、`switch`
3. **循环结构**：`for`、`while`、`do...while`

------

## 十三、常用关键字

| 关键字     | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| `static`   | 静态修饰符，属于类本身，可直接通过类名调用。最先执行。       |
| `final`    | 修饰类：不可被继承；修饰变量：不可修改；修饰方法：不可重写。 |
| `this`     | 当前类的实例引用。                                           |
| `super`    | 指向父类对象；构造函数中必须放在第一行。                     |
| `break`    | 跳出循环。                                                   |
| `continue` | 跳过本次循环进入下一次。                                     |
| `return`   | 结束当前方法；即使存在 `finally`，`finally` 仍会执行。       |

------

## 十四、注释与注解

### 1. 注释（Comment）

| 类型     | 写法          |
| -------- | ------------- |
| 文档注释 | `/** 内容 */` |
| 多行注释 | `/* 内容 */`  |
| 单行注释 | `// 内容`     |

### 2. 注解（Annotation）

以 `@` 开头，用于类、方法、属性等。
 常见注解：`@Override`、`@Deprecated`、`@SuppressWarnings`

## 十五、面向对象（OOP）

### 1. 类与对象

| 概念               | 说明                     |
| ------------------ | ------------------------ |
| **类（Class）**    | 对象的集合，是抽象的模板 |
| **对象（Object）** | 类的一个具体实例         |
| **关键字**         | `class`                  |
| **特征**           | 属性（成员变量）         |
| **行为**           | 方法（函数）             |

```
class Person {
    String name;
    int age;

    void sayHello() {
        System.out.println("Hello, my name is " + name);
    }
}

public class Test {
    public static void main(String[] args) {
        Person p = new Person();
        p.name = "Tom";
        p.age = 18;
        p.sayHello();
    }
}
```

------

### 2. 继承（extends）

- 使用关键字 `extends`。
- Java 中是 **单继承**。
- 不能继承：
  - 父类的 `private` 成员
  - 构造方法
  - 不同包下的 `default` 成员
- 子类可以继承父类的 `public`、`protected` 成员。

```
class Animal {
    public void eat() { System.out.println("Animal eat"); }
}

class Dog extends Animal {
    public void bark() { System.out.println("Dog bark"); }
}

public class Test {
    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.eat();
        dog.bark();
    }
}
```

------

### 3. 封装（Encapsulation）

- 使用 `private` 修饰属性
- 通过 `public` 的 get/set 方法来访问属性

```
class Student {
    private String name;

    public void setName(String n) {
        this.name = n;
    }
    public String getName() {
        return name;
    }
}
```

------

### 4. 多态（Polymorphism）

- **重写（Override）**：方法名、参数列表相同，方法体不同（子类重写父类）。
- **重载（Overload）**：方法名相同，参数列表不同（参数个数或类型不同）。

```
class Animal {
    public void speak() { System.out.println("Animal sound"); }
}

class Dog extends Animal {
    @Override
    public void speak() { System.out.println("Dog bark"); }
}

class MathTool {
    int add(int a, int b){ return a + b; }
    double add(double a, double b){ return a + b; }
}
```

------

## 十六、数组（Array）

1. **声明**

```
int[] arr1;
int arr2[];
```

1. **数组长度**：固定
2. **初始化**

```
int[] arr = new int[5];
int[] arr2 = {1, 2, 3, 4};
```

1. **赋值**

```
arr[0] = 100;
```

1. **长度属性**

```
System.out.println(arr.length);
```

1. **索引最大值**
    `arr.length - 1`，索引从 0 开始
2. **排序**（使用 `Arrays.sort`）

```
import java.util.Arrays;
int[] nums = {3, 1, 4};
Arrays.sort(nums);
```

引用类型排序需实现 `Comparable` 接口。

------

## 十七、集合框架（Collection Framework）

### 1. Collection

| 类型     | 特性                 | 常见实现类                |
| -------- | -------------------- | ------------------------- |
| **List** | 有序、可重复、可索引 | `ArrayList`, `LinkedList` |
| **Set**  | 无序、不可重复       | `HashSet`, `TreeSet`      |

常用方法：`add()`、`remove()`、`contains()`、`iterator()`、`size()`、`clear()`、`isEmpty()`

### 2. Map

| 特性         | 示例                                          |
| ------------ | --------------------------------------------- |
| 键值对存储   | `Map<String, Integer> map = new HashMap<>();` |
| 键不可重复   | 重复键会覆盖旧值                              |
| 可存 null 键 | 仅一个 null 键（HashMap）                     |

```
Map<String, Integer> map = new HashMap<>();
map.put("Tom", 18);
map.put("Jerry", 20);
```

### 3. Comparator 比较器

- `Comparable`：类内部定义排序规则
- `Comparator`：类外定义排序规则

------

## 十八、枚举（Enum）

### 1. 枚举定义与使用

```
enum Season {
    SPRING, SUMMER, AUTUMN, WINTER
}

public class Test {
    public static void main(String[] args) {
        Season s = Season.SPRING;
        System.out.println(s);
    }
}
```

- 枚举默认继承 `Enum` 类
- 可定义属性和方法
- 可用于表示固定常量（如状态、方向、星期）

------

## 十九、常用工具类

### 1. `Math`

```
System.out.println(Math.abs(-5));  // 绝对值
System.out.println(Math.pow(2, 3)); // 幂
System.out.println(Math.sqrt(16));  // 开方
```

### 2. `Random`

```
Random r = new Random();
int num = r.nextInt(100);  // 0-99随机数
```

### 3. `Date` & `Calendar`

```
Date date = new Date();
System.out.println(date);

Calendar c = Calendar.getInstance();
System.out.println(c.get(Calendar.YEAR));
```

------

## 二十、字符串处理

### `StringBuffer` 与 `StringBuilder`

- **StringBuffer**：线程安全（适合多线程）
- **StringBuilder**：非线程安全（速度更快）

```
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");
System.out.println(sb.toString());
```

------

## 二十一、异常处理

### 1. 异常分类

- `Throwable`
  - `Error`（系统错误，不可恢复）
  - `Exception`（程序异常，可处理）

### 2. 捕获异常

```
try {
    int x = 10 / 0;
} catch (Exception e) {
    System.out.println("异常：" + e.getMessage());
} finally {
    System.out.println("不管是否异常都会执行");
}
```

### 3. 自定义异常

```
class MyException extends Exception {
    public MyException(String msg) {
        super(msg);
    }
}

public class Test {
    public static void main(String[] args) throws MyException {
        throw new MyException("自定义异常");
    }
}
```

------

## 二十二、泛型（Generics）

| 符号 | 含义            |
| ---- | --------------- |
| `E`  | Element（元素） |
| `T`  | Type（类型）    |
| `K`  | Key（键）       |
| `V`  | Value（值）     |
| `?`  | 通配符          |

```
List<String> list = new ArrayList<>();
```

### 通配符：

- `<? extends superclass>`：上限通配，表示某个类的子类
- `<? super subclass>`：下限通配，表示某个类的父类

------

## 二十三、多线程

### 1. 线程生命周期

- 创建 → 就绪 → 运行 → 阻塞 → 死亡

### 2. 创建方式

```
// 继承Thread
class MyThread extends Thread {
    public void run() {
        System.out.println("线程运行");
    }
}

// 实现Runnable
class MyRun implements Runnable {
    public void run() {
        System.out.println("线程运行");
    }
}
```

### 3. 死锁四条件

- 互斥
- 占有且申请
- 不可抢占
- 循环等待

### 4. 守护线程

```
Thread t = new Thread();
t.setDaemon(true);  // 设置为守护线程
```

------

## 二十四、锁

- **作用**：保证多线程访问共享资源时的线程安全
- **常用方式**：
  - `synchronized` 关键字
  - `ReentrantLock` 可重入锁

```
synchronized (this) {
    // 同步代码块
}
```

------

## 二十五、IO 流

### 1.分类

| 分类   | 说明           | 示例                          |
| ------ | -------------- | ----------------------------- |
| 字节流 | 处理二进制数据 | `InputStream`, `OutputStream` |
| 字符流 | 处理文本数据   | `Reader`, `Writer`            |

| 方向   | 示例               |
| ------ | ------------------ |
| 输入流 | `FileInputStream`  |
| 输出流 | `FileOutputStream` |

### 2. 读取与写入

```
FileInputStream in = new FileInputStream("a.txt");
FileOutputStream out = new FileOutputStream("b.txt");
int c;
while ((c = in.read()) != -1) {
    out.write(c);
}
in.close();
out.close();
```

### 3. 序列化与反序列化

```
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("data.obj"));
oos.writeObject(obj);

ObjectInputStream ois = new ObjectInputStream(new FileInputStream("data.obj"));
Object o = ois.readObject();
```

- 类必须实现 `Serializable` 接口

------

## 二十六、接口与抽象类

### 1. 接口（interface）

- 使用 `interface` 声明
- 使用 `implements` 实现
- 只能包含 `public static final` 常量和 `abstract` 方法（JDK 8 以后可包含默认方法和静态方法）
- **支持多实现**

```
interface Flyable {
    void fly();
}

class Bird implements Flyable {
    public void fly() {
        System.out.println("Bird is flying");
    }
}
```

------

### 2. 抽象类（abstract class）

- 使用 `abstract class` 声明
- 使用 `extends` 继承
- 可以包含抽象方法和具体方法
- 只能单继承

```
abstract class Animal {
    abstract void makeSound();
    void eat() { System.out.println("Animal eats"); }
}

class Dog extends Animal {
    void makeSound() { System.out.println("Dog barks"); }
}
```