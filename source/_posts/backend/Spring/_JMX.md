# JMX (Java Management Extensions)

步骤：

1. 在Config类中加上`@EnableMBeanExport`注解，告诉Spring自动注册MBean

2. 编写MBean，实现管理属性和管理操作

## 例：给应用程序添加IP黑名单

需求本质上是在应用程序运行期间对参数、配置等进行热更新并要求尽快生效。如果以JMX的方式实现，我们不必自己编写自动重新读取等任何代码，只需要提供一个符合JMX标准的MBean来存储配置即可。

### 编写MBean

JMX的MBean通常以MBean结尾，因此我们遵循标准命名规范，首先编写一个`BlacklistMBean`：

```java
public class BlacklistMBean {
    private Set<String> ips = new HashSet<>();

    public String[] getBlacklist() {
        return ips.toArray(String[]::new);
    }

    public void addBlacklist(String ip) {
        ips.add(ip);
    }

    public void removeBlacklist(String ip) {
        ips.remove(ip);
    }

    public boolean shouldBlock(String ip) {
        return ips.contains(ip);
    }
}
```

使用JMX的客户端来实时热更新这个MBean。要给它加上一些注解，让Spring能根据注解自动把相关方法注册到MBeanServer中：

```java
@Component
@ManagedResource(objectName = "sample:name=blacklist", description = "Blacklist of IP addresses")
public class BlacklistMBean {
    private Set<String> ips = new HashSet<>();

    @ManagedAttribute(description = "Get IP addresses in blacklist")
    public String[] getBlacklist() {
        return ips.toArray(String[]::new);
    }

    @ManagedOperation
    @ManagedOperationParameter(name = "ip", description = "Target IP address that will be added to blacklist")
    public void addBlacklist(String ip) {
        ips.add(ip);
    }

    @ManagedOperation
    @ManagedOperationParameter(name = "ip", description = "Target IP address that will be removed from blacklist")
    public void removeBlacklist(String ip) {
        ips.remove(ip);
    }

    public boolean shouldBlock(String ip) {
        return ips.contains(ip);
    }
}
```

`BlacklistMBean`是一个标准的Spring管理的Bean。添加了`@ManagedResource`注解，表示这是一个MBean，将要被注册到JMX。objectName指定了这个MBean的名字，通常以`company:name=Xxx`来分类MBean。

对于属性，使用`@ManagedAttribute`注解标注。上述MBean只有get属性，没有set属性，说明这是一个只读属性。

> 注解标为ManagedAttribute的必须是setter、getter方法

对于操作，使用`@ManagedOperation`注解标注。上述MBean定义了两个操作：`addBlacklist()`和`removeBlacklist()`，其他方法如`shouldBlock()`不会被暴露给JMX。

### 使用MBean

例如，我们在`BlacklistInterceptor`对IP进行黑名单拦截：

```java
@Order(1)
@Component
public class BlacklistInterceptor implements HandlerInterceptor {
    final Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    BlacklistMBean blacklistMBean;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        String ip = request.getRemoteAddr();
        logger.info("check ip address {}...", ip);
        // 是否在黑名单中:
        if (blacklistMBean.shouldBlock(ip)) {
            logger.warn("will block ip {} for it is in blacklist.", ip);
            // 发送403错误响应:
            response.sendError(403);
            return false;
        }
        return true;
    }
}
```

可以用`jconsole`测试。

## 注

在实际项目中，通过JMX实现配置的实时更新其实并不常用，JMX更多地用于**收集JVM的运行状态和应用程序的性能数据**，然后**通过监控服务器汇总数据后实现监控与报警**。

App自身和JVM的的统计数据都通过JMX收集并发送给本机的一个Agent，Agent再将数据发送至监控服务器，最后以可视化的形式将监控数据通过Web等形式展示给用户。常用的监控系统有开源的[Prometheus](https://prometheus.io/)和以云服务方式提供的[DataDog](https://www.datadoghq.com/)等。