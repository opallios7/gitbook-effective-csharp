# Item #13 static 클래스 맴버는 static 생성자를 사용하여 초기화하라.

static 생성자는 접근할 클래스의 어떤 메서드, 변수, 프로퍼티보다 먼저 수행되는 특별한 생성자이다. 우리는 이러한 static 생성자를 이용하여 싱글턴 패턴을 구현할 수도 있고 클래스의 사용 이전에 무엇인가를 수행해야 할 필요가 있을때 활용할 수도 있다. static 변수를 인스턴스형 생성자 내에서 초기화한다거나 초기화만을 위한 특별한 private메서드를 이용하거나 혹은 특수한 초기화 구문 등은 가능한 사용하지 않는것이 좋다.

##### static 생성자는 C#으로 싱글턴(Singleton) 패턴을 구현할 때 자주사용된다.
싱글턴 패턴 구현 예
```
public class MySingleton
{
    private static readonly MySingleton _theOneAndOnly;
    
    static MySingleton()
    {
        _theOneAndOnly = new MySingleton();
    }
    
    public static MySingleton ThenOnly
    {
        get
        {
            return _theOneAndOnly;
        }
    }
    
    private MySingleton()
    {
    }
}
```

선언시 초기화 방법을 사용하면 try/catch문을 이용하여 예외처리를 할 수가 없기에 다음과 같이 생성자에서 처리하는 방법이 좋다.
```
static MySingleton()
{
    try
    {
        _theOneAndOnly = new MySingleton();
    }
    catch
    {
        // 예외처리
    }
}
```

## 결론
> 선언시 초기화를 사용하거나 static 생성자를 사용한 방법은 static 맴버변수를 초기화 할 수 있는 가장 훌륭한 방법이다.
