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


### Direct constructor calls

## 从这里开始

## 绑定