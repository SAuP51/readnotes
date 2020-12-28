# 0901. LSP: The Liskov Substitution Principle

In 1988, Barbara Liskov wrote the following as a way of defining subtypes.

What is wanted here is something like the following substitution property: If  for each object o1 of  type S there is an object o2 of  type T such that for all  programs P defined in terms of  T, the behavior of  P is unchanged when o1 is  substituted for o2 then S is a subtype of  T.1

To understand this idea, which is known as the Liskov Substitution Principle (LSP), let’s look at some examples.

1 Barbara Liskov,「Data Abstraction and Hierarchy,」SIGPLAN Notices 23, 5 (May 1988).

1988 年，Barbara Liskov 在描述如何定义子类型时写下了这样一段话：

这里需要的是一种可替换性：如果对于每个类型是 S 的对象 o1 都存在一个类型为 T 的对象 o2，能使操作类型的程序 P 在用 o2 替换 o1 时行为保持不变，我们就可以将 S 称为 T 的子类型。为了让读者理解这段话中所体现的设计理念，也就是里氏替换原则（LSP），我们可以来看几个例子。

## 9.1 Guiding the Use of Inheritance

Imagine that we have a class named License, as shown in Figure 9.1. This class has a method named calcFee(), which is called by the Billing application. There are two「subtypes」of License: PersonalLicense and BusinessLicense. They use different algorithms to calculate the license fee.

Figure 9.1  License, and its derivatives, conform to LSP

This design conforms to the LSP because the behavior of the Billing application does not depend, in any way, on which of the two subtypes it uses. Both of the subtypes are substitutable for the License type.

继承的使用指导

假设我们有一个 License 类，其结构如图 9.1 所示。该类中有一个名为 calcFee() 的方法，该方法将由 Billing 应用程序来调用。而 License 类有两个子类型：PersonalLicense 与 BusinessLicense，这两个类会用不同的算法来计算授权费用。上述设计是符合 LSP 原则的，因为 Billing 应用程序的行为并不依赖于其使用的任何一个衍生类。也就是说，这两个生类的对象都是可以用来替换 License 类对象的。

## 9.2 The Square/Rectangle Problem

The canonical example of a violation of the LSP is the famed (or infamous, depending on your perspective) square/rectangle problem (Figure 9.2).

Figure 9.2  The infamous square/rectangle problem

In this example, Square is not a proper subtype of Rectangle because the height and width of the Rectangle are independently mutable; in contrast, the height and width of the Square must change together. Since the User believes it is communicating with a Rectangle, it could easily get confused. The following code shows why:

Rectangle r = …

r.setW(5);

r.setH(2);

assert(r.area() == 10);

If the … code produced a Square, then the assertion would fail.

The only way to defend against this kind of LSP violation is to add mechanisms to the User (such as an if statement) that detects whether the Rectangle is, in fact, a Square. Since the behavior of the User depends on the types it uses, those types are not substitutable.

正方形 / 长方形问题

正方形 / 长方形问题是一个著名（或者说臭名远扬）的违反 LSP 的设计案例（该问题的结构如图 9.2 所示）。在这个案例中，Square 类并不是 Rectangle 类的子类型，因为 Rectangle 类的高和宽可以分别修改，而 Square 类的高和宽则必须一同修改。由于 User 类始终认为自己在操作 Rectangle 类，因此会带来一些混淆。例如在下面的代码中：

很显然，如果上述代码在，，处返回的是 Square 类，则最后的这个 assertion 是不会成立的。如果想要防范这种违反 LSP 的行为，唯一的办法就是在 User 类中增加用于区分 Rectangle 和 Square 的检测逻辑（例如增加 if 语句）。但这样一来，User 类的行为又将依赖于它所使用的类，这两个类就不能互相替换了。

## 9.3 LSP and Architecture

In the early years of the object-oriented revolution, we thought of the LSP as a way to guide the use of inheritance, as shown in the previous sections. However, over the years the LSP has morphed into a broader principle of software design that pertains to interfaces and implementations.

The interfaces in question can be of many forms. We might have a Java-style interface, implemented by several classes. Or we might have several Ruby classes that share the same method signatures. Or we might have a set of services that all respond to the same REST interface.

In all of these situations, and more, the LSP is applicable because there are users who depend on well-defined interfaces, and on the substitutability of the implementations of those interfaces.

The best way to understand the LSP from an architectural viewpoint is to look at what happens to the architecture of a system when the principle is violated.

LSP 与软件架构

在面向对象这场编程革命兴起的早期，我们的普遍认知正如上文所说，认为 LSP 只不过是指导如何使用继承关系的一种方法，然而随着时间的推移，LSP 逐渐演变成了一种更广泛的、指导接口与其实现方式的设计原则。

这里提到的接口可以有多种形式  —  —  可以是 Java 风格的接口，具有多个实现类；也可以像 Ruby 样，几个类共用一样的方法签名，甚至可以是几个服务响应同一个 REST 接口。

LSP 适用于上述所有的应用场景，因为这些场景中的用户都依赖于一种接口，并且都期待实现该接口的类之间能具有可替换性。想要从软件架构的角度来理解 LSP 的意义，最好的办法还是来看几个反面案例。

## 9.4 Example LSP Violation

Assume that we are building an aggregator for many taxi dispatch services. Customers use our website to find the most appropriate taxi to use, regardless of taxi company. Once the customer makes a decision, our system dispatches the chosen taxi by using a restful service.

Now assume that the URI for the restful dispatch service is part of the information contained in the driver database. Once our system has chosen a driver appropriate for the customer, it gets that URI from the driver record and then uses it to dispatch the driver.

Suppose Driver Bob has a dispatch URI that looks like this:

purplecab.com/driver/Bob

Our system will append the dispatch information onto this URI and send it with a PUT, as follows:

purplecab.com/driver/Bob

/pickupAddress/24 Maple St.

/pickupTime/153

/destination/ORD

Clearly, this means that all the dispatch services, for all the different companies, must conform to the same REST interface. They must treat the pickupAddress, pickupTime, and destination fields identically.

Now suppose the Acme taxi company hired some programmers who didn’t read the spec very carefully. They abbreviated the destination field to just dest. Acme is the largest taxi company in our area, and Acme’s CEO’s ex-wife is our CEO’s new wife, and … Well, you get the picture. What would happen to the architecture of our system?

Obviously, we would need to add a special case. The dispatch request for any Acme driver would have to be constructed using a different set of rules from all the other drivers.

The simplest way to accomplish this goal would be to add an if statement to the module that constructed the dispatch command:

if (driver.getDispatchUri().startsWith("acme.com"))…

But, of course, no architect worth his or her salt would allow such a construction to exist in the system. Putting the word「acme」into the code itself creates an opportunity for all kinds of horrible and mysterious errors, not to mention security breaches.

For example, what if Acme became even more successful and bought the Purple Taxi company. What if the merged company maintained the separate brands and the separate websites, but unified all of the original companies’ systems? Would we have to add another if statement for「purple」?

Our architect would have to insulate the system from bugs like this by creating some kind of dispatch command creation module that was driven by a configuration database keyed by the dispatch URI. The configuration data might look something like this:

| URI | Dispatch Format |
| --- | --- | --- |
| Acme.com | /pickupAddress/%s/pickupTime/%s/dest/%s |
| `*.*` | /pickupAddress/%s/pickupTime/%s/destination/%s |

And so our architect has had to add a significant and complex mechanism to deal with the fact that the interfaces of the restful services are not all substitutable.

违反 LSP 的案例

设我们现在正在构建一个提供出租车调度服务的系统。在该系统中，用户可以通过访问我们的网站，从多个出租车公司内寻找最适合自己的出租车。当用户选定车子时，该系统会通过调用 restful 服务接口来调度这辆车。接下来，我们再假设该 restful 调度服务接口的 UR 被存储在司机数据库中。一旦该系统选中了最合适的出租车司机，它就会从司机数据库的记录中读取相应的 URI 信息，并通过调用这个 URI 来调度汽车。也就是说，如果司机 Bob 的记录中包含如下调度 URI：

那么，我们的系统就会将调度信息附加在这个 URI 上，并发送这样一个 PUT 请求：

很显然，这意着所有参与该调度服务的公司都必须遵守同样的 REST 接口，它们必须用同样的方式处理 pickupAddress、pickupTime 和 destination 字段。

接下来，我们再假设 Acme 出租车公司现在招聘的程序员由于没有仔细阅读上述接口定义，结果将 destination 字段缩写成了 dest。而 Acme 又是本地最大的出租车公司，另外，Acme CEO 的前妻不巧还是我们 CEO 的新欢懂的！这会对系统的架构造成什么影响呢？

显然，我们需要为系统增加一类特殊用例，以应对 Acme 司机的调度请求。而这必须要用另外一套规则来构建。

最简单的做法当然是增加一条 if 语句：

然而很明显，任何一个称职的软件架构师都不会允许这样一条语句出现在自己的系统中。因为直接将「acme」这样的字串写入代码会留下各种各样神奇又可怕的错误隐患，甚至会导致安全问题。比如，Acme 也许会变得更加成功，最终收购了 Purple 出租车公司。然后，它们在保留了各自名字的同时却统一了彼此的计算机系統。在这种情况下，系统中难道还要再增加一条「purple」的特例吗？

软件架构师应该创建一个调度请求创建组件，并让该组件使用一个配置数据库来保存 UR 组装格式，这样的方式可以保护系统不受外界因素变化的影响。例如其配置信息可以如下：

但这样一来，软件架构师就需要通过增加一个复杂的组件来应对并不完全能实现互相替换的 restful 服务接。

## Conclusion

The LSP can, and should, be extended to the level of architecture. A simple violation of substitutability, can cause a system’s architecture to be polluted with a significant amount of extra mechanisms.

LSP 可以且应该被应用于软件架构层面，因为一旦违背了可替換性，该系统架构就不得不为此增添大量复杂的应对机制。