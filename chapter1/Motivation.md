# 目的

### 目的
在应用开发中，将所有东西放到一块是颇为烦人(tedious)的。有很多中方法去将数据、服务，表示类(presentation classes)相互连接。为了对比这些方法，我们来编写一个匹萨🍕订购网站的账单代码:

```java
public interface BillingService{
  /**
   * Attempts to charge the order to the credit card. Both successful and
   * failed transactions will be recorded.
   *
   * @return a receipt of the transaction. If the charge was successful, the
   *      receipt will be successful. Otherwise, the receipt will contain a
   *      decline note describing why the charge failed.
   */

   /**
    * 尝试使用信用卡支付订单。记录交易的成功或者失败。
    * 返回：一张交易的收据。 收据随着交易的成功而打出，否则，收据上将包含一个“取消记录”， 记录了交易失败的原因
    */

  Receipt chargeOrder(PizzaOrder order, CreditCard creditCard);

}
```

同类的实现一起，我们将为代码编写单元测试。在测试中，我们需要一个`FakeCreditCardProcessor`而不是个真实信用卡支付。





### 直接使用构造方法
这里的代码展示了我们`new`一个credit card processor 和 transaction logger:

```java
public class RealBillingService implements BillingService {
  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    CreditCardProcessor processor = new PaypalCreditCardProcessor();
    TransactionLog transactionLog = new DatabaseTransactionLog();

    try {
      ChargeResult result = processor.charge(creditCard, order.getAmount());
      transactionLog.logChargeResult(result);

      return result.wasSuccessful()
          ? Receipt.forSuccessfulCharge(order.getAmount())
          : Receipt.forDeclinedCharge(result.getDeclineMessage());
     } catch (UnreachableException e) {
      transactionLog.logConnectException(e);
      return Receipt.forSystemFailure(e.getMessage());
    }
  }
}
```

这段代码凸显了(poses)模块化(modularity)和可测性(testability)的问题。这种直接的、编译期间的对一个真实的card processor的依赖，意味着测试代码的时候讲使用真的信用卡付款！并且，当拒绝支付(*charge is declined*)或者服务不可用的时候，测试这些情况的话，显得捉襟见肘。





### 工厂
工厂类解偶(decouples)实现类和调用方(client)。一种简单的工厂，是使用静态方法为某个接口获取和设置模拟的(mock)实现类。一个工厂的实现，可以参照以下样板(boilerplate)代码:

```java
public class CreditCardProcessorFactory {

  private static CreditCardProcessor instance;

  public static void setInstance(CreditCardProcessor processor) {
    instance = processor;
  }

  public static CreditCardProcessor getInstance() {
    if (instance == null) {
      return new SquareCreditCardProcessor();/*译者注: 注意这里用的是Square*/
    }

    return instance;
  }
}
```
在我们的调用端实现的的代码中，我们仅仅将new的调用替换成工厂：

```java
public class RealBillingService implements BillingService {
  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    CreditCardProcessor processor = CreditCardProcessorFactory.getInstance();
    TransactionLog transactionLog = TransactionLogFactory.getInstance();

    try {
      ChargeResult result = processor.charge(creditCard, order.getAmount());
      transactionLog.logChargeResult(result);

      return result.wasSuccessful()
          ? Receipt.forSuccessfulCharge(order.getAmount())
          : Receipt.forDeclinedCharge(result.getDeclineMessage());
     } catch (UnreachableException e) {
      transactionLog.logConnectException(e);
      return Receipt.forSystemFailure(e.getMessage());
    }
  }
}
```

工厂使得书写一个合适的单元测试变得可行：

```java
public class RealBillingServiceTest extends TestCase {

  private final PizzaOrder order = new PizzaOrder(100);
  private final CreditCard creditCard = new CreditCard("1234", 11, 2010);

  private final InMemoryTransactionLog transactionLog = new InMemoryTransactionLog();
  private final FakeCreditCardProcessor processor = new FakeCreditCardProcessor(); /*译者注: 注意这里可以new一个FakeCard*/

  @Override public void setUp() {
    TransactionLogFactory.setInstance(transactionLog);
    CreditCardProcessorFactory.setInstance(processor);
  }

  @Override public void tearDown() {
    TransactionLogFactory.setInstance(null);
    CreditCardProcessorFactory.setInstance(null);
  }

  public void testSuccessfulCharge() {
    RealBillingService billingService = new RealBillingService(); 
    Receipt receipt = billingService.chargeOrder(order, creditCard);/*译者注: 这次就可以从工厂里获得FakeCard了*/

    assertTrue(receipt.hasSuccessfulCharge());
    assertEquals(100, receipt.getAmountOfCharge());
    assertEquals(creditCard, processor.getCardOfOnlyCharge());
    assertEquals(100, processor.getAmountOfOnlyCharge());
    assertTrue(transactionLog.wasSuccessLogged());
  }
}
```
这段代码非常笨拙(clumsy)。全局变量维持了mock的实现，所以在setUp和tearDown的过程中必须小心翼翼。如果`tearDown`失败了，那么全局变量仍然指向测试实例（译者注：）。这可能导致别的测试出现问题，并且限制了我们并行进行多个测试。

但是最严重的问题是，依赖*被隐藏在了代码里*。如果我们在一个`CreditCardFraudTracker`上添加一个依赖，我们必须重新跑这个测试，去找到哪些依赖被破坏了(*which ones will break*)。如果我们忘了为一个production service初始化一个工厂，那么直到尝试支付之前，我们都无法发觉。随着应用的不断庞大，管理(babysitting)这些工厂阻碍了开发效率(*becomes a growing drain on productivity*)。

质量问题可以通过QA或者验收测试(acceptance tests)发现。这或许就够了，但是我们当然可以做的更好。

### 依赖注入

和工厂一样，依赖注入也仅仅是一种设计模式。其核心原则是*将表现和依赖项解析分离(separate behaviour from dependency resolution)*。在我们的例子中，`RealBillingService`并没有职责去寻找`TransactionLog`和`CreditCardProcessor`。相反，它们通过构造方法的参数传入

```java
public class RealBillingService implements BillingService {
  private final CreditCardProcessor processor;
  private final TransactionLog transactionLog;

  public RealBillingService(CreditCardProcessor processor, 
      TransactionLog transactionLog) {
    this.processor = processor;
    this.transactionLog = transactionLog;
  }

  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    try {
      ChargeResult result = processor.charge(creditCard, order.getAmount());
      transactionLog.logChargeResult(result);

      return result.wasSuccessful()
          ? Receipt.forSuccessfulCharge(order.getAmount())
          : Receipt.forDeclinedCharge(result.getDeclineMessage());
     } catch (UnreachableException e) {
      transactionLog.logConnectException(e);
      return Receipt.forSystemFailure(e.getMessage());
    }
  }
}
```

我不需要任何工厂，并且通过移除`setUp`和`tearDown`样板，我们可以简化测试用例：
 
 ```java
 public class RealBillingServiceTest extends TestCase {

  private final PizzaOrder order = new PizzaOrder(100);
  private final CreditCard creditCard = new CreditCard("1234", 11, 2010);

  private final InMemoryTransactionLog transactionLog = new InMemoryTransactionLog();
  private final FakeCreditCardProcessor processor = new FakeCreditCardProcessor();

  /*译者注: 和上一版相比，这里不再使用setUp()和tearDown()了*/
  public void testSuccessfulCharge() {
    RealBillingService billingService
        = new RealBillingService(processor, transactionLog);
    Receipt receipt = billingService.chargeOrder(order, creditCard);

    assertTrue(receipt.hasSuccessfulCharge());
    assertEquals(100, receipt.getAmountOfCharge());
    assertEquals(creditCard, processor.getCardOfOnlyCharge());
    assertEquals(100, processor.getAmountOfOnlyCharge());
    assertTrue(transactionLog.wasSuccessLogged());
  }
}
```

现在，不论什么时候我们添加或移除以来，编译器会提醒我们测试代码需要修复。依赖*被暴露在了API签名(API signature)*。

不幸的是，现在`BillingService`的调用方需要去检查这些依赖。我们又可以用一些设计模式来修复一部分依赖！依赖`BillingService`的这些类，需要将其放入他们的构造方法(**)。对于那些处于顶层的类，拥有一个框架(译者注: 这里指依赖注入框架)是十分有用的。否则在你要使用服务的时候，你需要递归去创建依赖。

```java
  public static void main(String[] args) {
    CreditCardProcessor processor = new PaypalCreditCardProcessor();
    TransactionLog transactionLog = new DatabaseTransactionLog();
    BillingService billingService
        = new RealBillingService(processor, transactionLog);
    ...
  }
```



### Guice的依赖注入

依赖注入这种模式，使得代码模块化并且可测试，而Guice能让依赖注入变得更易书写。在我们的账单例子中使用Guice，我们需要首先告诉它如何将接口匹配(map)到它们的实现。这个配置在在Guice的module中完成，module是一种Java类，它实现了`Module`接口:

```java
public class BillingModule extends AbstractModule {
  @Override 
  protected void configure() {
    bind(TransactionLog.class).to(DatabaseTransactionLog.class);
    bind(CreditCardProcessor.class).to(PaypalCreditCardProcessor.class);
    bind(BillingService.class).to(RealBillingService.class);
  }
}
```

我们在`RealBillingService`的构造方法上面添加`@Injection`注解，这个注解可以引导Guice去使用它。Guice巡查被注解的构造方法，并且为每个参数检查值。(**)

```java
public class RealBillingService implements BillingService {
  private final CreditCardProcessor processor;
  private final TransactionLog transactionLog;

  @Inject
  public RealBillingService(CreditCardProcessor processor,
      TransactionLog transactionLog) {
    this.processor = processor;
    this.transactionLog = transactionLog;
  }

  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    try {
      ChargeResult result = processor.charge(creditCard, order.getAmount());
      transactionLog.logChargeResult(result);

      return result.wasSuccessful()
          ? Receipt.forSuccessfulCharge(order.getAmount())
          : Receipt.forDeclinedCharge(result.getDeclineMessage());
     } catch (UnreachableException e) {
      transactionLog.logConnectException(e);
      return Receipt.forSystemFailure(e.getMessage());
    }
  }
}
```

最终，我们可将所有东西放在一起。`Injector`可以用来获得任何被绑定类的实例。

```java
  public static void main(String[] args) {
    Injector injector = Guice.createInjector(new BillingModule());
    BillingService billingService = injector.getInstance(BillingService.class);
    ...
  }
```


[Getting started](Getting_Started.md)解释了它们是如何运作的。