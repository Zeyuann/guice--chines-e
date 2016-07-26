# 链接绑定(Linked Bindings)

链接绑定将类型映射在其实现上。下面这个例子将`TransactionLog`接口映射到了它的实现类`DatabaseTransactionLog`:

```java
public class BillingModule extends AbstractModule {
    @Override
    protected void configure() {
        bind(TransactionLog.class).to(DatabaseTransactionLog.class);
    }
}
```       
现在，当你调用`injector.getInstance(TransactionLog.class)`,或者当injector遇到`TransactionLog`的依赖时，将使用一个`DatabaseTransactionLog`。将一种类型链接(link)到任何一个其子类型，比如实现类或者继承类。你甚至可以将一个具体的类`DatabaseTransactionLog`链接到一个子类上面：

``` 
bind(DatabaseTransactionLog.class).to(MySqlDatabaseTransactionLog.class);
```

链接绑定也可以是链式的：

```java
public class BillingModule extends AbstractModule {
    @Override
    protected void configure() {
        bind(TransactionLog.class).to(DatabaseTransactionLog.class);
        bind(DatabaseTransactionLog.class).to(MySqlDatabaseTransactionLog.class);
    }
}
```

在这个例子里，当请求一个`TransactionLog`时，injector将返回一个`MySqlDatabaseTransactionLog`