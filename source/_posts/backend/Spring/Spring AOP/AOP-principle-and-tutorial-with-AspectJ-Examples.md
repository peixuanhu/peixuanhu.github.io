---
title: AOP原理、术语和AspectJ示例
categories:
  - [Backend, Spring, Spring AOP]
---

# AOP原理

在Java平台上，对于AOP的织入，有3种方式：

1. 编译期：在编译时，由编译器把切面调用编译进字节码，这种方式需要定义新的关键字并扩展编译器，AspectJ就扩展了Java编译器，使用关键字aspect来实现织入；
2. 类加载器：在目标类被装载到JVM时，通过一个特殊的类加载器，对目标类的字节码重新“增强”；
3. 运行期：目标对象和切面都是普通Java类，通过JVM的动态代理功能或者第三方库实现运行期动态织入。

最简单的方式是第三种，Spring的AOP实现就是基于JVM的动态代理，应用了代理模式。

AOP让我们把一些常用功能如权限检查、日志、事务等，从每个业务方法中剥离出来。

> AOP对于解决特定问题，例如事务管理非常有用，这是因为分散在各处的事务代码几乎是完全相同的，并且它们需要的参数（JDBC的Connection）也是固定的。
>
> 另一些特定问题，如日志，就不那么容易实现，因为日志虽然简单，但打印日志的时候，经常需要捕获局部变量，如果使用AOP实现日志，我们只能输出固定格式的日志，
>
> 因此，使用AOP时，必须适合特定的场景。

# Terminologies

## 概览

- Aspect：切面，即一个横跨多个核心逻辑的功能，或者称之为系统关注点；
- Joinpoint：连接点，即定义在应用程序流程的何处插入切面的执行；
- Pointcut：切入点，即一组连接点的集合；
- Advice：增强，指特定连接点上执行的动作；
- Introduction：引介，指为一个已有的Java对象动态地增加新的接口；
- Weaving：织入，指将切面整合到程序的执行流程中；
- Interceptor：拦截器，是一种实现增强的方式；
- Target Object：目标对象，即真正执行业务的核心逻辑对象；
- AOP Proxy：AOP代理，是客户端持有的增强后的对象引用。

## Static - Compile Time

### Aspect

Aspect is the concern that we are trying to implement generically.  The functionality you want to implement. What you want to achieve.

#### Examples

- Transaction Management
- Logging (e.g. Every method of business layer called, log the input and output parameters)
- Performance metrics (how much time a method took to execute)

### Pointcut

A regular expression which determines what are the methods are to be intercepted.

e.g. execution(* com.example.spring.business.aop.HiByeService.*(..))

### Advice

Action taken by an aspect at a particular join point. What is to be done when the Pointcut is met.



##  Dynamic - Runtime

### Join point

A point during the execution of a program. Execution of a specific AOP method.

All the runtime information about the method call.

Input parameters of the Advice?

### Weaving

The entire process of executing the Advice.

# AspectJ

## Annotations

### Aspect类

- @Order(3)，值越小优先级越高
- @Aspect

### AspectJ Advice Types

- **@Before**（前置通知）：先执行拦截代码，再执行目标代码。如果拦截器抛异常，那么目标代码就不执行了；

- **@After** （后置通知）：先执行目标代码，再执行拦截器代码。无论目标代码是否抛异常，拦截器代码都会执行；

- **@AfterReturning**（返回通知）：和@After不同的是，只有当目标代码正常返回时，才执行拦截器代码；

- **@AfterThrowing**（异常通知）：和@After不同的是，只有当目标代码抛出了异常时，才执行拦截器代码；

  > AfterReturning 和 AfterThrowing 两者互斥。如果方法调用成功无异常，则会有返回值；如果方法抛出了异常，则不会有返回值。

- **@Around** （环绕通知）：能完全控制目标代码是否执行，并可以在执行前后、抛异常后执行任意拦截代码，可以说是包含了上面所有功能

## Example

```java
@Aspect // AspectJ annotation
@Component // Managed by Spring
class MyAspect {
    
    @Before("execution(* com.example.spring.business.aop.HiByeService.*(..))") // 默认等价于pointcut="execution..."
    public void before(JoinPoint joinPoint) {
        System.out.print("Before ");
        System.out.println(joinPoint.getSignature().getName());
    }
    
    @AfterReturning(pointcut="execution(* com.example.spring.business.aop.HiByeService.*(..))", returning="result")
    public void afterReturning(JoinPoint joinPoint, Object result) {
        System.out.print("After ");
        System.out.print(joinPoint.getSignature().getName());
        System.out.println(", result is " + result);
    }
}

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringContextAOP.class)
public class AOPExampleTest {
    
    @Autowired
    private HiByeService service;
    
    @Test
    public void testSomething() {
        service.sayHi();
        service.sayBye();
    }
}
```

### 使用注解装配AOP

例：监控应用程序的性能

#### 1. 定义一个性能监控的注解

```java
@Target(METHOD)
@Retention(RUNTIME)
public @interface MetricTime {
  String value();
}
```

#### 2. 在需要被监控的关键方法上标注该注解

```java
@Component
public class UserService {
  // 监控register()方法性能
  @MetricTime("register")
  public User register(String email, String password, String name) {
    ...
  }
  ...
}
```

#### 3. 定义`MetricAspect`

```java
@Aspect
@Component
public class MetricAspect {
  @Around("@annotation(metricTime)")
  public Object metric(ProceedingJoinPoint joinPoint, MetricTime metricTime) throws Throwable {
    String name = metricTime.value();
    long start = System.currentTimeMillis();
    try {
      return joinPoint.proceed();
    } finally {
      long t = System.currentTimeMillis() - start;
      System.err.println("[Metrics] " + name + ": " + t + "ms");
    }
  }
}
```

`metric()`方法标注了`@Around("@annotation(metricTime)")`，它的意思是，符合条件的目标方法是带有`@MetricTime`注解的方法，因为`metric()`方法**参数类型**是`MetricTime`（注意参数名是`metricTime`不是`MetricTime`），我们通过它获取性能监控的名称。

> Aspect织入的地方取决于`metric`方法的第二个入参类型

有了`@MetricTime`注解，再配合`MetricAspect`，任何Bean，只要方法标注了`@MetricTime`注解，就可以自动实现性能监控。运行代码，输出结果如下：

```css
Welcome, Bob!
[Metrics] register: 16ms
```

## 步骤

1. 定义执行方法，并在方法上通过AspectJ的注解告诉Spring应该在何处调用此方法；
2. 标记`@Component`和`@Aspect`；
3. 在`@Configuration`类上标注`@EnableAspectJAutoProxy`。

# Note

1. 访问被注入的Bean时，总是调用方法而非直接访问字段
   - 因为CGLIB代理类不会调用super()，也就是父类即原始类的构造函数，所以自己继承来的该字段不会被初始化
   - 但调用方法会委托给**引用的原始类的实例**的该方法（跟父类就没关系了）
2. 编写Bean时，如果可能会被代理，就不要编写`public final`方法。

# References

1.  Spring doc for AspectJ: https://docs.spring.io/spring-framework/reference/core/aop/ataspectj.html
1.  注解装配AOP：https://www.liaoxuefeng.com/wiki/1252599548343744/1310052317134882
1.  AOP注意事项：https://www.liaoxuefeng.com/wiki/1252599548343744/1339039378571298
1.  Spring AOP Tutorial - with AspectJ Examples: [https://www.youtube.com/watch?v=Og9Fyew8ltQ](https://www.youtube.com/watch?v=Og9Fyew8ltQ)
