# Item #18 표준 Dispose 패턴을 구현하라

표준화된 패턴이란 IDisposable Interface를 구현하여 unmanaged 자원을 해제하고, 개발자들이 Dispose() 메소드의 명시적인 호출을 잊었을 경우를 대비하기 위해서 finalizer에도 적절한 해제 코드를 배치해 두는 것을 말한다. finalizer를 구현할 경우에는 Garbage Collector가 객체를 제거하는 데 있어 수행성능에 나쁜 영향을 주지만, 그럼에도 이러한 방식이 unmanaged 자원을 다루는 가장 유효한 방법이다.

만일 unmanaged 자원을 가지고 있는 클래스들이 상속관계에 있다면 최상위에 있는 클래스는 반드시 자원 해제를 위해서 IDisposable interface를 구현해야 하고 방어적인 코딩을 위해 finalizer도 동시에 구현해야만 한다. Dispose()와 finalizer는 자원의 해제를 위해서 특정 virtual 메소드에 자원해제를 요청하도록 구현하는 것이 좋다. 그리고 자원에 대한 누수가 발생하지 않게 보장할 수 있는 유일한 방법은 적절하게 finalizer를 구현하는 것이다.

IDisposable.Dispose() 메소드를 구현할 때에는 반드시 다음의 4가지 작업을 수행해야 한다.
1. unmanaged 자원의 해제
2. managed 자원의 해제(참조되지 않은 이벤트를 포함해서)
3. 객체가 dispose되었음을 표시하는 상태 플래그 값 설정. Dispose()가 호출된 이후에 사용자가 객체를 사용할 경우를 대비하여 모든 public 메소드에서는 상태 플래그값이 설정되어 있는 경우에 ObjectDisposed 예외를 발생시키도록 작성해야 한다.
4. finalization 동작이 수행되지 못하도록 함. GC.SuppressFinalize(this)를 호출하면 finalization 동작이 일어나지 못하도록 할 수 있다.

Dispose 패턴을 구현하는 간단한 예제
```
public class MyResourceHog : IDisposable
{
  private bool _alreadyDisposed = false;

  // finalizer
  // virtual Dispose 메소드 호출
  ~MyResourceHog()
  {
    Dispose(false);
  }

  // IDisposable interface 구현
  public void Dispose()
  {
    Dispose(true);
    GC.SuppressFinalize(true);
  }

  // virtual Dispose 메소드
  protected virtual void Dispose(bool isDisposing)
  {
    if(_alreadyDisposed)
      return;

    if(isDisposing)
    {
      // To do : managed 리소스를 해제한다.
    }
    // To do : unmanaged 리소스를 해제한다.

    _alreadyDisposed = true;
  }
}
```

만일 하위 클래스에서 추가적인 정리작업을 수행할 필요가 있다면 다음과 같이 protected Dispose(bool) 메소드를 override한다.
```
public class DerivedResourceHog : MyResourceHog
{
  private bool _disposed = false;

  protected override void Dispose(bool isDisposing)
  {
    if(_disposed)
      return;

    if(isDisposing)
    {
        // To do : managed 리소스를 해제한다.
    }
    // To do : unmanaged 리소스를 해제한다.

    base.Dispose(isDisposing);
    _disposed = true;
  }
}
```

<b>Dispose()와 finalizer는 반드시 방어적으로 구현해야 한다.</b>
객체의 Dispose()나 finalizer를 작성할 때 매우 주의를 기울여야 한다. 이러한 코드를 작성할 때 자원의 해제 이외에는 어떠한 다른 동작도 수행해서는 안된다. Dispose()나 finalizer 내에서는 절대 다른 처리 루틴을 포함시키지 말자. 만일 Dispose()나 finalizer에서 다른 처리 루틴을 수행하게 되면, 객체의 생명주기와 관련된 아주 복잡한 문제에 직면할 수 있다. 객체는 우리가 생성하는 시점에 만들어지지만 소멸시점은 Garbage Collector가 결정한다.

## 결론
> Managed 환경에서는 대부분의 경우finalizer를 구현할 필요가 없다. 하지만 특정 타입이 unmanaged 자원을 저장하고 있거나 IDisposable을 구현한 객체를 멤버로 가지고 있는 경우에는 finalizer를 구현하자. 우리가 IDisposable Interface의 구현이 필요하다면 항상 앞서 말한 Dispose 패턴 전체를 구현하자. 이렇게 하면 하위 클래스가 복잡하게 자신의 Dispose와 finalizer를 구현할 필요가 없다. Dispose 패턴을 사용하자. 이것이 우리 뿐만 아니라 우리가 작성한 클래스를 직접 이용하는 사람이나 클래스를 상속받아 사용하는 사람 모두에게 도움이 된다.
>
