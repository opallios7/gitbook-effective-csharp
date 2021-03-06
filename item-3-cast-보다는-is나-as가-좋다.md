# case보다는 is나 as가 좋다.

C#은 강력한 타입언어다. 특정 타입에서 다른 타입으로의 강제적인 형변환은 가능한 한 피하는 것이 좋다.

```
타입을 변경하는 두가지 방법

1. as 연산자를 이용한 타입변환
2. cast 연산자를 이용한 타입변환

타입변환 가능성 여부 검증을 위한 is 연산자가 존재함.
```

타입변환의 가능성 검증을 위해 is 연산자를 사용하고, 타입변환을 위해 as 연산자를 사용한다면 좀 더 안정적이고 방어적인 프로그램을 작성할 수 있다.

#### as 연산자를 활용한 타입변환 예제

>```
object o = Factory.GetObject();
Mytype t = o as Mytype;
if (t != null)
{
    // Mytype형의 t 객체사용
}
else
{
    // 오류 처리
}
```

### cast 연산자를 활용한 타입변환 예제

>```
try
{
    object o = Factory.GetObject();
    Mytype t;
    t = (Mytype) o;
    if (t != null)
    {
        // Mytype형의 t 객체 사용
    }
    else
    {
        // null인 경우의 오류 처리
    }
}
catch
{
    // 타입변환 오류 처리
}
```

as나 is 연산자는 런타임에 객체의 타입변환이 가능한지를 확인하기 위해서 사용자가 정의한 타입변환 연산자를 전혀 고려하지 않는다. 반면에 cast연산을 수행하는 코드를 사용하게 되면 컴파일러는 객체를 변환 요청된 타입으로 변경할 수 있는 다양한 타입변환 연산자를 고려하여 타입변환을 수행하는 코드를 만들어낸다.

as 연산자를 통한 타입변환이 불가능한 경우도 있다. as 연산자는 value타입에 대해서는 동작을 하지 않기에 다음과 같은 코드는 오류가 발생한다.
```
object o = Factory.GetValue();
int i = o as int;
```
<em><strong><span style="text-color:#FF0000">as 연산자는 타입변환이 실패할 경우 null을 반환하게 되는데 value 타입은 null이 될수 없기때문에 int와 value타입과는 같이 사용될 수 없다.</span></strong></em>


### 타입변환 검증을 위한 is 연산자

is연산자는 null에 대해서는 항상 false를 반환한다.

is 연산자는 as 연산자를 통해서 타입변환이 불가능한 경우에만 사용하는 것이 좋다. 그렇지 않으면 불필요한 중복연산을 처리한다. 다음은 그 예제 코드이다.
```
object o = Factory.GetObject();
Mytype t = null;
if( o is Mytype )
    t = o as Mytype;
```

### foreach루프는 타입변환을 위해 cast연산을 수행
foreach 루프는 value타입과 reference타입 모두에 대해서 동작해야 하므로 cast를 사용해야만 한다. 따라서 내부적인 cast연산이 실패할 경우 foreach 루프는 InvalidCastException이 발생된다.

## 결론
><em><strong> 타입변한은 가능한 한 피하는 것이 좋다. 하지만 때때로 타입변환을 해야 할 때가 있다. 만약 타입변환을 피할 수 없다면, as나 is 연산자를 먼저 사용할 수 있는지를 검토하도록 하자.</strong></em>
