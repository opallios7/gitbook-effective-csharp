# Item #5 항상 ToString()을 작성하라.

구조체와 같은 타입용도의 클래스를 작성할 때는 ToString()메소드를 만들어주는 것이 좋다. 여러가지 요인이 있겠지만 디버깅을 할 때 가장 많은 도움이 되는 것이 사실이다.

다음과 같은 기본적인 클래스가 있다고 가정하자.
```
public class Customer
{
    private string _name;
    private decimal _revenue;
    private string _contactPhone;
}
```

### 1. 구조가 비교적 단순한 타입 인 경우
위의 Customer타입의 경우 다음과 같이 _name을 반환하도록 만드는 것이 좋다.
```
public override string ToString()
{
    return _name;
}
```
메소드에서 override를 사용하는 이유는 닷넷 프레임워크 클래스 라이브러리는 콤보박스, 리스트박스, 텍스트박스뿐만 아니라 모든 컨트롤들에 대해서 객체의 내용을 출력하기 위해서 Object.ToString()의 override된 버전을 사용하기 때문이다.

### 2. 구조가 복잡한 타입 인 경우
Iformattable.ToString()을 이용하여 좀 더 정교한 출력 결과를 만들어 낼수 있다.

IFormattable.ToString()의 실제 구현 내용을 살펴보자
```
#region IFormattable Members
// format : 
// 이름출력 : n
// revenue 출력 : r
// 전화번호 출력 : p
// nr, np, npr 등과 같이 결합해서 사용할 수도 있다.
// "G"는 기본형태의 출력을 지원한다.
public string ToString(string format, IFormatProvider formatProvider)
{
    if(formatProvider != null)
    {
        ICustomerFormatter fmt = formatProvider.GetFormat(this.GetType()) as ICustomFormatter;
        if(fmt != null)
        {
            return fmt.Format(format, this, formatProvider);
        }
    }
    
    switch(format)
    {
        case "r" :
            return _revenue.ToString();
        case "p" :
            return _contactPhone;
        case "nr" :
            return string.Format("{0, 20}, {1, 10:C}", _name, _revenue);
        case "np" :
            return string.Format("{0, 20}, {1, 15}", _name, _contactPhone);
        case "pr" :
            return string.Format("{0, 15}, {1, 10:C}", _contactPhone, _revenue);
        case "pn" :
            return string.Format("{0, 15}, {1, 20}", _contactPhone, _name);
        case "rn" :
            return string.Format("{0, 10:C}, {1, 20}", _revenue, _name);
        case "rp" :
            return string.Format("{0, 10:C}, {1, 20}", _revenue, _contactPhone);
        case "nrp" :
            return string.Format("{0, 20}, {1, 10:C}, {2, 15}", _name, _revenue, _contactPhone);
        case "npr" :
            return string.Format("{0, 20}, {1, 15}, {2, 10:C}", _name, _contactPhone, _revenue);
        case "pnr" :
            return string.Format("{0, 15}, {1, 20}, {2, 10:C}", _contactPhone, _name, _revenue);
        case "rpn" :
            return string.Format("{0, 10:C}, {1, 15}, {2, 20}", _revenue, _contactPhone, _name);
        caes "rnp" :
            return string.Format("{0, 10:C}, {1, 20}, {2, 20}", _revenue, _name, _contactPhone);
        case "n" :
        case "G" :
        default :
            return _name;
    }
}
#endregion


// 다음과 같이 호출
Customer c1 = new Customer();
// ...
Console.WriteLine("Customer record : {0}", c1.ToString("nrp", null);
```

IFormatProvider에 대한 정의를 해보자.
```
// IFormatProvider 구현
public class CustomerFormatter : IFormatProvider
{
    #region IFormatProvider Members
    // IFormatProvider는 하나의 메서드 만을 가지고 있다.
    // 이 method는 인자로 전달된 interface타입을 구현한 객체를 반환한다.
    // ICustomerFormatter를 구현한 객체를
    // 반환하는 것이 일반적이다.
    public object GetFormat(Type formatType)
    {
        if(formatType == typeof(ICustomFormatter))
            return new CustomerFormatProvider();
        return null;
    }
    #endregion
    
    // Customer Class가 사용자 정의 출력형태를 지웡ㄴ하기 위한 구현한 포함 클래스
    private class CustomerFormatProvider : ICustomFormatter
    {
        #region ICustomFormatProvider : ICustomFormatter
        {
            #region ICustomFormatter Members
            public string Format(string format, object arg, IFormatProvider formatProvider)
            {
                Customer c = arg as Customer;
                if(c == null)
                    return arg.ToString();
                
                return string.Format("{0, 50}, {1, 15}, {2, 10:C}", c.Name, c.ContactPhone, c.Revenue);
            }
        }
        #endregion
    }
}   
```

## 결론
<em><strong>Object.ToString()을 overriding하는 것은 타입에 대한 문자열 출력을 제공하는 가장 간단한 방법이다. 타입을 생성할 때에는 항상 문자열 출력을 제공할 수 있도록 하자. 이렇게 하면 타입의 내용을 좀더 명백히 들여다 볼 수 있도록 하고 타입을 사용할 때에 편의성을 도모할 수 있다. 비교적 드믈기는 하지만 타입에 대한 복잡한 문자열 출력결과를 만들어내고 싶다면 IFormattable Interface를 구현하자. 이것이 타입을 이용할 사용자가 문자열 출력결과를 임의로 변경할 수 있는 표준화된 방법이다.</strong></em>