`DelegatingFilterProxy`: 让Servlet容器实例化的Filter，间接引用Spring容器实例化的`AuthFilter`。

实现原理：

1. Servlet容器从`web.xml`中读取配置，实例化`DelegatingFilterProxy`，注意命名是`authFilter`；
2. Spring容器通过扫描`@Component`实例化`AuthFilter`。