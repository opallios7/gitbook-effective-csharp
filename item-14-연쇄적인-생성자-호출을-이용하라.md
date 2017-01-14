# Item #14 연쇄적인 생성자 호출을 이용하라

C# 프로그래밍을 하는 동안에 다수의 생성자 내에서 중복적으로 수행되고 있는 코드를 발견했을 때, 중복코드들을 새로운 생성자로 분리시켜 호출하는 방식으로 초기화를 수행하도록 하자. 이렇게 함으로써 코드의 중복을 피할 수 있을 뿐만 아니라 좀 더 효율적인 코드를 구성할 수 있다. C# 컴파일러는 생성자에서 또 다른 생성자를 호출하는 구조를 특수한 문법의 일부로 인지하고, 변수에 대한 초기화나 기반 클래스의 생성자 호출 등의 중복 코드를 제거한다. 이런 방법을 통해 객체의 초기화를 위해 필요한 최소한의 코드만이 수행될 수 있도록 최적화를 수행한다.

다음 구현의 예를 보자.

```
public class MyClass
{
    private ArrayList _coll;
    private string _name;
    
    public MyClass() : this(0, "")
    {}
    
    public MyClass(int initialCount) : this(initialCount, "")
    {}
    
    public MyClass(ini initialCount, string name)
    {
        _coll = (initialCount > 0) ? new ArrayList(initialCount) : new ArrayList();
    }
}
```

위의 코드를 컴파일하면 다음과 같은 코드처럼 컴파일 된다.

```
public class MyClass
{
    private ArrayList _coll;
    private string _name;
    
    public MyClass()
    {
        this(0, "");
    }
    
    public MyClass(int initialCount)
    {
        this(initialCount, "");
    }
    
    public MyClass(int initialCount, string name)
    {
        object();
        this(initialCount, name);
    }
    
```

C#은 기본 인자값을 가진 메소드를 언어차원에서 문법적으로 제공하지 않기 때문에, 서로 다른 인자의 개수를 가진 생성자를 각각 정의해 주어야 한다. 이것은 코드의 중복이 일부 발생할 수 있음을 의미한다. 하지만 그렇다 하더라도 아래의 코드와 같이 중복코드를 포함하는 독립된 메소드를 정의하여 생성자 내에서 호출하도록 코드를 구현하는 것이 좋지 않은 방법이다. 이 경우 일부 비효율성이 발생할 소지가 있으므로 가능한 생성자를 연쇄적으로 호출하는 구조로 코드를 작성하는 것이 좋다. 

다음의 예는 인자값을 전달하지 않고 사용방식의 코드이다. (권장하지는 않는다.)

```
public class MyClass
{
    private ArrayList _coll;
    private string _name;
    
    public MyClass()
    {
        commonConstructor(0, "");
    }
    
    public MyClass(int initialCount)
    {
        commonConstructor(initialCount, "");
    }
    
    public MyClass(int initialCount, string name)
    {
        commonConstructor(initialCount, name);
    }
    
    private void commonConstructor(int initialCount, string name)
    {
        _coll = (initialCount > 0) ? new ArrayList(initialCount) : new ArrayList();
        _name = name;
    }
}
```

위의 코드를 컴파일하면 각 생성자에 객체 인스턴스 초기화 루틴이 반복생성된다. 즉, 중복된 코드가 발생한다는 의미이다.

```
public class MyClass
{
    private ArrayList _coll;
    private string _name;
    
    public MyClass()
    {
        this(0, "");
    }
    
    public MyClass(int initialCount)
    {
        this(initialCount, "");
    }
    
    public MyClass(int initialCount, string name)
    {
        object();
        this(initialCount, name);
    }
    
```

> <b> 생성자 연쇄 호출방식을 사용할 때는 this()구문을 이용한다. 기반 클래스의 생성자를 호출하려면 base()구문을 이용하면 된다. 하지만 두가지를 동시에 사용할 수는 없다.

C# 언어에서는 아주 간단한 클래스를 제외한다면 대부분의 클래스들은 하나 이상의 생성자를 가지고 있고, 이러한 생성자들의 주된 역할이 객체의 맴버들을 초기화하는 것이기 때문에 생성자의 연쇄 호출방식을 이용하여 객체 맴버들을 초기화하는 것은 매우 자연스럽다. 코드의 중복을 피하고 효율성을 높이기 위하여 반드시 생성자 연쇄 호출방식을 사용하도록 하자.

다음은 인스턴스 과정에서 내부적으로 수행되는 모든 동작들에 대한 순서를 나열한 것이다.
> 1. static 변수를 0으로 설정한다.
2. static 변수에 대하여 선언시 초기화 루틴을 수행한다.
3. 기반 클래스의 static 생성자를 수행한다.
4. static 생성자를 호출한다.
5. 인스턴스 변수를 0으로 설정한다.
6. 인스턴스 변수에 대하여 선언시 초기화 루틴을 수행한다.
7. 기반 클래스의 인스턴스 생성자를 수행한다.
8. 인스턴스 생성자를 호출한다.

## 결론
> C# 컴파일러는 객체가 생성될 때 객체의 모든 요소들이 초기화될 것임을 보장한다. 그러므로 아무런 추가 동작 없이도 최소한 객체가 점유하고 있는 메모리 공간은 모두 0으로 초기화될 것이고, 이는 static 맴버와 인스턴스 맴버를 구분하지 않는다. <b>간단한 리소스는 선언시 초기화를 사용하고 복잡한 초기화 과정이 필요하면 생성자를 사용하자. 그리고 생성자의 연쇄호출을 사용할 수 있도록 하자. 이렇게 함으로써 중복적으로 초기화가 일어나는 것을 방지할 수 있다.</b>
