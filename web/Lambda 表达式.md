# Lambda 表达式



## 🔹 Lambda 表达式结构

```
(参数) -> { 方法体 }
```

- **左边**：参数（输入）
- **右边**：方法体（要执行的代码 / 返回值）

| 问题                              | 出现原因                                                     | 解决办法 / 建议                                              |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Lambda 无法实现接口里多个方法** | Lambda 只能实现 **函数式接口**，即接口中 **只能有一个抽象方法**。如果接口有多个抽象方法，编译器无法推断你想实现哪一个方法 → 报错。 | 1. 拆接口：把每个方法放在独立的函数式接口里 2. 使用默认方法（`default`），Lambda 实现唯一抽象方法，其余方法有默认实现 |
| **Lambda + 大括号没有返回值报错** | 当 Lambda 使用 `{}` 包裹多语句时，Java 需要显式写 `return` 才能返回值。否则编译器认为是语句块，没有返回值 → 报错。 | 1. 单表达式：`(a,b) -> a + b`，自动返回 2. 多语句或大括号：`(a,b) -> { return a + b; }` |

## 🔹 一、Lambda 表达式的基本语法

```
(parameters) -> expression
(parameters) -> { statements; }
```

特点：

1. 省略了匿名类的冗余代码。
2. 可以推断参数类型，有时参数的小括号也能省略。
3. 如果只有一行语句，花括号和 `return` 都可以省略。

------

## 🔹 二、常见使用场景

### 1. **替代匿名内部类**

最常见的是简化函数式接口（只有一个抽象方法的接口，如 Runnable、Comparator 等）的写法。

```
// 传统写法
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello, Lambda!");
    }
}).start();

// Lambda 写法
new Thread(() -> System.out.println("Hello, Lambda!")).start();
```

------

### 2. **集合遍历 (forEach)**

```
List<String> list = Arrays.asList("Java", "Python", "C++");

// 普通 for-each
for (String s : list) {
    System.out.println(s);
}

// Lambda + forEach
list.forEach(s -> System.out.println(s));

// 方法引用（更简洁）
list.forEach(System.out::println);
```

------

### 3. **排序 (Comparator)**

```
List<String> list = Arrays.asList("java", "python", "c", "scala");

// 匿名内部类
Collections.sort(list, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
});

// Lambda 写法
Collections.sort(list, (a, b) -> a.compareTo(b));

// 更简洁（方法引用）
list.sort(String::compareTo);
```

------

### 4. **集合操作 (Stream API)**

Lambda 与 **Stream** 一起用非常强大。

```
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5);

// 过滤、映射、收集
List<Integer> result = nums.stream()
        .filter(n -> n % 2 == 0)     // 过滤偶数
        .map(n -> n * n)             // 平方
        .toList();                   // 收集为 List

System.out.println(result); // [4, 16]
```

------

### 5. **事件监听（GUI/回调）**

```
button.addActionListener(e -> System.out.println("Button clicked!"));
```

------

### 6. **自定义函数式接口**

```
@FunctionalInterface
interface MathOperation {
    int operate(int a, int b);
}

// 使用 Lambda
MathOperation add = (a, b) -> a + b;
MathOperation mul = (a, b) -> a * b;

System.out.println(add.operate(3, 5)); // 8
System.out.println(mul.operate(3, 5)); // 15
```

------

## 🔹 三、总结

Lambda 表达式在 Java 中的常见用法主要有：

1. 替代匿名内部类（Runnable、Comparator 等）。
2. 集合遍历（forEach）。
3. 排序（Comparator）。
4. 搭配 Stream 进行集合操作（过滤、映射、聚合）。
5. GUI 或回调事件监听。
6. 自定义函数式接口实现。

👉 核心就是 **简化代码 + 函数式编程**，最常见的就是在集合和流处理里用。