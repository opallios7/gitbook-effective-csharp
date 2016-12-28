## Item #2 Const 보다는 ReadOnly가 좋다.

C#은 상수형으로 컴파일타임(compile-time)상수와 런타임(Runtime)상수 두 가지를 가지고 있다.

<ul>
    <li>런타임 상수 : readonly</li>
    <li>컴파일타임 상수 : const</li>
</ul>

>```
// 컴파일 타임 상수
public const int _Millennim = 2000;
// 런타임 상수
public static readonly int _ThisYear = 2016;
```

<strong>런타임과 컴파일타임 상수의 근본적인 타이는 값이 치환되는 시점. 즉, 컴파일시점 또는 수행시점에 값이 치환된다.</strong> 

다음코드는 컴파일타임 상수를 초기화했음에도 내장자료형이 아닌 DateTime 타입을 new 연산자로 초기화 하려했기에 컴파일시 오류가 발생한다.

>```
private const DateTime _classCreation = new DateTime(2016, 1, 1, 0, 0, 0);
```

이러한 제약 때문에 컴파일타임 상수는 숫자와 문자열에 한해서만 사용될 수 있다.

readonly 대신 const를 썼을 대의 유일한 장점은 수행성능이다. 이미 알려진 상수값에 직접 접근하는 효율이 readonly로 지정된 변수의 값을 참조하는 것에 비해서 조금 더 빠르다. 그렇지만 이를 통해 얻을 수 있는 수행성능의 개선효과가 작고, 무엇보다 유연성을 감소시키는 단점이있다.

유연성을 포기하기 이전에 수행성능에 미치는 영향을 먼저 고려하여 const를 사용할지 readonly를 사용할 지를 정하는 것이 가장 좋은 프로그래밍 방법일 듯하다.

개인적으로는 readonly를 자주 사용하고 있다. 