## Item #1.  데이터 맴버 대신에 항상 프로퍼티를 사용하라

닷넷 프레임워크\(.Net Framework\)는 우리가 외부에서 접근 가능한 데이터에 대해서 프로퍼티를 사용할 것이라 가정하고 있다.

웹 컨트롤이나 윈도우 폼 컨트롤과 같은 유저 인터페이스 컨트롤들은 객체의 프로퍼티와 밀접하게 연관되어 있다. 데이터 바인딩 메커니즘은 타입 내의 값에 접근하기 위해 Reflection을 이요하는데, 이때 프로퍼티로 구현된 값만 찾는다.

>```
textBoxCity.DataBindings.Add("Text", address, "City");
```

라고 위의 내용으로 책에 쓰여져있다.

지금까지 습관적으로 내부/외부 변수접근을 구분해서 사용하기 위해 내부는 private의 맴버변수로, 외부는 public의 프로퍼티로 사용해 왔다.

프로퍼티로 사용하면 좋은 점은 인스턴스를 통하여 맴버변수의 값을 가져(<strong>get</strong>)오거나 할당(<strong>set</strong>)할때  Validation을 클래스 내부에서 우선 체크하기에 외부에서는 그것을 신경쓸 필요가 없기 때문이었다.

>```
Publlic class Customer
{
    private string _name;
    public string Name
    {
        get
        {
            retrun _name;
        }
        set
        {
            if ((value == null) || (value.length == 0))
                throw new ArgumentException("Name cannot be blank", "Name");
            _name = value;
        }
    }
}
```

사실 바인딩 개념까지 고려하진 않았었지만 바인딩에서 아주 유용하게 사용이 될 수 있다는 점이 더욱 매력적으로 다가온다.

이것은 C#뿐만 아니라 클래스의 OOP개념을 사용하는 언어에서는 외부와 연결된 부분은 데이터 맴버보다는 프로퍼티 방식으로 사용하는 것이 훨씬 효과적인 것으로 생각한다.
