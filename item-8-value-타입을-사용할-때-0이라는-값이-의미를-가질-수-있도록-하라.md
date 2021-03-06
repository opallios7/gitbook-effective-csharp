# Item #8 Value 타입을 사용할 때 0이라는 값이 의미를 가질 수 있도록 하라

기본적으로 닷넷 시스템은 value 타입으로 인스턴스를 만들 때 '0'으로 값을 초기화한다. 우리는 이러한 초기화를 막을 방법이 없다. 따라서 가능하다면 '0'이라는 값을 의미있는 유효한 값으로 활용하는 것이 좋다. 하지만 예외적으로 enum과 같은 자료형에서는 '0'을 의미있는 값으로 활용것이 좋지 않을 수 있다.

### Enum타입 자료형에서의 정의
enum과 같은 자료형에서는 의미값으로 '0'을 사용하지 말자. enum타입은 System.ValueType을 상속하고 있으며 각 항목은 항상 '0'으로부터 시작하는 값을 가지게 되지만 다음과 같이 각 항목이 가지는 값에 수정을 가할 수도 있다.

```
public enum Planet
{
    // 명시적으로 값 할당
    // 지정하지 않으면 기본 시작 값은 0
    Mercury = 1,
    Venus = 2,
    Earth = 3,
    Mars = 4,
    Jupiter = 5,
    Saturn = 6,
    Neptune = 7,
    Uranus = 8,
    Pluto = 9
}

// ...
Planet sphere = new Planet();
```
위 예제에서 sphere의 값은 '0'이 되는데 이 값은 Planet에서 정의한 유효한 값의 범위 내에 포함되어 있지 않다. enum형을 선언하는 이유는 사용할 수 있는 값의 범위를 제한하기 위함인데, 이 경우에는 정의되지 않은 값인 '0'을 가지게 되므로 잘못된 초기화라 할 수 있다. 그리고 이러한 enum타입을 비트연산에 활용해야 한다면 '0'값은 '다른 어떤 유효한 값도 가지지 않았음'의 의미로 사용한 것이 좋다. 또한 enum타입으로 인스턴스를 생성할 때에는 다음과 같이 반드시 명시적으로 값을 초기화하는 코드를 쓰는 것이 좋다.
```
Planet sphere = Planet.Mars;
```

### 명시적으로 값을 초기화 할 수 없는 상황
만약, 명시적으로 값을 초기화할 수 없는 상황이 발생하는 경우가 있다면 enum 타입을 정의할 때 다음과 같이 '0'이라는 값의 의미를 부여하되 '다른 어떤 유효한 값도 가지지 않음'의 의미로 정의한다.
```
public enum Planet
{
    None = 0,
    Mercury = 1,
    Venus = 2,
    Earth = 3,
    Mars = 4,
    Jupiter = 5,
    Saturn = 6,
    Neptune = 7,
    Uranus = 8,
    Pluto = 9
}

// ...
Planet sphere = new Planet();
```

### Enum을 플래그로 사용할 경우
enum 타입을 플래그로 사용할 경우 반드시 알아두어야 하는 규칙이 있다. enum형 타입을 비트연산에 활용하기 위해서 Flags라는 attribute를 사용할 수 있는데 이 때에도 반드시 '0'값을 None으로 선언해 주는 것이 좋다.
```
[Flags]
public enum Styles
{
    None = 0,
    Flat = 1,
    Sunken = 2,
    Raised = 4
}
```
많은 개발자들이 flag속성을 부여한 enum 값에 대해서 AND나 OR연산을 자주 수행하게 되는데 enum의 특정 항목이 '0'의 값을 가지는 경우 예기치 않은 심각한 문제를 일으키기도 한다. 다음의 예제는 Flat이라는 값이 '0'으로 정의되어 있는 경우(위의 enum 정의에서 None을 정의하지 않고, 각 항목에 명시적인 값을 주지 않으면 Flat은 '0'이 된다)라면 제대로 동작하지 않을 것이다.
```
// Styles.Flat이 0이면 결코 true가 될 수 없다.
if((FlagsAttribute & Styles.Flat) != 0)
    DoFlatThings();
```
Flat attribute를 사용하는 경우에도 '0'이라는 값을 유효한 값으로 정의하되 '다른 어떤 유효한 flag로 지정되지 않음'의 의미로 사용하도록 하자.

## 결론
> 닷넷 시스템은 value타입의 인스턴스를 항상 '0'으로 초기화한다. 사용자가 value타입을 활용함에 있어 이러한 초기화 자체를 명시적으로 막을 수 있는 방법은 없다. 따라서 <em><strong>가능한 '0'이라는 값을 유효한 값의 범주로 포함시키고, '다른 어떤 유효한 값도 가지지 않음을 의미'를 유지</strong></em>할 수 있도록 하자.
