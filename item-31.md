# Item #31 작고 단순한 메소드가 더 좋다.

닷넷 런타임은 애플리케이션이 시작되기 이전에 모든 IL코드를 미리 JITing을 수행하지 않고, 메소드 단위로 JITing을 수행한다. 이렇게 함으로써 애플리케이션의 최초 기동시간을 적절한 수준으로 유지할 수 있다. 어떤 메소드든지 JITing이 수행되기 이전에는 호출될 수 없다. 따라서 부가적인 수행을 담당하는 루틴들을 분리하여 메소드를 세분화하는 것이 메소드 호출의 수를 줄이는 방법보다 훨씬 더 효과적이다. 따라서 크기가 작은 메소드가 큰 메소드에 비해 더 좋은 성능을 보인다.

작고 단순한 메소드는 JIT 컴파일러가 enregistration을 좀더 쉽게 할 수 있도록 도와준다. enregistration이란 지역변수를 저장하기 위해서 스택을 사용하지 않고 CPU 내의 레지스터에 위치시키는 것을 말한다. 지역변수 개수를 적게 사용할수록 JIT 컴파일러가 enregistration을 수행하는 코드를 작성할 확률이 높아진다. 수행흐름이 얼마나 단순한가의 정도도 enregistration을 수행하는 코드의 작성여부에 영향을 미치는데, 만일 메소드가 단지 하나의 루프만을 가지는 경우 루프 변수는 레지스터상에 위치될 확률이 높아진다. 하지만 메소드가 다수의 중복 루프를 가지고 있는 경우라면, 어떤 루프 변수를 레지스터상에 배치할 것인가에 대한 어려운 결정을 내려야 할 것이고 이 경우 enregistration이 수행되지 않을 확률이 훨씬 높아진다. 메소드는 단순할수록 더 좋다. 작은 메소드일수록 더욱 더 적은 개수의 지역변수를 사용할 것이고 이점은 JIT컴파일러가 좀 더 쉽게 레지스터를 이용한 최적화 코드를 만들어내는 데 도움을 줄 것이다.

코드를 가능한 간결하게 작성함으로써 JIT 컴파일러가 최적의 결정을 내릴 수 있도록 하는 방법이 유일하다. 크기가 작은 메소드를 구성하면 inlining이 수행될 확률이 높아진다. 여기서 한 가지 주의할 점은 아무리 작은 코드의 메소드라 할지라도 해당 메소드가 virtual로 지정되어 있거나, 내부적으로 try/catch 구문을 포함하는 경우에는 inlining이 수행되지 않는다. 

## 결론
> C# 코드가 실제 수행을 위한 maching 코드로 변경되는 과정은 두 단계를 거친다는 것을 반드시 기억해야 한다. C# 컴파일러는 어셈블리 내에 IL로 구성된 코드를 만들고, JIT 컴파일러는 특정 메소드가 호출되는 시점에 메소드 단위로 IL코드를 maching코드로 만든다. 메소드의 크기가 작다면 JIT 컴파일러에 의해 최적화될 더 많은 기회를 가지게 되며, inlining이 수행될 가능성도 더 많아진다. 추가적으로 단순한 제어흐름을 가진 메소드도 최적화될 기회가 그렇지 않은 메소드에 비해 많은 편이다. 메소드 내의 복잡한 분기를 가지지 않는 메소드들은 지역 변수를 레지스터상에 할당하는 최적화가 적용될 가능성도 높다. 코드를 간결하게 구성해야 함은 물론이고 어떻게 작성하는 것이 좀 더 효율적인지에 대해서도 고려하면서 코드를 작성해야 한다.