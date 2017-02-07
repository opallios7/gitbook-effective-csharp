# Item #26 IComparable과 IComparer를 이용하여 순차관계를 구현하라

IComparable은 객체간의 기본적인 순차관계를 정의할 목적으로 사용된다. IComparer는 관계연산(<, >, <=, >=)의 의미를 별도로 정의하거나, 기존의 관계연산이 타입별로 다른 의미를 가질 수 있도록 하면, interface구현에 따른 런타임시 수행성능의 비효율성을 극복하기 위해서 사용될 수 있다.

IComparable interface는 CompareTo()라는 하나의 메소드만을 정의하고 있다. 이 메소드는 C 라이브러리의 strcmp()라는 함수와 유사한 메소드로서 현재 객체가 비교할 객체보다 작을 경우 0보다 작은 값을, 같은 경우 0을, 클 경우 0보다 큰값을 반환하도록 구현해야 한다.

IComparable 구현 예
```
public struct Customer : IComparable
{
    private readonly string _name;
    
    public Customer(string name)
    {
        _name = name;
    }
    
    #region IComparable Members
    int IComparable.CompareTo(object right)
    {
        if(!(right is Customer))
            throw new ArgumentException("Argument not a customer", "right");
        Customer rightCustomer = (Customer)right;
        
        return _name.CompareTo(rightCustomer._name);
    }
    #endregion
    
    public int CompareTo(Customer right)
    {
        return _name.CompareTo(right._name);
    }
}


Customer c1;
Employee e1;
if(( c1 as IComparable).CompareTo(e1) > 0)
    Console.WriteLine("Customer one is greater");
```

IComparable interface를 구현할 때 명시적으로 interface를 구현하되 public 메소드로 적절한 인자형을 받도록 overload 메소드를 제공해라. 이렇게 하면 성능을 개선하는 효과가 있을 뿐만아니라 CompareTo() 메소드를 잘못 사용할 가능성을 줄일수 있다.

## 결론
> IComparable과 IComparer는 타입간의 순차관계를 정의하는 표준화된 기법이다. IComparable은 타입의 일반적인 순차관계를 표현하기 위해서 구현한다. 만일 IComparable을 구현하는 경우라면, IComparable의 CompareTo()와 일관성을 유지하기 위해서 비교 연산자(<, >, <=, >=)도 동시에 overload하는 것이 좋다. 또한 IComaprable.CompareTo()는 System.Object형 인자를 취하므로, 각 타입에 부합하는 CompareTo()메소드를 overload하는 것이 좋다. IComparer는 기본적인 순차관계와는 별도로 다른 형태의 순차관계를 추가로 제공하거나, 순차관계를 제공하지 않는 타입에 대하여 정렬을 수행해야 할 경우 사용될 수 있다.
