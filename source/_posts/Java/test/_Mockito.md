Mock其实就是为了保证单测的non-determinitistc性质，不能连接实际的数据库，而是要构造假的repository, service来测试代码本身的正确性

文档：https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html