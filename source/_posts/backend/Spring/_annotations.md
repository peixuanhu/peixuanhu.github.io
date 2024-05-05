

注解总结：



### 注入配置

有@Configuration的Config类添加注解@PropertySource("app.properties")

例：

```java
@Configuration
@ComponentScan
@PropertySource({ "app.properties", "other.properties" }) // 表示读取classpath的properties文件
public class AppConfig {
    @Value("${app.zone:Z}") // 如果获取不到，取默认值Z
    String zoneId;

    @Bean
    ZoneId createZoneId() {
        return ZoneId.of(zoneId);
    }
}
```

在需要读取的地方注入：

```java
@Component
public class ZoneService {
  @Value("#{AppConfig.zoneId}")
  private String zoneId;
}
```



### Scheduler

Config类中加上`@EnableScheduling`，在某个Bean的方法中加上`@scheduled`注解。

> 多个`@Scheduled`方法完全可以放到一个Bean中，这样便于统一管理各类定时任务。

例：

```java
@Component
public class TaskService {
    final Logger logger = LoggerFactory.getLogger(getClass());

    @Scheduled(initialDelay = 60_000, 
               fixedDelayString = "${task.checkDiskSpace:PT2M30S}")
    public void checkSystemStatusEveryMinute() {
        logger.info("Start check system status...");
    }
}
```

以字符串`PT2M30S`表示的`Duration`就是2分30秒。

#### Cron任务

例：**每天凌晨2:15**执行报表任务

Cron源自Unix/Linux系统自带的crond守护进程，以一个简洁的表达式定义任务触发时间。在Spring中，也可以使用Cron表达式来执行Cron任务，在Spring中，它的格式是：

```
秒 分 小时 天 月份 星期 年
```

年是可以忽略的，通常不写。每天凌晨2:15执行的Cron表达式就是：

```
0 15 2 * * *
```

每个工作日12:00执行的Cron表达式就是：

```
0 0 12 * * MON-FRI
```

每个月1号，2号，3号和10号12:00执行的Cron表达式就是：

```
0 0 12 1-3,10 * *
```

Cron表达式还可以表达每10分钟执行，例如：

```
0 */10 * * * *
```

这样，在每个小时的0:00，10:00，20:00，30:00，40:00，50:00均会执行任务，实际上它可以取代`fixedRate`类型的定时任务。



在Spring中，我们定义一个每天凌晨2:15执行的任务：

```java
@Component
public class TaskService {
    ...

    @Scheduled(cron = "${task.report:0 15 2 * * *}")
    public void cronDailyReport() {
        logger.info("Start daily report task...");
    }
}
```





@Transactional

> Spring的JDBC事务是基于`ThreadLocal`实现的



@RestController

使用restful API的controller



在POJO某属性添加的注解：

@JsonIgnore

@JsonProperty(access = Access.WRITE_ONLY) 只准输入不准输出

@JsonProperty(access = Access.READ_ONLY) 只准输出不准输入



