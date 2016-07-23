# 1. 用户手册

## 目的
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


### 直接使用构造函数
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
      return new SquareCreditCardProcessor();
    }

    return instance;
  }
}
```


## 从这里开始

## 绑定