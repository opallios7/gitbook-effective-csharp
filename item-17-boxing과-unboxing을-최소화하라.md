# Item \#17 boxing과 unboxing을 최소화하라

boxing이란 value타입 객체를 타입이 구체적으로 정의되지 않은 reference 타입 객체 내부로 포함하여, reference 타입이 요구되는 경우에도 value 타입을 쓸 수 있도록 해주는 메커니즘을 말한다. unboxing이란 boxing되어 있는 value 타입 객체의 복사본을 획득하는 과정을 말한다. boxing과 unboxing은 value 타입을 System.Object 타입이 필요한 시점에 사용하기 위해서 반드시 필요하다. 하지만 boxing과 unboxing은 항상 수행성능에 좋지 않은 영향을 미친다. boxing과 unboxing 동작은 항상 복사과정을 수반하기 때문에, 때때로 이로 인해 기대하지 않은 오류가 발생하기도 한다. boxing과 unboxing은 피할 수 있다면 가능한 한 피하는 것이 좋다.

boxing의 예

```
int i = 25;
object o = i;
Console.WriteLine(o);
```

unboxing의 예

```
object o;
int i = (int) o;
string output = i.ToString();
```

## 결론

> 가능한 한 boxing이 일어나는 것을 피하자. value 타입은 System.Object나 interface에 대한 reference로 boxing될 수 있다. 이러한 변경은 내부적으로 일어나기 때문에 어디에서 boxing이 일어나고 있는지를 찾기란 쉽지않다. 이것은 닷넷 프레임워크라는 환경과 언어의 특성이므로 피할 수 없다. 또한 boxing과 unboxing은 우리가 원하지 않도라도 복사를 일으키고 때때로 버그의 원인이 되기도 하며 성능상의 문제점을 유발한다. value타입을 collection에 삽입하려고 하거나, value타입에 대하여 System.Object에서 정의한 메소드를 호출하거나 또는 System.Object로 casting을 수행하는 등의 동작은 모두 boxing을 유발시킨다. 이러한 boxing은 피할 수만 있다면 가능한 한 피하는 것이 좋다.



