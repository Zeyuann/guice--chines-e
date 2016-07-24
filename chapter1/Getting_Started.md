#  入门指南
*开始学习如何用Guice依赖注入。*

### 入门指南
利用依赖注入，对象通过它们的构造函数来接受依赖。为了构造一个对象，你首先要构建(build)他们的依赖。但为了构建这些依赖，你需要构建*这些依赖*的依赖，如此下去。所以，当你构建一个对象的时候，你真的需要构建*对象图(object graph)*。

手动构建对象图不仅是劳动密集的(labour intensive)且易错的(error prone),还会让测试变得困难。相反，Guice可以为你建立对象图。但是首先，Guice需要经过配置，才能构造出完全符合你需求的图。

举个栗子，我们们将`BillingService`这个类启动，它的构造方法中接受了依赖的接口：`CreditCardProcessor`和`TransactionLog`。`BillingService`的构造方法被Guice调用，为了使这个过程更显而易见，我们添加了`@Inject`注解：

```java
class BillingService {
  private final CreditCardProcessor processor;
  private final TransactionLog transactionLog;

  @Inject
  BillingService(CreditCardProcessor processor, 
      TransactionLog transactionLog) {
    this.processor = processor;
    this.transactionLog = transactionLog;
  }

  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    ...
  }
}
```

我们希望通过`PaypalCreditCardProcessor`和`DatabaseTransactionLog`去构建`BillingService`。Guice使用了*bindings*去将类型和实现进行匹配。一个`module`是存放了bindings的集合，每个binding表示流式的(fluent)，英语式的(English-like)方法调用。

```java
public class BillingModule extends AbstractModule {
  @Override 
  protected void configure() {

     /*
      * This tells Guice that whenever it sees a dependency on a TransactionLog,
      * it should satisfy the dependency using a DatabaseTransactionLog.
      * 
      * 这里告诉了Guice，不论何时Guice看到TransactionLog类型的依赖，Guice都应该将依赖适配(satisfy)* 为DatabaseTransactionLog。
      */
    bind(TransactionLog.class).to(DatabaseTransactionLog.class);

     /*
      * Similarly, this binding tells Guice that when CreditCardProcessor is used in
      * a dependency, that should be satisfied with a PaypalCreditCardProcessor.
      * 
      * 同样地，这个binding告诉了Guice，当CreditCardProcessor配用为一个依赖，那么Guice应该适配为
      * DatabaseTransactionLog。
      */
    bind(CreditCardProcessor.class).to(PaypalCreditCardProcessor.class);
  }
}
```

The modules are the building blocks of an injector, which is Guice's object-graph builder. First we create the injector, and then we can use that to build the BillingService:

modules是一个`injector`的构建块，而`injector`是Guice的对象图构建器(builder)。首先我们产生一个injector，然后我们使用它去构建`BillingService`：

```java
public static void main(String[] args) {
    /*
     * Guice.createInjector() takes your Modules, and returns a new Injector
     * instance. Most applications will call this method exactly once, in their
     * main() method.
     * 
     * Guice.createInjector()取到你的Modules，并且返回一个新的Injector实例。大多数应用会且仅会
     * 在main()中调用这个方法一次，
     */
    Injector injector = Guice.createInjector(new BillingModule());

    /*
     * Now that we've got the injector, we can build objects.
     *
     * 现在我们取到了injector，我们可以构建对象了。
     */
    BillingService billingService = injector.getInstance(BillingService.class);
    ...
  }
```
在构建billingService的过程中，我们已经通过Guice构造了一个小的对象图。这个图包含了billing service和其依赖的credit card processo和transaction log。