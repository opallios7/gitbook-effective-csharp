# Item #16 Garbage를 최소화하라

Garbage Collector는 우리를 대신하여 메모리 관리를 훌륭하게 수행해 준다. 매우 효율적인 방법으로 더 이상 사용되지 않는 객체를 삭제하지만 그 어떠한 방법을 쓴다 해도 힙을 기반으로 하는 객체를 생성하고 파괴하는 동작은 하지 않는 것에 비한다면 여전히 많은 프로세서 시간을 소비하다. Garbage Collector를 너무 혹사시키지 말자.
그럼 Garbage Collector를 최소화기 위해방법을 다음의 방법을 제시한다.

+ 빈번하게 사용하는 지역변수는 맴버변수로
+ 공통적으로 사용하는 객체들은 싱글턴 형태로
+ Immutable 타입을 생성하기 위해 Builder클래스를 사용

### 빈번하게 사용하는 지역변수는 맴버변수로

OnPaint()와 같이 매우 빈번하게 호출되는 메소드의 내부에서 사용하는 지역변수는 맴버변수로 선언하여 사용하는게 좋다. 다음의 예제를 보자.

내부 변수 사용
```
protected override void OnPaint(PaintEventArgs e)
{
    // Paint 이벤트가 발생될때마가 동일 폰트를 매번 생성한다.
    using(Font myFont = new Font("Arial", 10.0f))
    {
        e.Graphics.DrawString(DateTime.Now.ToString(), myFont, Brushes.Black, new PointF(0, 0));
    }
    base.OnPaint(e);
}
```

맴버변수로 변경해서 사용
```
private readonly Font _myFont = new Font("Arial", 10.0f);

protected override void OnPaint(PaintEventArgs e)
{
    e.Graphics.DrawString(DateTime.Now.ToString(), myFont, Brushes.Black, new PointF(0, 0));
    
    base.OnPaint(e);
}
```
위와 같이 수정하면 Paint 이벤트 처리시마다 발생하는 garbage를 완전히 제거할 수 있으므로 Garbage Collector는 훨씬 더 적을 일을 수행해도 된다. 다시 말해 reference 타입의 지역변수를 포함하고 있는 메소드가 반복적으로 자주 호출된다면 지역변수를 맴버변수로 변경하는 것이 좋다. 하지만 빈번하게 호출되는 메소드 내의 지역변수이거나, 반복적으로 호출되는 메소드 내의 지역변수에 대해서만 맴버변수로의 변경을 수행해야 한다. 절대로 다른 모든 지역변수를 맴버변수로 변경하지 마라.

### 공통적으로 사용하는 객체들은 싱글턴 형태로
위의 코드에서 사용된 Brushes.Black와 같은 static 프로퍼티는 동일 객체에 대한 반복적인 생성을 피할 수 있는 방법을 보여주고 있다. 매우 일반적으로 사용되는 reference타입 객체는 static 맴버변수로 변경하는 것이 좋다.

Brushes클래스는 내부적으로 특정 Brush에 대한 요청이 있을 경우에만 객체를 생성하도록 lazy evaluation 알고리즘을 이용하여 구현되어 있다. 다음의 코드를 보자
```
private static Brush _blackBrush;
public static Brush Black
{
    get
    {
        if (_blackBrush == null)
            _blackBrush = new SolidBrush(Color.Black);
        
        return _blackBrush;
    }
}
```
위의 코드를 보면 Black Brush가 처음으로 요청될 때만 브러시를 생성하는 것을 알수 있다. Brushes객체는 단지 하나의 검정브러시에 대한 reference만을 유지하고 있으면서 사용자의 요청시마다 동일한 Brush객체를 반환한다.

### Immutable 타입을 생성하기 위해 Builder클래스를 사용

System.String 객체는 immutable 타입으므로 객체가 생성된 이후에는 그 내용을 변경할 수 없다. 따라서 string 객체가 포함하는 문자열의 내용을 수정하려고 시도하면 실제로 새로운 객체가 생성되고 이전 객체는 garbage화 된다. 다음과 같은 코드가 있다고 가정하자.

```
string msg = "Hello, ";
msg += thisUser.Name;
msg += ". Today is ";
msg += System.DateTime.Now.ToString();
```

위의 코드는 내부적으로는 다음과 같이 수행된다.

```
string msg = "Hello, ";
// 이해를 돕기 위한 코드이다. 컴파일 되지는 않는다.
string tmp1 = new String(msg + thisUser.Name);
msg = tmp1; // "Hello, "는 gerbage가 된다.
string tmp2 = new String(msg + ". Today is ");
msg = tmp2; // "Hello <user>"는 garbage가 된다.
string tmp3 = new String(msg + DateTime.Now.ToString());
msg = tmp3; // "Hello <user>. Today is "는 garbage가 된다.
```

위와 같이 발생되는 garbage를 줄이기 위해서는 다음과 같이 string.format을 사용하면 된다.

```
string msg = string.Format("Hello, {0}. Today is {1}", thisUser.Name, DateTime.Now.ToString());
```

좀더 복잡한 문자열을 생성해야 하는 경우 StringBuilder 클래스를 활용한다.

```
StringBuilder msg = new StringBuilder("Hello, ");
msg.Append(thisUser.Name);
msg.Append(". Today is");
msg.Append(DateTime.Now.ToString());
string finalMsg = msg.ToString();
```


## 결론
> Garbage Collector는 어플리케이션이 사용하는 메모리를 효율적으로 관리해주는 역할을 수행한다. 하지만 힙에 객체를 생성하고 삭제하는 과정은 시간을 필요로 한다. 필요하지 않는 객체를 과도하게 생성하는 것을 피하라. 함수 내에 reference 타입의 지역변수를 반복적으로 생성하지 않도록 하기 위해서 맴버변수로의 변경을 고려하고, 공통으로 사용하는 객체에 대해서는 static변수로 선언하자. 마지막으로 immutable타입을 구성하고 있는 경우라면 mutable builder 클래스를 제공할 것인지를 고려해보자.


