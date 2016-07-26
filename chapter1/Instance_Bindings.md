# 实例绑定(Instance Bindings)

### 实例绑定
你可以为一种类型去绑定这个类型的特定实例。这通常仅仅用于在这种情况：这个对象自己并没有依赖，比如值对象(value objects)(译者注：值对象就是没有任何业务，仅仅用于传递数据的简单对象。from有道词典)。：

```java
    bind(String.class)
        .annotatedWith(Names.named("JDBC URL"))
        .toInstance("jdbc:mysql://localhost/pizza");
    bind(Integer.class)
        .annotatedWith(Names.named("login timeout seconds"))
        .toInstance(10);
```

Avoid using .toInstance with objects that are complicated to create, since it can slow down application startup. You can use an @Provides method instead.

应当避免在`.toInstance`中使用一个复杂对象，因为这会使拖慢应用程序的启动。你可以使用`@Provider`方法代替。
