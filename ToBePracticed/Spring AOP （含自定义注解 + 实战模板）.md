# **Spring AOP （含自定义注解 + 实战模板）**

------

## 1️⃣ 什么是 AOP？

AOP（Aspect Oriented Programming，面向切面编程）是一种：

> 在不修改业务代码的情况下，为方法统一添加额外逻辑的编程思想。

它可以在方法 **执行前 / 执行后 / 执行异常时** 自动插入代码，用于处理系统级的通用功能。

------

## 2️⃣ 为什么要使用 AOP？

AOP 专门解决项目中的“横切关注点”，即多处重复出现的系统级逻辑。

### ✔ 1. 去除重复代码

不需要在每个方法里都写：

- 日志打印
- 参数校验
- 权限校验
- 异常捕获
- 统计接口耗时
- 缓存逻辑
- 防重复提交
- 限流

这些全部放到 AOP，一个地方统一管理。

### ✔ 2. 动态、可插拔的系统功能

无需修改业务代码即可 “加功能”，非常灵活。

------

## 3️⃣ AOP 能做什么？

几乎所有与方法执行相关的系统功能，都可以用 AOP 实现：

| 功能         | AOP 是否适合    |
| ------------ | --------------- |
| 接口日志     | ✔ 最常用        |
| 接口耗时统计 | ✔               |
| 权限校验     | ✔               |
| 缓存逻辑     | ✔（@Cacheable） |
| 数据脱敏     | ✔               |
| 统一异常监控 | ✔               |
| 限流         | ✔               |
| 防重复提交   | ✔               |
| 参数校验     | ✔               |

------

## 4️⃣ AOP 底层原理

Spring AOP 使用“动态代理”在方法的前后插入你写的额外逻辑。

- **类有接口** → 使用 **JDK 动态代理**
- **类没有接口** → 使用 **CGLIB 生成子类代理**

你写了切面后，Spring 自动把你的 Bean 换成代理对象，你感知不到。

------

## 5️⃣ AOP 核心概念

| 名称             | 含义                   | 示例                        |
| ---------------- | ---------------------- | --------------------------- |
| 切面 Aspect      | 横切关注点功能模块     | 日志切面、权限切面          |
| 连接点 JoinPoint | 可以被 AOP 拦截的位置  | 方法执行                    |
| 切点 Pointcut    | 具体要拦截哪些方法     | execution(* com.xxx..*(..)) |
| 通知 Advice      | 被织入的逻辑           | @Before、@Around            |
| 织入 Weaving     | 把切面逻辑插入目标方法 | 代理类调用                  |

------

## 6️⃣ AOP 的通知类型

| 注解            | 执行时机               | 场景               |
| --------------- | ---------------------- | ------------------ |
| @Before         | 方法执行前             | 权限校验、参数日志 |
| @After          | 方法执行后（不管异常） | 清理资源           |
| @AfterReturning | 返回值后               | 记录返回值         |
| @AfterThrowing  | 抛异常时               | 异常监控           |
| **@Around**     | 包裹整个方法（最强）   | 日志、耗时、缓存   |

------

## 7️⃣ Spring 常用切点表达式

拦截某包下所有方法：

```
execution(* com.example.service..*(..))
```

含义：

- `com.example.service` 包
- 所有子包
- 所有方法
- 所有参数

------

## 8️⃣ 实战：基于注解的企业级 AOP 模板

这是企业开发中最常用的 AOP 模板结构：

- @MyLog：用于记录日志
- AOP 切面：统一输出入参、返回值、耗时

下面的代码可以直接复制到你的项目即可运行。

------

### ✨ 8.1 自定义注解 @Log

[自定义注解笔记](https://github.com/FlowersFall123/studyNotes/blob/main/ToBePracticed/Java%20%E8%87%AA%E5%AE%9A%E4%B9%89%E6%B3%A8%E8%A7%A3%20%2B%20AOP.md)

📁 `com.signBridge.common.tool.MyLog`

```
package com.xxx.annotation;

import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface MyLog {
    String value() default "";  // 可填写业务说明
}
```

------

### ✨ 8.2 日志切面 LogAspect

📁 `com.xxx.aop.LogAspect.java`

```
package com.signBridge.common.tool;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.aspectj.lang.reflect.MethodSignature;

import java.lang.reflect.Method;

@Aspect
@Component//微服务不需要，因为会在MvcConfig里面注册，避免重复注册
public class LogAspect {
    private final ObjectMapper mapper = new ObjectMapper();

    // 匹配所有加了 @MyLog 的方法
    @Pointcut("@annotation(com.signBridge.common.tool.MyLog)")
    public void logPointcut() {}

    @Around("logPointcut()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {

        long start = System.currentTimeMillis();

        Method method = ((MethodSignature) pjp.getSignature()).getMethod();
        MyLog logAnno = method.getAnnotation(MyLog.class);

        String methodName = method.getName();
        String params = mapper.writeValueAsString(pjp.getArgs());

        System.out.println("========== AOP 日志开始 ==========");
        System.out.println("业务描述: " + logAnno.value());
        System.out.println("方法: " + methodName);
        System.out.println("入参: " + params);

        Object result = pjp.proceed();

        String resultJson = mapper.writeValueAsString(result);

        long cost = System.currentTimeMillis() - start;

        System.out.println("返回值: " + resultJson);
        System.out.println("耗时: " + cost + " ms");
        System.out.println("========== AOP 日志结束 ==========");

        return result;
    }
}
```

------

### ✨ 8.3 在业务方法上使用 @MyLog

📁 `UserService.java`

```
@Service
public class UserService {

    @MyLog("查询用户信息")
    public String getUser(String id) {
        System.out.println("【业务方法执行】查询：" + id);
        return "User-" + id;
    }
}
```

只需加一个注解，日志自动记录。

**补充：微服务的话需要注册到对应的公共模块的MvcConfig配置里面(自动装配)**

```
/**
 * 注册全局 LogAspect 切面
 * @return
 */
@Bean
public LogAspect logAspect() {
    log.info("----------- 注册全局 LogAspect 切面 -----------");
    return new LogAspect();
}
```

------

## 9️⃣ 为什么推荐用“注解 + AOP”？

优势非常明显：

### ✔ 业务代码更干净

不用写各种日志、校验、try-catch。

### ✔ 灵活可扩展

想给一个方法加日志 → 加一个注解即可。

### ✔ 统一规范

所有日志格式一致，便于排查问题。

------

## 🔟 AOP 使用注意事项（面试高频）

### ❌ 以下方法不会被 AOP 拦截：

- `private`
- `static`
- `final`

因为代理类无法覆盖这些方法（无法织入）。

### ❌ 同类内部方法调用不会触发 AOP

因为没有经过代理对象，会直接调用本体。

解决方式：

- 加接口 + 注入自身代理
- 或使用 `AopContext.currentProxy()`

------

## AOP、拦截器、过滤器的区别

| 技术                  | 拦截层级                       | 适用场景                   |
| --------------------- | ------------------------------ | -------------------------- |
| Filter（过滤器）      | Servlet 层                     | 跨域、编码、登录校验       |
| Interceptor（拦截器） | Spring MVC 层                  | token 校验、登录验证       |
| AOP                   | 方法层（Controller / Service） | 日志、性能监控、权限、事务 |

它们是不同层次，不冲突，经常一起用。
