# 绑定有关的注解(Binding Annotations)

###绑定有关的注解
有时你可能需要为一种类型进行多个绑定。例如，你可能需要Palpal信用卡和Google支付的这两个支付的processor(译者注：1.请回顾和” 目的(Motivation)“那节的例子，2.processor在这里不做翻译)。为了实现这个功能，bindings支持你用可选的*绑定注解(binding annotations)*。这些注解，同类型一起，可以唯一地识别出绑定。这种对应(pair)我们称之为*key*

为了定义一个绑定注解，我们需要在众多的import之后，加入另外的两行。将定义放入目标`.java`文件，或者在注解处标明(译者注：即不将注解import进来，而用包名＋注解名直接写在注解处)。

```java
package example.pizza;

import com.google.inject.BindingAnnotation;
import java.lang.annotation.Target;
import java.lang.annotation.Retention;
import static java.lang.annotation.RetentionPolicy.RUNTIME;
import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.ElementType.METHOD;

@BindingAnnotation @Target({ FIELD, PARAMETER, METHOD }) @Retention(RUNTIME)
public @interface PayPal {}
```

你不需要去弄清楚所有的这些元注解，但是如果你对此感兴趣：
* `@BindingAnnotation`告诉Guice，这是一个“绑定注解”。如果某个成员在任何时候拥有多个“绑定注解”，那么Guice会报错。
* `@Target({FIELD, PARAMETER, METHOD})`对于使用者来说是一种方便(*courtesy*)。它避免了一种情形，即`@PayPal`被意外地使用在了无法提供服务的地方。(译者注：注解被用在了不合适的地方，没有了依赖注入的效果)
* `@Retention(RUNTIME)`使注解在运行时生效

若想使用已经被注解的绑定，只需要将注解作用在待注入的参数上：

```java
public class RealBillingService implements BillingService {

    @Inject
    public RealBillingService(@PayPal CreditCardProcessor processor, 
        TransactionLog transactionLog) {
    ...
        }
}
```

最后，我们通过注解来建立一个绑定。在bind()表达式中使用了可选的子句(clause)—— annotatedWith：

```java
    bind(CreditCardProcessor.class)
        .annotatedWith(PayPal.class)
        .to(PayPalCreditCardProcessor.class);
```





### @Named
Guice自带一个内置的“绑定注解”，就是`@Named`，配合一个字符串使用。

```java
public class RealBillingService implements BillingService {

  @Inject
  public RealBillingService(@Named("Checkout") CreditCardProcessor processor,
      TransactionLog transactionLog) {
    ...
  }
```

如果要绑定一个特定的名称，那么可以使用`Names.named`创建一个实例，并且把这个实例传递给`annotatedWith`：

```java
    bind(CreditCardProcessor.class)
        .annotatedWith(Names.named("Checkout"))
        .to(CheckoutCreditCardProcessor.class);
```

因为编译器无法检查到这个字符串，所以我们推荐您谨慎(sparingly)使用`@Named`。




### 为注解绑定属性

Guice支持拥有属性的“绑定注解”。在一些不常见的情形下，你可能需要这样的注解：

1. 建立`@Interface`注解
2. 建立interface注解的实现类。并按照[Annotation Javadoc]()中关于`equals()`和`hashCode()`的相关步骤走。把这个实现类的一个实例放入`annotatedWith()`子句。