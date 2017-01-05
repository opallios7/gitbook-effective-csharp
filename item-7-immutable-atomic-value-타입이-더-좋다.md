# Item #7 immutable atomic value 타입이 더 좋다.

immutable 타입은 간단하다. 이 타입은 생성된 이후에는 마치 상수와 같이 변경될 수 없는 타입이다.
또한 본질적으로 멀티쓰레드에 대해서 안정적으로 동작한다. 다수의 쓰레드가 동시에 접근하더라도 항상 동일한 내용에 접근하게 된다.

모든 타입을 immutable 타입으로 구성하는 것은 좋지 않다. 만일 그렇게 하면 어플리케이션의 상태를 변경하기 위해서 항상 새로운 상태를 가진 객체를 만들어야 한다. 그렇기 때문에 immutable 타입은 단지 atomic value 타입에 대해서만 분리하여 적용하는 것이 좋다.

### mutable한 타입
```
public struct Address
{
    private string _line1;
    private string _line2;
    private string _city;
    private string _state;
    private int _zipCode;
    
    public string Line1
    {
        get { return _line1; }
        set { _line1 = value; }
    }
    
    public string Line2
    {
        get { return _line2; }
        set { _line2 = value; }
    }
    
    public string City
    {
        get { return _city; }
        set { _city = value; }
    }
    
    public string State
    {
        get { return _state; }
        set {
            ValidateState(value);
            _state = value; 
        }
    }
    
    public int ZipCode
    {
        get { return _zipCode; }
        set {
            ValidateZipCode(value);
            _zipCode = value;
        }
    }
    
    //...
}
 
    
// 사용 예
Address a1 = new Address();
a1.Line1 = "111 S. Main";
a1.City = "Anytown";
a1.State = "IL";
a1.ZipCode = 61111;

// 수정
a1.City = "Ann Arbor";        // Zip, State는 유효하지 않다.
a1.ZipCode = 48103;           // State는 유효하지 않다.
a1.State = "MI";              // 모든것이 유효하다.   
```
mutable한 타입의 객체를 생성하여 값을 변경하는 경우 아주 짧은 시간이라 하더라도 객체가 유효하지 않는 상태가 될수 있다.

### immutable한 타입
mutable한 타입을 immutable한 타입으로 변경하기 위해서는 가장 먼저 모든 필드들을 readonly 형태로 변경하고 set accessor를 제거한다.
```
public struct Address
{
    private readonly string _line1;
    private readonly string _line2;
    private readonly string _city;
    private readonly string _state;
    private readonly int _zipCode;
    
    public string Line1
    {
        get { return _line1; }
    }
    
    public string Line2
    {
        get { return _line2; }
    }
    
    public string City
    {
        get { return _city; }
    }
    
    public string State
    {
        get { return _state; }
    }
    
    public int ZipCode
    {
        get { return _zipCode; }
    }
}    
```

이 타입을 실제로 유용하게 만들려면 타입 내부의 필드들을 완벽하게 초기화 할 수 있는 다양한 생성자를 추가하는 것이 좋다. 다음의 코드를 추가한다.
```
public Address(string line1, string line2, string city, string state, int zipCode)
{
    _line1 = line1;
    _line2 = line2;
    _city = city;
    _state = state;
    _zipCode = zipCode;
    
    ValidateState(state);
    ValidateZipCode(zipCode);
}
```
> immutable 타입을 만들때에는 절대로 타입의 내부상테를 변경할 수 있는 여지를 남겨 두어서는 안된다.


### immutable 타입의 복잡성을 피하기 위한 3가지 전략
+ 다양한 생성자를 제공하여 좀 더 쉽게 개체를 생성할 수 있게 한다.
+ factory 메소드를 활용한다. (factory 메소드는 자주 사용되는 일반적인 값을 가진 개체를 좀 더 쉽게 만들 수 있도록 한다.)
+ 객체 생성을 위한 추가 클래스를 제공한다.


## 결론
> immutable 타입은 코드를 좀더 간단하고 유지하기 쉽게 만든다. <em><strong>타입 내에 아무런 의미없이 프로퍼티에 대한 get/set accessor를 만들지 말자.</strong></em> 타입을 생성하는 초기에는 항상 타입이 immutable 타입이 될수 있을지에 대해서 고려해 보아야 한다.

