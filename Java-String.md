## Java String 

모든 언어에서 제일 처음 배우는 자료형이 `String` 이다.  모든 시작은 "Hello World" 따라 써보며 시작하기 때문이다.

알고리즘 풀이시 C++ 에서 Java 진영으로 넘어오면서 제일 큰 이점은 역시 `String` 관련 문제들이였다. 매우 사용하기 편하지만 원래 편한만큼 조심해서 써야한다. JVM 내부적으로 나름대로 최적화를 많이 해주었지만 그래도 신경쓰고 사용하자.

### Java String 은 Immutable한 참조 자료형이다.

new 생성자로 생성이 가능하고 리터럴 형식으로 생성이 가능하다.  또한 heap 메모리에서 관리한다는 사실은 다른 참조 자료형과는 다를게 없지만 immutable 하나는 틍징이 있다.

```java
String a = "test";
String b = new String("test");
a == b; // false
a.equals(b); // true
```

두 가지 방법 모두 String 객체를 생성하지만 동작 방식이 다르다. new 로 선언된 것은 `String constant pool` 에 들어가지 않는다. 밑의 사진을 보면 이해가 갈 수 있다.

![img](https://lh3.googleusercontent.com/3Iko09F6vHHT4tvp2cZ_I92lXKUHKqwQ_cqwk1WoLQsvca8O0AokSXg-W_ixK1B4m6tGFZNGwiCK2JDUG9cTvvib3yY4EazZXTqwrj1bFT9tGRqFn5DDiSQ5z_x1Hty406fx_H4)

#### 동작원리

동작원리를 알고 보면 new String 으로 선언하면 `String constant pool` 외부에 위치하게 된다. 리터럴로 선언하면 내부적으로 `interning` 이라는 과정이 진행된다. 즉 "string" 이라고 선언하면 `String constant pool`에 존재하는 리터럴이면 그 객체를 참조해 사용하고 없으면 생성하면서 `interning` 과정이 진행된다. `interning` 과정이 진행되면 문자열을 선언하면 `pool`에서 체크를 하고 다음부터 새 객체를 만들지 않는다.

```java
String a = "test";
String b = new String("test");
String c = new String("test").intern();
a == b // false
a == b.intern(); // true
a.equals(b); // true
a == c; // true
a.equals(c); // true
```

`interning` 방식을 위에서 보면 알 수 있을 것이다. 

가끔 유틸 클래스중 `interning` 과정을 거치지 않는 함수들이 있는데 대표적으로 .substring() 이다. `interning` 과정을 거치지 않아서 동일성 비교가 먹히지 않는다. 이러한 상황이 있으니**문자열 비교시에는 `equals` 같은 메소드를 사용해 비교하자.**



### String constatnt pool 위치 변경

Java 7 이 되면서 `String constant pool` 의 위치가 `Perm`에서 `Heap` 로 변경이 되었다. Perm 은 늘릴 수는 있지만 고정된 사이즈이고 런타임에서 확장되지 않는다고 한다. 따라서 OutOfMemoryException 의 위험이 존재해 Heap 영역으로 위치를 변경하였다고 한다. Heap 영역에 올림으로   이 영역에 존재하는 String 객체들은 Heap 영역에 존재하기 때문에 GC의 대상이 된다.

### String Immutable 특징

`String` 은 불변객체라고 다들 소개한다. 그런데 불변 객체인데 값이 바뀌는 것을 볼 수 있다.

``` java
String a = "test";
a = "change";
System.out.println(a); // change
```

사실 값이 바뀌는 것이 아니라 a 라는 변수가 "change" 라는 (pool 에 없다면 String 객채를 생성하고 ) String 객체를 참조하는 것이다. 따라서 primitive 변수 처럼 값이 변한다고 생각하고 사용하다보면 메모리 관리 측면에서 매우 비효율 적이 될 수 있다.

##### 따라서 String  `+` 연산에서 문제가 많이 발생한다.  새로운 객체가 계속 생기기 때문

```java
String a = "Dog";
String b = "Cat";
String c = a + b;
System.out.println(c); // "DogCat"
```

이런 상황이 많이 나온다. 특히 `query` 작성시 혹은 `URI` 작성시 문제가 생긴다. 또한 반복문을 돌면서 + 연산을 할 경우 매우 큰 문제가 생길 수 있다. `+` 대신 `StringBuffer & StringBuilder`을 사용하자.

### StringBuffer & StringBuilder ?

수정가능한 문자열 객체라고 보면 될것이다. 그래서 변경작업에 대해 새로 객체를 할당하는 오버헤드가 적다. 따라서 문자열 연산이 많을 경우 사용하면 매우 이득이다.

저 둘의 차이는 StringBuffer 는 `ThreadSafe` 하다는 점이다. 둘 다 슈퍼클래스는 AbstractStringBuilder로써 구현한 abstract method는 같다. 하지만 `StringBuffer`는 내부적인 함수에 `synchronized`가 걸려있다. 따라서 멀티스레드 환경에서는 `StringBuffer` 를 사용하면 좋다. 그런데 웹 어플리케이션에서는 대부분 멀티스레드 환경이고 개발자가 판단하기 어려운 경우 StringBuffer 를 사용하자. 저 둘의 성능차이는 오류나서 고생하는 것을 감당할만 하다. 뭐 알고리즘이나 단순 계산하는 로직일 경우 StringBuilder 를 사용하면된다. 

그런데 특이점은 Java 8 까지  `+` 경우 내부적으로는 StringBuilder 을 사용한다고 한다. 그러면 굳이 StringBuilder 을 선언해서 쓸 필요가 없지 않을까? 라는 생각도 들지만 + 연산 할때마다 새로운 StringBuilder 객체를 생성하기 때문에 개발자 스스로 한번 선언한 후에 `.append()` 메소드로 작업하는게 성능향상에 좋다. 특히 쿼리문 작성시 가독성을 높이려고 `+` 연산자가 많이 들어갈 때가 있는데 밑의 사례처럼 작성하라

```java
String q = "insert into";
q += "table";
q += "item1":
q += item1Value.toString();
db.excute(q);
StringBuilder sb = new StringBuilder();
sb.append("insert into");
sb.append("table");
sb.append("item1");
sb.append(item1Value.toString());
db.excute(sb); // or db.excute(sb.toString())
```

#### 성능 차이

그 성능에 매우 집착을 보이는 이들이 있다. 이러한 경우도 `StringBuilder 가 무조건 빠르니까 이거 써야해` 하면서 `Thread-Safe` 하지 않는 상황에서도 사용할 수 있다. 밑에 2억번  돌려본 차이이다.

```java
public static void main(String[] args) {
        long t0 = System.currentTimeMillis();
        StringBuffer buf = new StringBuffer();
        for (long i = 0 ; i < 20000000; i++){
            buf.append("test string");
        }
        System.out.println("Buffers : "+(System.currentTimeMillis() - t0));

        t0 = System.currentTimeMillis();
        StringBuilder building = new StringBuilder();
        for (long i = 0 ; i < 20000000; i++){
            building.append("test string");
        }
        System.out.println("Builder : "+(System.currentTimeMillis() - t0));
    }
```

```java
Buffers : 1238
Builder : 930
```

2억번 `append` 결과의 차이가 0.3초이다. 2억번 돌려야 0.3 초 성능 차이를 겨우 느낄 수 있는데 `Thread-Safe` 하지 않다면 `StringBuffer` 을 쓰는 것이 정신건강에 좋을 것이다. 

* 2억번이상 돌리는 경우가 있지 않느냐 ?

라고 한다면 뭐 .... 로직 구현을 다시 한번 살펴봐야 하지 않을까? 





