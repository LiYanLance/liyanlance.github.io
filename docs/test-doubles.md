# 测试中的 mock 们

文章来源: https://blog.cleancoder.com/uncle-bob/2014/05/14/TheLittleMocker.html


```java
interface Authorizer {
  public Boolean authorize(String username, String password);
}
```


### Dummy

如果被测对象依赖 Authorizer, 但这个 test 中并不关心 Authorizer 的行为和结果(函数和返回值)
比如你必须要传入 Authorizer 的实例来构造被测对象, 但是这个实例并 **不会在测试中被使用**. 

我们就可以建一个 **Dummy** Authorizer

```java
public class DummyAuthorizer implements Authorizer {
    public Boolean authorize(String username, String password) {
	    return null;
    }
}
```

当我们要测试 System 的loginCount 方法时:

```java
public class System {
    public System(Authorizer authorizer) {
        this.authorizer = authorizer;
    }

    public int loginCount() {
        //returns number of logged in users.
    }
}

@Test
public void newlyCreatedSystem_hasNoLoggedInUsers() {
    System system = new System(new DummyAuthorizer());
    assertThat(system.loginCount(), is(0));
}
```

注意, Dummy 的 authorize 方法返回了 null.
如果有人试图在测试里使用 Dummy, 会得到 NullPointerException.
因为 **Dummy 并不该被真正的使用**.


#### Dummy 是 mock 吗? 

不严谨的回答, 是的.
mock 这个词, 在日常语境中, 被用于代指各种各样要用于测试的 object.
你可以把任何用于测试的 object 叫做 mock.

但是,但是  
这些用于测试的 object, 其实应该被叫作 Test Doubles.

Test Doubles 来自于 Stunt double (特技替身).

在究竟看什么是 mock 之前, 先看一点其他东西

### Stub

```java
public class AcceptingAuthorizerStub implements Authorizer {
    public Boolean authorize(String username, String password) {
	    return true;
  	}
}
```

这是一个 **总是返回 true 的 test double**. 叫做 stub.

如果你想用来测试用户登录后的一些特性, 就可以用这个. 绕过 authorize 的测试, 不用再在这里测一次. 因为 authorize 方法应该在自己的类中测试, 不应该耦合到这里.

这样我们在写自己的测试时也不用关心 authorize 的实现是不是有 bug.

如果想测试用户未登录的特性, 可以让 authorize 返回 false. 也是一个 stub.
Stub 并不限制返回的值, 但是返回值不会因为函数接收的数据改变.

### Spy

如果我们再加一些东西上去. 比如被测函数是否被调用的标志位.
那我们就得到了一个 spy.

```java
public class AcceptingAuthorizerSpy implements Authorizer {
    public boolean authorizeWasCalled = false;

    public Boolean authorize(String username, String password) {
        authorizeWasCalled = true;
        return true;
    }
}
```

如果你希望明确的知道 authorize 在 caller 中被调用了, 就可以使用 spy.

在测试中 spy 像 stub 一样使用, 只是在测试用例的最后, 会检查 authorizeWasCalled 来确认 authorize 是不是被调用了.

spy 用来**监视 caller**. 能够**记录**各种各样的**信息**. 比如调用的次数.参数. 可以使用 spy 来查看被测算法的内部调用情况.

但这同时就导致了 使用的 spy 越多, 测试和实现的耦合越高.

这会让你的测试非常脆弱, 一些不该导致测试 failed 的代码改动, 也可能会让测试挂掉.

### Mock

真正的 mock
```java

public class AcceptingAuthorizerVerificationMock implements Authorizer {
  public boolean authorizeWasCalled = false;

  public Boolean authorize(String username, String password) {
        authorizeWasCalled = true;
        return true;
  }

  public boolean verify() {
        return authorizedWasCalled;
  }
}
```
Mock 会把 测试的**断言**(assertion) 放在 mock 的函数里(例子中是verify), 也就是说 一个 真正的 mock 明确的**知道自己要测的是什么.**

Mock 测试的是**行为**.  
它不在乎方法的返回值,  
而在乎方法是不是被调用了, 传入的参数是什么, 什么时候调用的, 调用了多少次等等, 这些类似的行为.

mock 是特殊的 spy.

### Fakes

最后一种

```java
 public class AcceptingAuthorizerFake implements Authorizer {
          public Boolean authorize(String username, String password) {
                return username.equals("Bob");
          }
  }
```

Fake 有业务逻辑, 可以用不同的数据, 得到不同的行为.
各种模拟器 就可以归入 fakes.

除了 fake, 其他 test doubles 都不包含业务逻辑.

## 小结

Mock 是一种特殊的 Spy. (带assertion, 明确知道被测的行为是什么)  
Spy 是特殊的 Stub (能够提供调用信息)  
Stub 是 Dummy 的一种. (返回特定的值)  
Dummy 永远不会被 被测对象使用 (返回无意义的东西)

Fake 带业务逻辑, 其他各种 test doubles 都不带.


#### 深入阅读 
[Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)
