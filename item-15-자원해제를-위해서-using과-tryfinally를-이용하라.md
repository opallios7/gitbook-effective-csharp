# Item #15 자원해제를 위해서 using과 try/finally를 이용하라

Unmanaged 자원들을 사용하는 타입들은 IDisposable interface의 Dispose()메소드를 호출하여 명시적으로 자원을 해제해 주어야 한다. 닷넷 환경에서의 이러한 책임은 타입 자체나 시스템의 책임이 아니라 타입을 사용하는 사용자의 책임이다. Dispose()가 항상 호출될 수 있도록 코드를 작성하기 가장 쉬운 방법은 using문장과 try/finally 블록을 사용하는 것이다.

다음과 같은 코드가 있다고 가정하자.
```
public void ExecuteCommand(string connString, string commandString)
{
    SqlConnection myConnection = new SqlConnection(connString);
    SqlCommand mySqlCommand = new SqlCommand(commandString, myConnection);
    
    myConnection.Open();
    mySqlCommand.ExecuteNonQuery();
}
```

위의 코드를 객체 생성시에 using구문을 사용하면 Dispose()메소드가 자연스럽게 호출된다.
```
public void ExecuteCommand(string connString, string commandString)
{
    using(SqlConnection myConnection = new SqlConnection(connString))
    {
        using(SqlCommand mySqlCommand = new SqlCommand(commandString, myConnection))
        {
            myConnection.Open();
            mySqlCommand.ExecuteNonQuery();
        }
    }
}
```

IDisposable interface를 구현한 객체의 Dispose()메소드가 호출되도록 하는 가장 간단한 방법은 using 구문을 사용하는 방법이다. 하지만 만일 IDisposable interface를 구현하지 않은 타입을 using 구문과 함께 사용하면 C# 컴파일러는 컴파일 오류를 발생한다.
```
// object가 IDisposable을 구현하지 않은 경우 컴파일 되지 않는다.
using (object obj = Factory.CreateResource())
    Console.WriteLine(obj.ToString()); 
```

특정 타입이 IDisposable interface를 구현하고 있는지의 여부를 고려하지 않고 using구문을 사용하고 싶을 경우에는 다음과 같은 방어적인 코딩이 도움이 될 수 있다.
```
object obj = Factory.CreateResource();
using(obj as IDisposable)
    Console.WriteLine(obj.ToString());
```
특정 객체에 대해여 using구문을 사용해야 할지가 명확하지 않다면, using 구문을 일단 사용해보자. 오류가 나면 수정하라.

더 복잡한 경우를 살펴보자.

첫번째 예제에서는 myConnection과 mySqlCommand 두개의 객체에 대해서 Dispose()메소드가 호출되어야 한다. 두개의 객체에 대해서 각기 using문을 사용하면, 각각 try/finally 블록을 독립적으로 생성하게 된다. 그리하여 다음과 같은 IL코드를 생성한다.
```
public void ExecuteCommand(string connString, string commandString)
{
    SqlConnection myConnection = null;
    SqlCommand mySqlCommand = null;
    
    try {
        myConnection = new SqlConnection(connString);
        try {
            mySqlCommand = new SqlCommand(commandString, myConnection);
            myConnection.Open();
            mySqlCommand.ExecuteNonQuery();
        }
        finally {
            if(mySqlCommand != null)
                mySqlCommand.Dispose();
        }
    }
    finally {
        if(myConnection != null)
            myConnection.Dispose();
    }
}
```
위와 같이 보기 흉한 코드를 생성하는데 이럴때는 try/fianlly 구문을 직접구현하는 것이 더 나을 때가 있다.
```
public void ExecuteCommand(string connString, string commandString)
{
    SqlConnection myConnection = null;
    SqlCommand myCommand = null;
    
    try {
        myConnection = new SqlConnection(connString);
        mySqlCommand = new SqlCommand(commandString, myConnection);
        
        myConnection.Open();
        mySqlCommand.ExecuteNonQuery();
    }
    finally {
        if(mySqlCommand != null)
            mySqlCommand.Dispose();
        if(myConnection != null)
            myConnection.Dispose();
    }
}
```

## 결론
> 하나의 객체만을 사용하고 있는 경우라면 단순히 using을 사용하는 것이 자원의 해제를 자동화는 가장 좋은 방법이다. 만일 다수의 객체에 대해서 Dispose()메소드가 호출되어야 하는 경우라면, using문장을 반복 배치하거나 try/finally블록을 직접 구현하는 것이 좋다.

> Dispose()는 메모리로부터 객체를 제거하지 않는다. 이것은 단지 unmanaged 자원을 해제하는 것을 도와주는 시기를 제공해 줄 뿐이다. 그렇기 때문에 만일 객체가 다른 부분에서 참조되고 있는 경우라면, Dispose()를 호출했을 때 문제가 발생할 수 있다. 따라서 프로그램 내의 다른 부분에서 참조되고 있는 객체의 경우는 절대로 Dispose()를 호출해서는 안된다.

> IDisposable interface를 구현한 객체는 항상 적절한 시점에 Dispose()가 호출될 수 있도록 코드를 작성하라.
