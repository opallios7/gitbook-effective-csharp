# Item #20 interface의 구현과 virtual 메소드의 overriding을 구분하라

interface를 구현하는 것은 virtual 메소드를 overriding을 하는 것과 유사해 보인다. 다른 타입에서 선언한 멤보를 구현한다는 측면에서 유사하기 때문에 가끔은 혼란스럽기도 하다. 하지만 interface의 구현은 virtual 메소드에 대한 overriding과는 매우 다르게 동작한다. interface내에 선언한 맴버는 virtual이 아니다. 따라서 상위 클래스에서 이미 특정 interface를 구현해 두었다면 하위 클래스는 해당 interface 맴버를 overriding할 수 없다. 하위 클래스에서도 동일한 interface를 구현할 수 있으나 이는 상위 클래스에서 구현한 것을 대체하게 된다. 이것이 virtual 메소드에 대한 overriding과 다른 개념이다.

다음과 같은 인터페이스와 상위 클래스, 하위 클래스가 있다고 하자.
```
interface IMsg
{
    void Message();
}

public class MyClass : IMsg
{
    public void Message()
    {
        Console.WriteLine("MyClass");
    }
}

public class MyDerivedClass : MyClass
{
    public new void Message()
    {
        Console.WriteLine("MyDerivedClass");
    }
}
```

Message()라는 메소드의 접근은 다음과 같이 실행된다.

```
MyDerivedClass d = new MyDerived Class();
d.Message();        // "MyDerivedClass"의 Message가 실행된다.
IMsg m = d as IMsg;
d.Message();        // "MyClass"의 Message()가 실행된다.
```

위와같이 interface를 통해 접근을 하게되면 상위 클래스의 메소드가 접근된다. 왜? 앞에서 말한 interface는 virtual가 아니기 때문이다. interface는 일종의 약속이기 때문에 interface에 선언된 모든 요소에 대해 필요한 클래스에서 구현해야 한다. 

그렇다면, 상위 클래스가 아닌 하위 클래스로 접근을 하려면 어떻게 해야 할까? 다음과 같이 interface를 구현하고 IMsg interface를 통해 Message()를 호출하면 하위 클래스의 Message가 실행된다.
```
public class MyDerivedClass : MyClass, IMsg
{
    public new void Message()
    {
        Console.WriteLine("MyDerivedClass");
    }
}
```

명시적으로 interface를 구현한다는 것은 구현되지 않은 interface를 새롭게 구현하는 것이 일반적이지만 때로는 이미 구현된 것을 대체하는 경우도 있다. 이러한 두가지 차이점이 때로는 interface를 구현하는 것과 virtual 메소드를 overriding하는 것을 혼란스럽게 만들기도 한다. interface를 선언하고 구현하는 이유는 사용자가 타입을 사용할 때 좀더 적절한 동작방식을 규정하기 위해서라고 할 수 있다.

## 결론
> interface는 virtual메소드가 아니라 타입이 구현해야 할 약속이기 때문에 interface를 이미 구현하고 있는 클래스를 상속받아 새로 구현을 하고 싶을 때는 언제, 어떻게 구현할 것인지를 좀 더 명확히 해야 한다. 