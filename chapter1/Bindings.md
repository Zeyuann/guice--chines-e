# 绑定(Bindings)

*概述Guice中的bindings*

### Bindings
injector的工作是为对象装配图。当你请求一个既定类型的实例时，它能指明需要构建什么，解析依赖，并且把所有东西链接起来(wires)。利用bindings来配置你的injector，可以解释(specify)这些依赖是如何解析的。

#### 创建Bindings
创建bindings, 拓展`AbstractModule`，并且覆盖它的`configure`方法。在方法体中，调用`bind()`来指定(specify)每个binding。这些方法是经过类型检查的，所以当你使用了错误的类型时，编译器可以给出error提示。一旦你创建好了modules，将它们作为参数传递给`Guice.createInjector()`，就可以构建一个injector了。

利用modules去创建
[链式绑定(Linked Bindings)](Linked_Bindings.md)，
[实例绑定(Instance Bindings)](Instance_Bindings.md)，
[@Provides 方法](@Provide_Methods.md)，
[生产者绑定(Provider Bindings)](Provider_Bindings.md)，
[构造方法绑定(Constructor Bindings)](Constructor_Bindings.md)， 
[无目标绑定(Untargeted Bindings)](Untargeted_Bindings.md)。


#### 更多的bindings
除了bindings，你可以指定包含 built-in bindings的injector。当请求一个依赖但是找不到响应的依赖时，它尝试去创建一个 just-in-time binding。injector也包含了一些bindings，这些bindings为是为调用者提供的，并且包含了其他的bindings。(*The injector also includes bindings for the providers of its other bindings.*)