



## Conditional

Spring本身提供了条件装配`@Conditional`，但是要自己编写比较复杂的`Condition`来做判断，比较麻烦。Spring Boot则为我们准备好了几个非常有用的条件：

- @ConditionalOnProperty：如果有指定的配置，条件生效；
- @ConditionalOnBean：如果有指定的Bean，条件生效；
- @ConditionalOnMissingBean：如果没有指定的Bean，条件生效；
- @ConditionalOnMissingClass：如果没有指定的Class，条件生效；
- @ConditionalOnWebApplication：在Web环境中条件生效；
- @ConditionalOnExpression：根据表达式判断条件是否生效。

首先，定义配置`storage.type=xxx`，用来判断条件，默认为`local`：

```yaml
storage:
  type: ${STORAGE_TYPE:local}
```

设定为`local`时，启用`LocalStorageService`：

```java
@Component
@ConditionalOnProperty(value = "storage.type", havingValue = "local", matchIfMissing = true)
public class LocalStorageService implements StorageService {
    ...
}
```

设定为`aws`时，启用`AwsStorageService`：

```java
@Component
@ConditionalOnProperty(value = "storage.type", havingValue = "aws")
public class AwsStorageService implements StorageService {
    ...
}
```

设定为`aliyun`时，启用`AliyunStorageService`：

```java
@Component
@ConditionalOnProperty(value = "storage.type", havingValue = "aliyun")
public class AliyunStorageService implements StorageService {
    ...
}
```

注意到`LocalStorageService`的注解，当指定配置为`local`，或者配置不存在，均启用`LocalStorageService`。

## 加载配置文件

加载配置文件可以直接使用注解`@Value`，但多次引用同一个`@Value`不但麻烦，而且`@Value`使用字符串，缺少编译器检查，容易造成多处引用不一致。

为了更好地管理配置，Spring Boot允许创建一个Bean，持有一组配置，并由Spring Boot自动注入。

假设我们在`application.yml`中添加了如下配置：

```yaml
storage:
  local:
    # 文件存储根目录:
    root-dir: ${STORAGE_LOCAL_ROOT:/var/storage}
    # 最大文件大小，默认100K:
    max-size: ${STORAGE_LOCAL_MAX_SIZE:102400}
    # 是否允许空文件:
    allow-empty: false
    # 允许的文件类型:
    allow-types: jpg, png, gif
```

可以首先定义一个Java Bean，持有该组配置，保证Java Bean的属性名称与配置一致即可。然后，我们添加两个注解：

```java
@Configuration
@ConfigurationProperties("storage.local")
public class StorageConfiguration {

    private String rootDir;
    private int maxSize;
    private boolean allowEmpty;
    private List<String> allowTypes;

    // TODO: getters and setters
}
```

注意到`@ConfigurationProperties("storage.local")`表示将从配置项`storage.local`读取该项的所有子项配置，并且，`@Configuration`表示`StorageConfiguration`也是一个Spring管理的Bean，可直接注入到其他Bean中。

```java
@Component
public class StorageService {
    final Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    StorageConfiguration storageConfig;

    @PostConstruct
    public void init() {
        logger.info("Load configuration: root-dir = {}", storageConfig.getRootDir());
        logger.info("Load configuration: max-size = {}", storageConfig.getMaxSize());
        logger.info("Load configuration: allowed-types = {}", storageConfig.getAllowTypes());
    }
}
```

这样一来，引入`storage.local`的相关配置就很容易了，因为只需要注入`StorageConfiguration`这个Bean，这样可以由编译器检查类型，无需编写重复的`@Value`注解。