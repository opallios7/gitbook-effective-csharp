# Item #11 foreach 루프가 더 좋다.

C#의 foreach문은 do, while, for를 이용한 루프보다 좀 더 다양하게 쓸 수 있으며 어떠한 collection에 대해서도 최고의 순회 코드를 만들어 낼 수 있다. 이 구문은 닷넷 프레임워크 내의 collection interface와 매우 밀접하게 연관되어 있으며 C# 컴파일러는 서로 다른 collection 타입을 사용할 경우라도 항상 최적의 코드를 생성한다. collection에 대한 순회를 수행할 때는 다른 루프 구분을 사용하지 말고 항상 foreach를 사용하자. 다음의 예제를 살펴보자.

```
int[] foo = new int[100];

// 루프1
foreach(int i in foo)
    Console.WriteLine(i.ToString());
    
// 루프2
for(int index = 0; index < foo.Length; index++)
    Console.WriteLine(foo[index].ToString());
    
// 루프3
int len = foo.Length;
for(int index = 0; index < len; index++)
    Console.WriteLine(foo[index].ToString());
```

최소한 닷넷 프레임워크 1.1이상에 포함된 모든 C# 컴파일러에 대해서는 루프 1번이 가장 짧은 코드를 가지고 있고 가장 빠른 속도를 보인다.(닷넷 프레임워크 1.0의 C# 컴파일러는 루프 2번이 가장 빠르고 루프 1번이 가장 느리다.) 대부분의 C/C++ 개발자들은 루프 3번이 가장 효율적일 것이라고 생각하지만, 실제로는 위의 예제 중 가장 좋지 않은 방법이다. Length 프로퍼티를 루프 바깥쪽으로 빼버리면 JIT 컴파일러가 루프 내부에서 배열의 범위를 확인하는 코드를 제거할 수 있는 기회를 잃어버리게 된다.

C#코드는 안전하게 관리되는 환경하에서 수행된다. 배열의 인덱스를 포함하여 모든 메모리 위치 정보는 사전에 검사되어야 한다. 이 때문에 루프 3번은 다음과 같은 구조로 컴파일 된다.

```
// 루프 3번이 컴파일러에 의해 생성된 코드
int len = foo.Length;
for(int index = 0; index < len; index++)
{
    if(index < foo.Length)
        Console.WriteLine(foo[index].ToString());
    else
        throw new IndexOutOfRangeException();
}
```

## 결론
> foreach는 매우 융통성 있는 언어요소다. 배열의 최소, 최대 범위를 자동으로 결정하여 올바르게 코드를 만들어 주며 다차원 배열에 대해서도 활용이 가능할 뿐만 아니라, 참조되는 타입이 항상 올바른 타입이 될 수 있도록 처리해 준다(가장 효율적인 구성을 위해서). 그리고 무엇보다도 루프에 대한 가장 효율적인 코드를 만들어낸다. 이는 collection 순회를 위한 최고의 방법이다. 또한 foreach를 사용하면 코드의 변화에 강하기 때문에 전체 코드를 뒤지면서 수정해야 하는 수고를 덜어주기에 생산성에 많은 도움이 된다.