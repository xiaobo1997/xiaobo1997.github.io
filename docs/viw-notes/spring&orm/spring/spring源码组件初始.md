**Spring的核心是容器，而容器并不唯一**，框架本身就提供了很多个容器的实现，大概分为两种类型：一种是不常用的BeanFactory，这是最简单的容器，只能提供基本的DI功能；还有一种就是继承了BeanFactory后派生而来的**应用上下文**，其抽象接口也就是我们上面提到的的ApplicationContext，它能提供更多企业级的服务，例如解析配置文本信息等等，这也是应用上下文实例对象最常见的应用场景。

- 加载读取xml文件中的信息，再通过动态代理生成具体对象