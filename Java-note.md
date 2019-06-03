## JAVA Note



### JAVA Pass By Value ??????

Java 는 Pass-By-Value -> 불변성을 지키기위해

근데 왜 객체를 넘기면 레퍼런스처럼 작동하나? -> 넘어가는 Value 가 Reference 가 넘어감 즉 객체의 Value 는 Reference 라고 보면 된다. 

따라서 불변성 있도록 작업하고 싶으면 객체를 방어적 복사 까지 고려해주어야 한다.

### JAVA Generic Primitive Type?

Map , List 사용하다보면 primitive 타입은 키 값, value 값으로 사용하지 못하는 것 을 볼 수 있다. 방싱된 타입으로만 해야한다.

이유는 자바 Generics 이 primitive 타입을 사용하지 못한다. 오브젝트만 사용가능 하다.

```java
List<Integer> list = new ArrayList<>();
list.add(1);
Integer num1 = list.get(0);
```

실제 컴파일 과정에서 이렇게 변환되기 때문에 Object 타입만 쓰도록 한다.

```java
List list = new ArrayList();
list.add(1);
Integer num1 = (Integer)list.get(0);
```

즉 제네릭에서 사용하는 Object 클래스는 모든 오브젝트의 슈퍼클래스이다. primitives 는 Object 가 아니기 때문에 제네릭 타입에서 사용할 수 없다.

```
물론 Java 컴파일러 측에서 Auto-Boxing 을 내부적으로 이용한다면 primitive 타입을 쓸 수 있을 수도 있다. 하지만 JVM 은 primitive 타입과 reference 타입은 다른 bytecode 를 사용하기 때문에 COST 가 비싸다고 한다.?
```

