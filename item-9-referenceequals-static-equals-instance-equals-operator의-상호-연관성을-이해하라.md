# ReferenceEquals(), static Equals(), instance Equals(), operator==의 상호 연관성을 이해하라

C#은 두 개의 객체가 '동일한가'를 확인하기 위해서 다음의 4가지 메소드를 제공한다.
```
public static bool ReferenceEquals(object left, object right);
public static bool Equals(object left, object right);
public virtual bool Equals(object right);
public static bool operator==(Myclass left, Myclass right);
```

> 위의 4개 메소드들 중 앞쪽 2개의 static 메소드는 절대 재작성하면 안된다. 이유는 해당 메소드의 내부에는 상호연관된 다른 메소드에 영향을 주기 때문이다.

### static Object.ReferenceEquals()
Object.ReferenceEquals()은 두개의 변수가 동일 객체를 참고하고 있는 경우 true를 반환한다. 다시말해 두개의 변수가 단일 객체를 참조하는 동일한 객체 참조자를 담고 있는 경우 true를 반환한다는 것이다. 하지만 value 타입에 대해서는 ReferenceEquals()를 호출하면 항상 false가 반환된다. 설령 하나의 value타입 객체에 대해서 자기자신과 비교를 해도 false를 반환한다. 그 이유는 메소드의 호출시에 value 타입 객체가 reference타입으로 boxing이 일어나기 때문이다.
```
int i = 5;
int j = 5;

if(Object.ReferenceEquals(i, j))
    Console.WriteLine("Never happens.");
else
    Console.WriteLine("Always happens.");
    
if(Object.ReferenceEquals(i, i))
    Console.WriteLine("Never happens.");
else
    Console.WriteLine("Always happens.");
```

### static Object.Equals()
Object.Equals()는 최상위 클래스인 Object.Equals()에 정의되어 있다. 하지만 실제 구현부는 다음과 같이 인자로 전달된 객체에게 동일성을 비교하도록 위임하여 처리한다.
```
public static bool Equals(object left, object right)
{
    if(left == right)
        return true;
    
    if((left == null) || (right == null))
        return false;
    
    return left.Equals(right);
}
```
> Object.ReferenceEquals()와 마찬가지로 Object.Equals()는 런타임에 형식을 알 수 없는 두개의 객체들의 동일성 여부를 확인하는 역할을 정확히 구현하고 있기 때문에 절대로 재정의해서는 안된다.

### virtual Object.Equals()
새로 작성중인 어떤 타입에 대해서 기본적인 Object.Equals()의 동작이 적절하지 않을 경우 재정의를 수행해야 한다. Reference타입의 인스턴스형 Object.Equals()의 기본동작은 Object.ReferenceEquals()와 완전히 동일하다. 그런데 value 타입의 경우에는 그 동작방식이 다른데, 그 이유는 모든 value타입의 공통 상위타입인 System.ValueType이 Object타입의 인스턴스형 Equals()의 기본 동작을 재정의하고 있기 때문이다. 따라서 새로운 value 타입을 만들 때에는 항시 ValueType.Equals()의 구현부를 대체할 수 있는 인스턴스형 Equals()를 재작성하자. Reference 타입의 경우에도 새로 작성하는 클래스가 닷넷 프레임워크 클래스 라이브러리의 규칙을 따르고자 한다면 참조자에 대한 단순비교가 아닌 실제 객체가 담고있는 내용을 기반으로 동일성 여부를 판단할 수 있도록 인스턴스형 Equals() 메소드를 재정의해야만 한다.
```
public override bool Equals(object right)
{
    // null인지 확인
    if(right == null)
        return false;
    
    if(object.ReferenceEquals(this, right)
        return true;
    
    if(this.GetType() != right.GetType())
        return false;
        
    // 내용비교 메소드
    return CompareFooMembers(this, right as Foo);
```
> value 타입을 만들 때는 항상 인스턴스형 Equals()를 재정의하고, Reference타입을 만들때는 System.Object에 정의되어 있는 Equals()의 동작방식을 따르고 싶지 않을 경우에 한해서만 재정의를 수행하자. Equals() 메소드를 재정의할 때는 항상 이러한 규칙을 따르도록 하자.

### Operator==()
value 타입을 정의하고 있는 경우라면 operator==()을 반드시 재정의해야 한다. 그 이유는 value 타입에 대해서 인스턴스형 Equals() 메소드를 재정의해야 하는 이유와 마찬가지로 기본적인 구현부가 성능상의 문제를 야기할 수 있기 때문이다. 정확하게 말하면 value 타입의 Equals()메소드를 재정의하는 경우에만 operator==()를 재정의하는 것이 좋다. 그리고 Reference타입의 경우에는 operator==()를 재정의해야 하는 경우는 거의 없다. 실제로 닷넷 프레임워크의 클래스들은 reference 타입에 대해서는 operator==() 메소드가 값이 아닌 단순 객체 참조자 비교만을 수행하기를 기대한다.

## 결론
> C#언어는 우리에게 4가자의 비교연산 방법을 제공한다. 하지만 이 중 우리가 자체적으로 비교연산을 재정의해야 하는 메소드는 2가지뿐이다. static Object.ReferenceEquals()와 static Object.Equals()는 런타임에 피연산자의 타입을 몰라도 제대로 비교연산을 수행할 수 있도록 잘 정의되어 있다. 우리가 value 타입을 정의하는 경우라면 수행성능의 향상을 위해서 인스턴스형 Equals()메소드와 operator==()를 재정의하자. Reference 타입을 정의하는 경우라면 객체에 대한 참조자를 비교하는 System.Object의 기본동작을 변경해야 할 경우에 한해서 인스턴스형 Equals()만을 재정의하자.






















