# Item #4 #if 대신 Conditional Attribute를 사용하라.

C#에서는 Conditional Attribute가 있는데 이것은 특정 메소드가 환경설정 값에 따라 호출될 수 있는지 여부를 결정하며, #if/#endif를 사용하는 것에 비해 깔끔한 코드를 유지할 수 있도록 도와준다.

#### #if/#endif를 사용한 예
```
private void CheckState()
{
#if DEBUG
    Trace.WriteLine("Entering CheckState for Person");
    
    string methodName = new StackTrace().GetFrame(1).GetMethod().Name;
    Debug.Assert(_lastName != null, methodName, "Last Name cannot be null");
    Debug.Assert(_lastName.Length > 0, methodName, "Last Name cannot be blank");
    Debug.Assert(_firstName != null, methodName, "First Name cannot be null");
    Debug.Assert(_firstName.Length > 0, methodName, "First Name cannot be blank");
    
    Trace.WriteLine("Exiting CheckState for Person");
#endif
}
```

호출하는 부분

```
public string LastName
{
    get
    {
        CheckState();
        return _lastName;
    }
    set
    {
        CheckState();
        _lastName = value;
        CheckState();
    }
}
```
위와 같은 방법으로 사용하게 되면 Release모드로 빌드를 수행할 경우 CheckState()는 아무런 내용도 없는 메소드로 컴파일 된다. 즉, 불필요한 메소드가 생성된 것과 같은 효과이다.

이러한 것을 보완하기 위해 Conditional Attribute를 사용할 수 있다.

#### Conditional Attribute를 사용한 예
```
[Conditional("DEBUG")]
private void CheckState()
{
    Trace.WriteLine("Entering CheckState for Person");
    string methodName = new StackTrace().GetFrame(1).GetMethod().Name;
    Debug.Assert(_lastName != null, methodName, "Last Name cannot be null");
    Debug.Assert(_lastName.Length > 0, methodName, "Last Name cannot be blank");
    Debug.Assert(_firstName != null, methodName, "First Name cannot be null");
    Debug.Assert(_firstName.Length > 0, methodName, "First Name cannot be blank");
    Trace.WriteLine("Exiting CheckState for Person");
}
```

Conditional Attribute는 C# 컴파일러에게 이 메소드는 컴파일러가 DEBUG라는 환경변수 값이 정의되어 있을 때만 호출될 수 있다고 알려준다. Conditional Attribute는 CheckState()메소드의 컴파일과 코드 생성에는 영향을 미치지 않는다. 대신 CheckState()메소드를 호출하는 측의 코드 생성에 영향을 준다. 만일 DEBUG라는 환경변수값이 정의되어 있다면, 컴파일러는 다음과 같은 코드를 생성할 것이다.
```
public string LastName
{
    get
    {
        CheckState();
        return _lastName;
    }
    set
    {
        CheckState();
        _lastName = value;
        CheckState();
    }
}
```
하지만 DEBUG 환경변수가 정의되어 있지 않다면 다음과 같은 코드가 만들어진다.
```
public string LastName
{
    get
    {
        return _lastName;
    }
    set
    {
        _lastName = value;
    }
}
```

##### <em> #if/#endif와 Conditional Attribute를 사용하는 경우의 호출하는 코드부분을 비교해 보면 쉽게 이해가 될 것이라 생각한다. </em>

Conditional Attribute는 한 개 이상의 환경변수와 함께 쓰일 수 있다. 다수의 Conditional Attribute를 쓰게되면 각각의 조건들이 <strong><span style="color:red">'OR 조건'</span></strong>으로 결합된다.
```
[Conditional("DEBUG"), Conditional("TRACE")]
privaite void CheckState()
{
...
}
```

만약 두 개 이상의 환경변수가 동시에 정의되어 있을 경우에 한해서 호출될 수 있도록 조건을 주고 싶다면 다음과 같이 전처리를 활용할 수 있다.
```
#if (VAR1 && VAR2)
#define BOTH
#endif
```

## 결론
><em><strong>Conditional Attribute는 단지 메소드에 대해서만 지정이 가능하다. 추가적인 제한사항으로는 Conditional Attribute를 지정하는 메소드는 반드시 void 형태의 리턴 타입을 가져야 한다는 점과 메소드 내의 일부 문장에 대해서만은 Conditional Attribute를 사용할 수 없다는 점이다.</strong></em>
