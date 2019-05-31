# Effective Java 3E 2장

## Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라

#### 장점 1. 이름을 가질 수 있다. 

이름이 있다는 것은 가독성이 좋아진다고 볼 수 있다. 

메이플 스토리 캐릭터를 만든다고 가정하고 이해를 해보자. (예전엔 주사위였지만 지금은 만들어 주니까...)

```java
class MapleStoryCharacter{
    String name; // 이름
    int str; // 힘
    int dex; // 민첩
    int	magic; // 인트
    int lux; // 럭
    
    public MapleStoryCharacter(String name, int str, int dex, int ints, int lux) {
        this.name = name;
        this.str = str;
        this.dex = dex;
        this.ints = ints;
        this.lux = lux;
    }
    public static MapleStoryCharacter newWarrior(String name){
        return new MapleStoryCharacter(name,12,5,4,4);
    }
    public static MapleStoryCharacter newWizard(String name){
        return new MapleStoryCharacter(name,4,4,12,5);
    }
    public static MapleStoryCharacter newArcher(String name){
        return new MapleStoryCharacter(name,4,12,4,5);
    }
    public static MapleStoryCharacter newThief(String name){
        return new MapleStoryCharacter(name,4,5,4,12);
    } 
}
```

생성자만 사용한다면 parameter 값으로 판단해야 한다.

```java
MapleStoryCharacter warrior = new MapleStoryCharacter("타락파워전사",12,5,4,4);
MapleStoryCharacter wizard = new MapleStoryCharacter("마법사일까?",4,4,12,5);	
```

정적 팩토리 메서드를 사용한다면 매서드 명으로 무엇을 생성하는지 알아 보기 쉽다.

```java
MapleStoryCharacter theif = MapleStoryCharacter.newThief("도적띠");
MapleStoryCharacter archer = MapleStoryCharacter.newArcher("아시안느");
```

좀 더 직관적으로 만들 수 있을 것이다. 

#### 장점 2. 이름을 가질 수 있다. 

위와 같은 상황은 객체가 고유한 특성을 가지고 진행되어야 하기 때문에 객체를 생성한다. 만약 읽기 작업만(?) 필요한 객체가 있다면 immutable 객체를 생성하고 사용하면 된다. `java.lang.Boolean.valueOf` 가 대표적인 예이다.

```java
public static final Boolean TRUE = new Boolean(true);
public static final Boolean FALSE = new Boolean(false);

public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
}
```

미리 선언해 놓고 필요할때 가져다 쓴다. 

#### 장점 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

팩터리 패턴처럼 하위 타입을 팩토리 메서드에서 반환 시켜 줄 수 있다. 자바 8 부터는 인터페이스가 정적 메서드를 자길 수 없다는 제한이 풀려서 정적 멤버들 상당수를 인터페이스 자체에 둘 수 있다. 

* 자바 9 는 private 정적 메서드까지 허락한다.

```java
class MapleStoryCharacter implements Character
```

추가해주고 

```java
import java.security.InvalidParameterException;

public interface Character {
    static Character getInstance(String type, String name) {
        switch (type) {
            case "warrior":
                return MapleStoryCharacter.newWarrior(name);
            case "wizard":
                return MapleStoryCharacter.newWizard(name);
            case "thief":
                return MapleStoryCharacter.newThief(name);
            case "archer":
                return MapleStoryCharacter.newArcher(name);
            default:
                throw new InvalidParameterException();
        }
    }
}

```

```java
Character warrior = Character.getInstance("warrior","타락파워전사");
```

뭐 이런식으로 사용가능 한거 같다. (String type 를 enum 으로 바꾸면 더 좋고)

#### 장점 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.

- `EnumSet` 클래스는 생성자 없이 public static 메소드, `allOf()`, `of()` 등을 제공함. 그 안에서 리턴하는 객체의 타입은 enum 타입의 개수에 따라 `RegularEnumSet` 또는 `JumboEnumSet`으로 달라짐.

```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> var0) {
        Enum[] var1 = getUniverse(var0);
        if (var1 == null) {
            throw new ClassCastException(var0 + " not an enum");
        } else {
            return (EnumSet)(var1.length <= 64 ? new RegularEnumSet(var0, var1) : new JumboEnumSet(var0, var1));
        }
    }
    
class RegularEnumSet<E extends Enum<E>> extends EnumSet<E> {}
class JumboEnumSet<E extends Enum<E>> extends EnumSet<E> {}
```

객체 타입은 노출하지 않고 감춰져 있기 때문에 추후에 JDK의 변화에 따라 새로운 타입을 만들거나 기존 타입을 없애도 문제가 되지 않음.

사용자는 어떤 것을 사용하던 원하는 값을 얻을 수 있다.

#### 장점 5. 정적 패터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

대표적인 예로 `JDBC` 가 있다.

Dirver 클래스가 클래스 로더에 의해 로드가 되면 자체적으로 인스턴스를 만들어 DirverManager 에 등록이 되어야 한다. Class.forName(String name)은 파라미터로 받은 name에 해당하는 클래스를 로딩하며, 클래스가 로드 될 때 static 필드의 내용이 실행되는 것을 이용해 자기 자신을 DriverManager 클래스에 등록한다. 

하지만 JVM이 동작을 시작하고 코드가 실행되기 전 까지는 어떤 Driver가 사용될 지 모르기 때문에, 동적으로 드라이버를 로딩하기 위해 리플랙션을 사용한다,

- 자바 6부터는 `java.util.ServiceLoader`라는 일반적인 용도의 서비스 프로바이더를 제공하지만, `JDBC`가 그 보다 이전에 만들어졌기 때문에 `JDBC`는 [ServiceLoader](https://docs.oracle.com/javase/9/docs/api/java/util/ServiceLoader.html)를 사용하진 않는다.

#### 단점 1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

상속 보다는 구현을 활용하라는 OOP의 원칙 (extends 대신 implements?)

- 상속을 제대로 사용하지 못하면 소프트웨어는 많이 불안정해짐.
- 상속은 캡슐화를 위반할 가능성이 높음,
- 불변 타입인 경우나 상속 대신 컴포지션을 사용하도록 권장하기 때문에 장점이라 말할 수 도 있다.

#### 단점 2 . 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

API 설명에 명확히 드러나지 않음으로 파악하기 어렵다. 아직 자바독이 처리를 못해주는데 처리를 해주는 날이 온다면 매우 좋을 것이다.

그러므로 문서화를 잘하고 함수명 규약을 잘 지키면 괜찮다.

```
자주 쓰는 규약
from
of
valueOf
getInstance/instance
newInstance/create
getType
newType
type
```

## Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라

정적 팩터리와 생성자에는 선택적 매개변수가 많을 때 번거로워진다는 단점이 존재한다.

### 1. 점층적 생성자 패턴

```java
public class Foo {
	private int fooInt; // 선택
    private int fooInt2; // 선택
    private String fooStr; // 필수
    private List<Bar> bars; // 선택
  
    public Foo(String fooStr) {
    	this(fooStr, 0);
    }
  
    public Foo(String fooStr, int fooInt) {
        this(fooStr, fooInt, new ArrayList<Bars>());
    }
    
    public Foo(String fooStr, int fooInt, int fooInt2) {
        this(fooStr, fooInt, fooInt2 new ArrayList<Bars>());
    }
  
    public Foo(String fooStr, int fooInt,  List<Bar> bars) {
        this.fooStr = fooStr;
        this.fooInt = fooInt;
        this.bars = bars;
    }
 }
```

* 필요없는 매개변수를 넘겨야한다.
* 매개변수가 많아지면 경우의 수 만큼 생성자가 생겨야 한다는 문제가 생김
* 또한 자료형이 같은게 연속으로 정의 될 경우 Client 가 제대로 이해하기 어렵게 된다.

### 2. 자바빈즈 패턴 

인자없는 생성자를 만들고 세터로 데이터를 다 넣어버리는 방법

```java
public class Foo {
    private int fooInt; // 선택
    private int fooInt2; // 선택
    private String fooStr; // 필수
    private List<Bar> bars; // 선택
  	
    public Foo(){}
    
    public void setFooInt(int fooInt) {this.fooInt = fooInt;}
    public void setFooInt2(int fooInt2) {this.fooInt2 = fooInt2;}
    public void setFooStr(String fooStr) {this.fooStr = fooStr;}
    public void setBars(List<Bar> bars) {this.bars = bars;}
 }
```

```java
Foo foo = new Foo();
foo.setFooInt(29);
foo.setFooStr("test")
```

어디서든 Setter 을 호출 할 수 있음으로 불변성을 지킬 수 없다.

Thread safe 하지도 않다.

- 여러번의 메소드 호출로 나누어져 인스턴스가 생성되서 생성과정을 거치는동안 일관된 상태가 유지 되지 못함. ( 모든 초기화가 이루어지기전에 다른 Thread 에서 사용해버릴수도 있음 )
- 변경 불가능한 클래스를 만들 수 없음

이런 문제점때문에 freezing 을 사용하기도 한다. 하지만 컴파일러가 사용자가 freeze 메서드를 확실히 호출했는지  확신하기 어려워 오류를 잡기 어렵다.

### 3. 빌더 패턴

Builder 는 매개변수가 많이 늘어나거나 가변인자를 사용할 경우 장점

위와 같은 상황이면

```
Foo foo = new Foo.Builder().fooStr("foofoo").fooInt2(123),fooInt(1);
```

이런식으로 사용한다.

근데 빌더를 직접 구현해서 선택적인 변수만 빌더를 쓰게 할 수 있지만, 나는 [롬복 @Builder](https://projectlombok.org/features/Builder)를 쓸 것이다. 하지만 사용시 AccessLevel 을 잘 설정해주어야 한다.

## Item 3. private 생성자가 열거 타입으로 싱글턴임을 보증하라

### 1. public static final 필드 방식의 싱글턴

```java
public class Foo {
	public static final Foo INSTANCE = new Foo(); // static final!!
    private Foo() { 
        if(INSTANCE != null){ // 인스턴스가 있는데 다시 부른것
            throw new RuntimeException("비정상적인 접근");
        }
    } // private 생성자!!
    public void leaveTheBuilding() { .. . }
}
```

이 방법은 싱글턴임이 API에 명백히 드러난다. 또한 매우 간결하다.

자바의 리플랙션을 이용한 `AccessibleObject.setAccessible()`을 사용해 `private` 생성자를 다시 다시 호출 가능하기 때문에 방어적인 코드를 삽입해 주었다.

### 2. 정적 팩터리 방식의 싱글턴

이제 인스턴스 필드를 private 으로 숨기고 정적 팩토리 메서드로 반환하는 방법이다.

```java
public class Foo {

    private static final Foo INSTANCE = new Foo();

    private Foo() {
    }

    public static Foo getInstance() {
        return INSTANCE;
    }

}
```

API를 변경하지 않고도 싱글턴을 포기할 수 있다. 또한 클라이언트 코드를 고치지 않고도 포기 가능하다. 하지만 리플랙션을 이용한 접근 또한 막지 못한다.



* 두가지 방법은 직렬화 역직렬화 과정에서 새로운 인스턴스가 만들어진다. 이것을 에방하려면 readResolve() 메서드를 사용하자

```java
public class Foo {
    private static final Foo INSTANCE = new Foo();
    private Foo(){
    }
    
    public static Foo getInstance(){return INSTANCE;}
    
    private Object readResolve(){
      return INSTANCE; // 원하던 인스턴스를 반환하고 생기는 인스턴르는 GC 로 보내버린다.
    }
}
```



###  3. 열거 타입 방식의 싱글턴 - 바람직한 방법

원소가 하나인 열거타입을 선언하자.

위의 모든 문제점을 해결할 수 있다. 근데 이러한 코드를 본적은 없다... 많이 쓰는 방식인지 궁금하다.

- INSTANCE 가 생성될 때, multi thread 로 부터 안전, Thread safe
- 단 한번의 인스턴스 생성을 보장.
- 사용이 간편.
- enum value는 프로그램 전역에서 접근이 가능.

```java
public enum Foo {
    INSTANCE;
}
```

단, Enum 외 클래스를 상송해야 한다면 사용불가능 하다.

