CORS，全称Cross-Origin Resource Sharing，是HTML5规范定义的如何跨域访问资源。

在开发REST应用时，很多时候，是通过页面的JavaScript和后端的REST API交互。在JavaScript与REST交互的时候，有很多安全限制。默认情况下，浏览器按同源策略放行JavaScript调用API，即：

- 如果A站在域名`a.com`页面的JavaScript调用A站自己的API时，没有问题；
- 如果A站在域名`a.com`页面的JavaScript调用B站`b.com`的API时，将被浏览器拒绝访问，因为不满足同源策略。

同源要求**域名**要完全相同（`a.com`和`www.a.com`不同），**协议**要相同（`http`和`https`不同），**端口**要相同 。

如果A站的JavaScript访问B站API的时候，B站能够返回响应头`Access-Control-Allow-Origin: http://a.com`，那么，浏览器就允许A站的JavaScript访问B站的API。

## Spring设置CORS

### 1. 使用@CrossOrigin

可以在`@RestController`的**class级别或方法级别**定义一个`@CrossOrigin`，例如：

```java
@CrossOrigin(origins = "http://abc.com:8080")
@RestController
@RequestMapping("/api")
public class ApiController {
    ...
}
```

上述定义在`ApiController`处的`@CrossOrigin`指定了只允许来自`abc.com`跨域访问，允许多个域访问需要写成数组形式，例如`origins = {"http://a.com", "https://www.b.com"})`。如果要允许任何域访问，写成`origins = "*"`即可。

如果有多个REST Controller都需要使用CORS，那么，每个Controller都必须标注`@CrossOrigin`注解。

### 2. 使用CorsRegistry

在`WebMvcConfigurer`中定义一个全局CORS配置，下面是一个示例：

```java
@Bean
WebMvcConfigurer createWebMvcConfigurer() {
    return new WebMvcConfigurer() {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/api/**") // 对指定URL设置规则
                    .allowedOrigins("http://abc.com:8080")
                    .allowedMethods("GET", "POST")
                    .maxAge(3600);
            // 可以继续添加其他URL规则:
            // registry.addMapping("/rest/v2/**")...
        }
    };
}
```

这种方式可以创建一个全局CORS配置，如果仔细地设计URL结构，那么可以一目了然地看到各个URL的CORS规则，**推荐使用这种方式配置CORS**。

