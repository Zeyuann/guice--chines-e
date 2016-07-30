# @Provides方法

### @Provides方法

但你需要用代码去创建对象时，可以使用@Provides方法。这个方法必须在module中定义，并且必须有一个`@Provides`注解。
方法的返回值是被绑定的类型(bound type)。injector一旦需要一个这个类型的实例，injector酒会调用这个方法。

```java
public class BillingModule extends AbstractModule {
  @Override
  protected void configure() {
    ...
  }

  @Provides
  TransactionLog provideTransactionLog() {
    DatabaseTransactionLog transactionLog = new DatabaseTransactionLog();
    transactionLog.setJdbcUrl("jdbc:mysql://localhost/pizza");
    transactionLog.setThreadPoolSize(30);
    return transactionLog;
  }
}
```

如果`@Provides`方法拥有一个绑定注解，例如`@PayPal`或是`@Named("Checkout")`，Guice将绑定被注解的类型。依赖会作为参数传入到方法中。injector将在调用方法之前，尝试去将这些依赖绑定起来。

```java
  @Provides @PayPal
  CreditCardProcessor providePayPalCreditCardProcessor(
      @Named("PayPal API key") String apiKey) {
    PayPalCreditCardProcessor processor = new PayPalCreditCardProcessor();
    processor.setApiKey(apiKey);
    return processor;
  }
```



### 抛出异常
Guice不允许Providers直接抛出异常，`@Provides`抛出的异常将被`ProvisionException`封装起来。将任何类型的异常从`@Provides`抛出，不论是运行时异常或是受检异常，不是一种好的实践。如果你因为某种原因需要抛出一个异常，你可能需要[ThrowingProviders extension]()中的`@CheckedProvides`注解的那些方法。