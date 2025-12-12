# AOP 深度解析

## 1. 什么是 AOP？

**AOP（Aspect-Oriented Programming，面向切面编程）** 是一种编程范式，旨在通过分离**横切关注点**（Cross-cutting Concerns）来提高模块化程度。

简单来说，如果说 OOP（面向对象编程）是把系统看作一个个独立的“对象”自上而下地构建（纵向），那么 AOP 则是通过“切面”在多个对象之间横向切入，将那些与业务逻辑无关但又需要在多个地方重复执行的代码（如日志、安全、事务）抽取出来，统一管理。

### 核心目标

**将“业务逻辑”与“系统服务”解耦。**

## 2. 为什么要使用 AOP？（解决什么问题）

在传统的 OOP 开发中，我们经常遇到这样的情况：

假设你有一个 `UserService`，里面有 `addUser`、`deleteUser` 等方法。你需要做以下几件事：

1. **权限检查**：只有管理员能删人。
2. **日志记录**：记录“某人在某时删除了用户”。
3. **事务管理**：确保操作要么全成功，要么全回滚。
4. **核心业务**：真正的删除用户逻辑。

### 没有 AOP 时的代码 (Spaghetti Code)

```
public void deleteUser(Long id) {
    // 1. 权限检查 (重复代码)
    if (!SecurityContext.isAdmin()) {
        throw new SecurityException("无权操作");
    }

    // 2. 开启事务 (重复代码)
    Transaction tx = transactionManager.begin();

    try {
        // 3. 日志记录 (重复代码)
        Logger.log("开始删除用户: " + id);

        // --- 核心业务逻辑 (只有这一行是真正的业务) ---
        userDao.delete(id);
        // ---------------------------------------

        // 提交事务
        tx.commit();
    } catch (Exception e) {
        // 回滚事务
        tx.rollback();
        // 异常处理
        Logger.error("删除失败", e);
        throw e;
    }
}
```

**问题：**

- **代码冗余**：每个方法都要写一遍日志、事务、权限代码。
- **维护困难**：如果要修改日志格式，需要修改所有的方法。
- **逻辑混乱**：核心业务逻辑被大量的非业务代码掩盖。

## 3. AOP 的核心术语 (汉堡包模型)

理解 AOP 必须理解以下 5 个核心概念。我们可以用**“汉堡包”**做比喻：汉堡的肉饼是核心业务，面包和蔬菜是切面。

| **术语**      | **英文**      | **解释**                                                     | **比喻**                                   |
| ------------- | ------------- | ------------------------------------------------------------ | ------------------------------------------ |
| **切面**      | **Aspect**    | 关注点的模块化（比如“日志模块”或“事务模块”）。它包含了“通知”和“切入点”。 | 整个汉堡的面包片+生菜。                    |
| **连接点**    | **Joinpoint** | 程序执行过程中**可以**插入切面的点（比如方法调用前、方法调用后、抛出异常时）。 | 汉堡里所有**可以**放调料的地方。           |
| **切入点**    | **Pointcut**  | **真正**插入了切面的连接点。通过表达式定义（例如：只在 `delete*` 方法上生效）。 | 你决定**实际上**要把番茄酱挤在肉饼的上面。 |
| **通知/增强** | **Advice**    | 切面在特定切入点执行的动作（代码逻辑）。分为前置、后置、环绕等。 | 番茄酱的味道（具体的逻辑）。               |
| **织入**      | **Weaving**   | 将切面应用到目标对象并创建代理对象的过程。                   | 制作汉堡的过程（把面包盖在肉上）。         |
| **目标对象**  | **Target**    | 被代理的对象（核心业务类）。                                 | 汉堡中间的肉饼。                           |

## 4. 实战基础：切入点表达式 (Pointcut Cheatsheet)

在写代码前，必须知道“怎么拦”。`execution` 表达式是最常用的，但也最容易写错。

**语法：** `execution(修饰符? 返回值 包名.类名.方法名(参数))`

- **最通用（拦截 Service 包下所有方法）：** `execution(* com.example.service..*.*(..))`
  - 第一个 `*`：任意返回值
  - `..`：当前包及其子包
  - 第二个 `*`：任意类
  - 第三个 `*`：任意方法
  - `(..)`：任意参数
- **只拦截特定前缀（如查询方法）：** `execution(* com.example.service.*.find*(..))`
- **基于注解（推荐，最灵活）：** `@annotation(com.example.annotation.MyLog)`

## 5. 实战进阶：获取参数、返回值与控制流程

这是 AOP 最强大的地方。你可以通过 `JoinPoint` 和 `ProceedingJoinPoint` 对象来获取上下文信息。

### 5.1 获取方法名和参数 (`JoinPoint`)

适用于 `@Before`, `@After` 等通知。

```
@Before("execution(* com.example.service.UserService.*(..))")
public void logArgs(JoinPoint joinPoint) {
    // 1. 获取方法签名（方法名）
    String methodName = joinPoint.getSignature().getName();

    // 2. 获取参数列表（数组）
    Object[] args = joinPoint.getArgs();

    System.out.println("正在执行方法: " + methodName);
    System.out.println("参数是: " + Arrays.toString(args));
}
```

### 5.2 获取返回值 (`@AfterReturning`)

如果你想记录方法的返回值，需要使用 `returning` 属性。

```
// returning 的值 "result" 必须和方法参数名一致
@AfterReturning(pointcut = "execution(* com.example.service.UserService.*(..))", returning = "result")
public void logResult(Object result) {
    System.out.println("方法执行成功，返回值为: " + result);
    // 注意：在这里你只能读取返回值，不能修改它。
}
```

### 5.3 环绕通知：完全控制 (`@Around`)

这是**最强大**的通知。它可以决定是否执行目标方法，或者**修改参数**，或者**篡改返回值**。

```
@Around("execution(* com.example.service.UserService.getUser(..))")
public Object auditMethod(ProceedingJoinPoint pjp) throws Throwable {
    Object[] args = pjp.getArgs();
    String methodName = pjp.getSignature().getName();

    // 【修改参数】：如果第一个参数是 String，把它改成 "ModifiedName"
    if (args.length > 0 && args[0] instanceof String) {
        args[0] = "ModifiedName";
    }

    Object result = null;
    try {
        // 【关键】：执行目标方法。如果不调这行，目标方法就被拦截了！
        result = pjp.proceed(args); // 传入修改后的参数
        
        // 【篡改返回值】：
        if (result instanceof String) {
            result = result + " (被AOP修改了)";
        }
    } catch (Throwable e) {
        System.out.println("方法报错了: " + e.getMessage());
        throw e; // 继续抛出，让外层感知异常
    }
    return result;
}
```

### 5.4 生产级实战：基于注解的 AOP

在实际项目中，我们通常不会用复杂的 `execution` 表达式，而是自定义注解。

**第一步：定义注解**

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RecordLog {
    String value() default ""; // 允许传入操作描述
}
```

**第二步：编写切面**

```
@Aspect
@Component
public class AnnotationAspect {

    // 直接拦截带有 @RecordLog 注解的方法
    @Around("@annotation(recordLog)") 
    public Object logWithAnnotation(ProceedingJoinPoint pjp, RecordLog recordLog) throws Throwable {
        // 1. 获取注解上的值
        String action = recordLog.value();
        System.out.println("正在执行操作: " + action);

        // 2. 执行目标方法
        return pjp.proceed();
    }
}
```

**第三步：使用**

```
@RecordLog("删除用户")
public void deleteUser(Long id) { ... }
```

### 5.5 通用参数处理 (反射实战)

当你无法预知参数类型，但需要对所有对象做统一处理（如：统一去空格、统一填充 `updateTime`）时，可以使用 **反射**。

推荐使用 Spring 的 `ReflectionUtils`，它比原生反射更安全、简便。

```
@Around("execution(* com.example.service..*.*(..))")
public Object genericHandle(ProceedingJoinPoint pjp) throws Throwable {
    Object[] args = pjp.getArgs();

    for (Object arg : args) {
        if (arg == null) continue;

        // 场景1：如果对象有 "operator" 字段，自动填入当前用户
        Field operatorField = ReflectionUtils.findField(arg.getClass(), "operator");
        if (operatorField != null) {
            ReflectionUtils.makeAccessible(operatorField); // 突破 private 限制
            ReflectionUtils.setField(operatorField, arg, "SystemAdmin");
        }

        // 场景2：如果对象是 String 类型，自动去除首尾空格
        // 使用 doWithFields 遍历所有字段
        ReflectionUtils.doWithFields(arg.getClass(), field -> {
            ReflectionUtils.makeAccessible(field);
            Object value = field.get(arg);
            if (value instanceof String) {
                field.set(arg, ((String) value).trim());
            }
        });
    }

    return pjp.proceed(args);
}
```

### 5.6 性能优化建议：公共接口 (BaseEntity)

虽然反射很灵活，但如果这些参数对象都是你定义的 DTO，建议让它们实现一个公共接口（例如 `BaseEntity`）。这样你可以直接强转为接口类型来操作，而**不需要用反射**，性能会更好，代码也更安全。

**1. 定义接口**

```
public interface BaseEntity {
    void setUpdateTime(Date date);
    void setOperator(String operator);
}
```

**2. 优化后的切面代码**

```
@Around("execution(* com.example.service..*.*(..))")
public Object interfaceHandle(ProceedingJoinPoint pjp) throws Throwable {
    Object[] args = pjp.getArgs();

    for (Object arg : args) {
        // 直接判断是否实现了接口，无需反射，性能极高
        if (arg instanceof BaseEntity) {
            BaseEntity entity = (BaseEntity) arg;
            entity.setUpdateTime(new Date());
            entity.setOperator("SystemAdmin");
        }
    }

    return pjp.proceed(args);
}
```

## 6. AOP 避坑指南：自调用失效 (Self-Invocation)

这是 90% 的开发者都会踩的坑。

### 现象

在同一个类内部，方法 A 调用方法 B，方法 B 上的切面（如 `@Transactional` 或 AOP 日志）**不会生效**。

```
@Service
public class UserService {
    
    public void methodA() {
        // 直接调用内部方法
        this.methodB(); // <--- 这里的切面会失效！
    }

    @Transactional // 或者 @RecordLog
    public void methodB() {
        System.out.println("执行 methodB");
    }
}
```

### 原因

Spring AOP 是基于**代理 (Proxy)** 的。

- 当外部调用 `userService.methodA()` 时，调用的是代理对象。
- 但在 `methodA` 内部调用 `this.methodB()` 时，`this` 指向的是**目标对象本身**，而不是代理对象。
- **绕过了代理，就绕过了切面。**

### 解决方案

1. **自我注入 (Self-Injection)**：

   ```
   @Service
   public class UserService {
       @Autowired
       private UserService self; // 注入代理后的自己
   
       public void methodA() {
           self.methodB(); // <--- 走代理，切面生效！
       }
   }
   ```

2. **避免内部调用**：将 `methodB` 抽取到另一个 Service 中（最推荐）。

## 7. 总结

- **OOP** 封装业务，**AOP** 封装服务。
- **常用场景**：日志、事务、权限、异常处理。
- **核心技巧**：掌握 `@Around` 环绕通知、基于注解的配置以及 `ReflectionUtils` 通用处理。
- **注意陷阱**：内部调用 (`this.method()`) 会导致 AOP 失效，务必注意。