# Java 自定义注解 + AOP 

------

## 一、什么是自定义注解？

**注解（Annotation）** 是 Java 在 JDK1.5 引入的功能，用来给代码增加“元数据”。
 自定义注解允许开发者按照自己的业务需求定义新注解，让 AOP、反射等机制自动解析并执行对应逻辑。

常见用途：

- 日志记录（@Log）
- 权限校验（@CheckAuth）
- 接口限流（@RateLimit）
- 参数校验（@NotNull）
- 缓存（@Cacheable）
- 事务控制（@Transactional）

------

## 二、定义一个自定义注解的关键元素

一个注解需要关注三点：

| 元注解        | 作用                                           |
| ------------- | ---------------------------------------------- |
| `@Target`     | 该注解可以放在什么位置（方法、类、字段等）     |
| `@Retention`  | 注解在何种生命周期可见（源码 → 编译 → 运行时） |
| `@Documented` | 是否生成到 Javadoc 中                          |
| `@Inherited`  | 子类是否继承父类注解                           |

建议：
 **AOP 使用注解，Retention 必须是 RUNTIME**，否则无法在运行时获取。

------

## 三、自定义注解示例

需求：编写一个注解，用来标记需要记录日志的方法。

### ✔️ 注解代码 @MyLog

```
package com.example.annotation;

import java.lang.annotation.*;

@Target(ElementType.METHOD) // 用于方法
@Retention(RetentionPolicy.RUNTIME) // 运行时可见
@Documented
public @interface MyLog {
    String value() default ""; // 可选字段，用于描述
}
```

------

## 四、如何使用注解

在方法上使用：

```
@MyLog("添加用户操作")
public void addUser(User user) {
    // 业务逻辑
}
```

------

## 五、结合 AOP 解析注解

### ✔️ AOP 切面示例

```
package com.example.aspect;

import com.example.annotation.MyLog;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Slf4j
@Aspect
@Component
public class LogAspect {

    // 切入点：扫描所有包含 @MyLog 的方法
    @Pointcut("@annotation(com.example.annotation.MyLog)")
    public void logPointcut() {}

    // 前置通知
    @Before("logPointcut() && @annotation(myLog)")
    public void before(JoinPoint joinPoint, MyLog myLog) {
        String methodName = joinPoint.getSignature().getName();
        log.info("执行方法：{}，描述：{}", methodName, myLog.value());
    }

    // 后置返回
    @AfterReturning(value = "logPointcut()", returning = "result")
    public void afterReturning(Object result) {
        log.info("方法执行结束，返回：{}", result);
    }

    // 异常通知
    @AfterThrowing(value = "logPointcut()", throwing = "ex")
    public void afterThrowing(Exception ex) {
        log.error("方法异常：{}", ex.getMessage());
    }
}
```

------

## 六、自定义注解常用属性

| 类型         | 用法                 |
| ------------ | -------------------- |
| `String`     | 描述信息             |
| `int / long` | 限流次数、超时时间等 |
| `boolean`    | 是否启用某个特性     |
| `Class<?>`   | 绑定特定类型         |
| 枚举类型     | 选择不同策略         |

示例：

```
import java.lang.annotation.*;

@Target(ElementType.METHOD) // 作用在方法上
@Retention(RetentionPolicy.RUNTIME) // 运行时生效
@Documented
public @interface RateLimit {

    int maxCount() default 5;          // 最大访问次数，默认 5 次
    long timeout() default 60000L;     // 超时时间（毫秒），默认 1 分钟
    boolean enable() default true;     // 是否启用限流，默认开启
}
```

```
@RestController
public class TestController {

    @RateLimit(maxCount = 3, timeout = 10000L) // 3 次 / 10 秒
    @GetMapping("/test")
    public String testRateLimit() {
        return "请求成功";
    }
}

```

------

## 七、自定义注解最佳实践

### 1. Retention 必须为 RUNTIME

否则 AOP、反射无法读取。

### 2. Target 设置准确

常用：

- `METHOD` → 方法级增强
- `TYPE` → 类级增强
- `FIELD` → 字段注入（如 @Autowired）

### 3. 尽量提供默认值

避免每次都必须写一堆参数。

### 4. 注解字段尽量简单

避免嵌套复杂对象。

------

## 八、自定义注解常见的应用场景模板

你可以直接复制使用：

### ✔️ 权限校验注解

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface CheckAuth {
    String role();
}
```

### ✔️ 接口限流注解

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RateLimit {
    int maxCount();
    long windowTime() default 1000;
}
```

### ✔️ 参数校验注解

```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface NotNull {
    String message() default "字段不能为空";
}
```

------

## 九、如何在 AOP 中读取注解信息？

简单三步：

```
Method method = ((MethodSignature) joinPoint.getSignature()).getMethod();
MyLog myLog = method.getAnnotation(MyLog.class);
String desc = myLog.value();
```

------

## 十、 总结：自定义注解 + AOP 的完整流程

1. **定义注解**（声明 @Target、@Retention）
2. **在业务方法上使用注解**
3. **编写 AOP 切面类**
   - 定义切点：`@annotation(...)`
   - 获取注解内容
   - 编写增强逻辑（日志、权限、限流等）
4. **启动 Spring AOP**
5. **运行时 Spring 自动拦截并执行增强逻辑**