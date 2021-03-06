# Item #10 GetHashCode()의 함정에 유의하라

해시테이블이나 Dictionary와 같이 해시 코드를 기반으로 하는 collection에 Key로 활용할 타입을 정의하는 경우를 제외하고 GetHashCode()메소드를 재정의하지 않는 것이 좋다.

닷넷에서는 모든 객체들이 해시 코드를 가지고 있으며, 이 값은 System.Object.GetHashCode()에 의해서 반환된다. GetHashCode()를 재정의하기 위해서는 최소한 다음의 3가지 규칙을 준수해야 한다.

>1. 만일 두개의 객체가 동일하다면(operator== 에 의해서 정의된 동일여부) 두 객체는 동일한 해시코드를 생성해야 한다. 하지만 해시코드를 이용하여 특정 컨테이너 내에서 다수의 객체들을 찾아내는 데 사용할 수 없다.
2. 특정 객체 a에 대해서 a.GetHashCode()의 반환값은 객체의 인스턴스가 생성된 이후에는 변하지 않아야 한다. 어떠한 시점에 a 객체의 GetHashCode()를 호출하더라도 반환되는 값은 항상 동일해야 한다. 해시 기반 컨테이너 내에서는 객체를 찾을 때 해시코드를 이용하여 저장공간을 검색하기 때문에 이 값이 변경되면 객체가 저장된 올바른 저장공간을 찾지 못할 수 있다.
3. 해시 함수는 모든 입력 값에 대해서 integer의 표현범위 내에서 골고루 잘 분산되어야 한다. 이러한 특성이 해시기반 컨테이너의 수행성능에 영향을 미친다.


## 결론
> GetHashCode()는 매우 특별한 요구사항들을 가지고 있다. 동일한 객체들은 동일 해시코드를 가져야 한다. 해시코드는 반드시 인스턴스별로 불변의 값을 가져야 하고 그값이 효과적으로 잘 분산되어 있어야 한다. immutable 타입들에 대해서는 3가지 규칙이 모두 지켜져야 하고, 다른 타입들은 동작방식에 따라 차이가 있을 수 있다.