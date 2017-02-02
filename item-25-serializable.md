# Serializable 타입이 더 좋다.

serialization은 타입의 핵심적인 기능 중 하나이다. 따라서 우리가 만드는 타입 중에 UI 전용 컨트롤, 윈도우 혹은 폼을 가지는 타입을 제외한 모든 타입은 반드시 serialization기능을 제공해야 한다. 추가적으로 개발해야 하는 코드가 많지 않기 때문에 serialization 기능은 반드시 제공해야 하는 것이 좋다. 많은 경우 Serializable attribute만을 추가하는 것으로 충분하다.

다음은 Serializable을 사용하는 예제이다.

```
[Serializable]
public class MyType
{
    private string _label;
    private int _value;
}
```

닷넷 serialization은 객체 내의 모든 맴버변수를 출력 스트림으로 내보낼 수 있다. 또한 닷넷 seiralization은 포함하는 모든 객체들에 대해서 객체 그래프를 구성한다. 설사 순환참조가 있는 경우라도 serialize와 deserialize 메소드를 통해 정확하게 객체를 저장하고 복원할 수 있다. 또한 닷넷 프레임워크의 serialization 구현부는 객체가 deserialize 될 때 객체간의 복잡한 참조관계를 재구성한다. 아무리 복잡한 참조관계가 있다 하더라도 항시 정확하게 객체를 재생성 할 수 있다. 마지막 주요특징은 Serializable attribute가 지정된 모든 객체는 바이너리와 SOAP형태로 serialization 될 수 있다는 것이다. 

serializable attribute를 추가하는 것은 serialize 가능한 객체를 만드는 가장 간단한 방법이다. 하지만 가장 간단한 방법이 항상 최선의 방법이라고 말하기는 힘들다. 어떤 경우네는 객체내의 모든 맴버에 대해서 serialization이 필요한 것은 아니다. 그러므로 Serializable attribute가 추가된 타입이라 하더라도 멤버를 각각에 대해서 serialization 여부를 독립적으로 지정할 수 있어야 한다. serialization을 원하지 않는 멤버에 대해서는 NonSerialized attribute를 사용하여 특정 멤버를 serialize 과정에서 재베시킬 수 있다.

다음은 NonSerialized를 사용하는 예제이다.

```
[Serializable]
public class MyType
{
    private string _label;
    private int _value;
    
    [NonSerialized]
    private int _cachedValue;
    
    private OtherClass _object;
}
```

serializ된 데이터는 프로그램의 버전이 변경되더라도 계속 사용될수 있어야 한다. 이전 버전에서 serialize된 데이터는 새로운 버전의 프로그램에서도 여전히 읽을 수 있어야 한다. 만일 여러개의 버전에서도 수행될 수 있는 serializ가능한 타입을 구성하려 하는 경우라면, serialization 처리를 위해서 ISerializable interface를 구현하여 좀 더 세밀한 제어를 수행해야 한다. ISerializable interface에 의해 처리되는 메소드나 저장소와 관련된 부분은 Serializable attribute를 사용하는 경우와 일치하기 때문에, Serializable attribute를 이용하던 타입에 대하여 추가적인 serialization 동작이 필요한 경우에도 특별한 변경없이 ISerializable interface를 구현하기만 하면 된다.

다음의 예제에 나오는 serialization 생성자의 구현 예제는 Serialization attribute를 지정하여 자동으로 생성되는 코드와 함께 이전 버전과 최신 버전의 MyType 객체를 동시에 가져오는 방법을 보여준다.

```
using System.runtime.Serialization;
using System.Security.Permissions;

[Serializable]
public sealed class MyType : ISerializable
{
    private string _label;
    private int _value;
    
    [NonSerialized]
    private int _cachedValue;
    
    private OtherClass _object;
    
    private const int DEFAULT_VALUE = 5;
    private int _value2;
    
    private MyType(SerializationInfo info, StreamingContext cntxt)
    {
        _label = info.GetString("_label");
        _object = (OtherClass)info.GetValue("_object", typeof(OtherClass));
        try
        {
            _value2 = info.GetInt32("_value2");
        }
        catch(SerializationException e)
        {
            _value2 = DEFAULT_VALUE;
        }
    }
    
    [SecurityPermission(SecurityAction.Demand, SerializationFormatter = true)]
    void ISerializable.GetObjectData(SerializationInfo inf, StreamingContext ctx)
    {
        inf.AddValue("_label", _label);
        inf.AddValue("_object", _object);
        inf.AddValue("_value2", _value2);
    }
}
```

serialization 스트림으부터 값을 저장하고 획득하는 순서는 항상 일치하여야 한다. 일고 쓰는 코드의 순서가 상속 관계 전반에 걸쳐서 동일한 순서를 유지하지 않는다면 serialization은 정상적으로 동작하지 않을 것이다.

## 결론
> 닷넷 프레임워크는 객체의 serialization을 위한 쉽고 표준화된 방법을 제공한다. 만일 객체가 영속성을 가져야 한다면 이러한 표준화된 방법을 이용하는 것이 좋다. 만일 우리가 작성하는 타입이 serialization을 지원하지 않는다면, 이 타입을 사용하는 클래스들은 serialization을 지원하지 않을 수도 있다. 가능한 다른 사용자들이 쉽게 serialization을 구현할 수 있도록 모든 타입을 serialization이 가능하도록 만들고, 기본적으로 제공되는 기능을 이용하되 기본기능이 충분하지 않을 경우에는 ISerializable을 구현하자.
