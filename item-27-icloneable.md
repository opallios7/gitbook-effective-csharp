# Item #27 ICloneable의 구현을 피하라

ICloneable은 유요한 interface임에는 분명하다. 하지만 반드시 지켜야만 하는 규칙들이 있다. value 타입에 대해서는 ICloneable interface를 추가해서는 안되며, 상속관계에 있는 타입들에 대해서는 가장 하위의 클래스에서만 필요에 따라 ICloneable interface를 구현하는 것이 좋다. 상위 클래스는 하위 클래스가 ICloneable interface를 구현할 때, 상위 클래스의 멤버를 복사할 수 있도록 protected 복사 생성자를 제공하는 것이 좋다. 이러한 경우를 제외하고는 ICloneable를 사용하지 말자!!

