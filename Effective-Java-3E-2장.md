# Effective Java 3E 2장

## 객체 생성과 파괴

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

- 자바 9 는 private 정적 메서드까지 허락한다.

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

- 필요없는 매개변수를 넘겨야한다.
- 매개변수가 많아지면 경우의 수 만큼 생성자가 생겨야 한다는 문제가 생김
- 또한 자료형이 같은게 연속으로 정의 될 경우 Client 가 제대로 이해하기 어렵게 된다.

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



- 두가지 방법은 직렬화 역직렬화 과정에서 새로운 인스턴스가 만들어진다. 이것을 에방하려면 readResolve() 메서드를 사용하자

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



### 3. 열거 타입 방식의 싱글턴 - 바람직한 방법

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

단, Enum 외 클래스를 상속해야 한다면 사용 불가능 하다.



## Item 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때가 있다. 객체지향적이지 않는 방식이지만 쓰일 때가 있다. ex) java.util.Arrays , java.util.Collections

정적 멤버만 있는건 인스턴스화를 하려고 만든게 아니기 때문에 막아야한다. 생성자를 명시하지 않으면 컴파일러가 자동으로 public 기본 생성자를 만든다.  추상 클래스로 만든느 것은 인스턴스화를 막을 수 없다. 상속해서 쓰면 그만이다. 따라서 private 생성자를 추가하자.

```java
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    // 기본 생성자가생기지 않도록 한다,
    private UtilityClass() {  
        throw new AssertionError(); // 에러를 던질 필요는 없지만 혹시 내부에서 호출할 경우가 있을 수 도 있기 때문.
    }
}
```

## Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

내부적인 필드를 미리 정의하는 것 보다 생성자나 팩토리로 리소스를 전달하는 방법을 사용하자. 의존하는 리소스에 따라 행동을 달리하는 클래스를 만들 때는 싱글톤이나 스태틱 유틸 클래스를 사용하지 말자.

의존성 주입이 유연함과 테스트 용이함을 크게 향상 시켜준다. 하지만 코드의 양이 방대해지면 복잡하게 보일 수 있다. 따라서 대거, 쥬스, 스프링 같은 프레임웤을 사용하면 더욱 편리하다.

## Item 6. 불필요한 객체 생성을 피하라

Immutable 객체는 항상 재사용이 가능하다.

문자열 객체는 JVM 위에 동일한 문자열이 있으면 재사용한다. new String("foo"); 해서 사용 절대 금지.

Auto Boxing 과정을 많이 줄이자. 되도록 primitive 타입 사용 특히 계산하는 과정에서 Boxing 타입을 사용하지 말자

정규식에 사용할 `Pattern` 같은 경우 미리 private static final Patten 선언하고 쓰면 좋다 . `String.matches`는 내부적으로 `Pattern` 객체를 만들어 쓰는데 그 객체를 만들려면 정규 표현식으로 [유한 상태 기계](https://ko.wikipedia.org/wiki/유한_상태_기계)로 컴파일 하는 과정이 필요함. 비싼 객체임.

지연초기화는 되도록 하지말자 성능에 큰 변화가 없지만 코드가 복잡해보임. 또한 오바해서 자신만의 pool 을 만들지마라 메모리 사용량을 늘리고 성능을 떨어트린다. 

객체 생성은 비싸니 피해라가 아니라 비싼 객체 생성을 피하라 이다.

## Item 7. 다 쓴 객체 참조를 해제하라

### 1. 메모리 직접 관리

Stack 같은 경우 pop 과정을 수행한다면 pop 된 자리는 객체 해제를 해야한다. 비활성영역이라고 하는데 프로그래머는 비활성영역이라고 인지가능하지만 GC 는 인지 하지 못한다. 

- 따라서 pop 된 위치를 null 처리를 해주어야 GC 가 인지할 수 있다. 
- 잘못된 참조로 원하지 않는 동작을 차단하고, NullPointerException 이 바로 일어나도록 할수있음

사실 null 처리 하는 것은 위험할 수 있고 코드를 지저분하게 만들 수 있다. 따라서 일반적으론 그 변수를 scope 밖으로 밀어내는 방법이 더욱 좋다.

### 2. 캐시

**캐시의 키**에 대한 레퍼런스가 캐시 밖에서 필요 없어지면 해당 엔트리를 캐시에서 자동으로 비워주는 `WeakHashMap`을 쓸 수 있음 단, `WeakHashMap` 은 이러한 상황에서만 쓸 수 있다.

특정 시간이 지나면 캐시값이 의미가 없어지는 경우에 백그라운드 쓰레드를 사용하거나 (아마도 `ScheduledThreadPoolExecutor`), 새로운 엔트리를 추가할 때 부가적인 작업으로 기존 캐시를 비우는 일 추가할 것. (`LinkedHashMap` 클래스는 `removeEldestEntry`라는 메서드를 제공함.)

### 3. 콜백

클라이언트가 콜백을 등록만하고 해지하지 안흔다면 쌓여만 가게 된다. 이럴떄 약한 참조(`Weak Reference`)로 콜백을 걸면 GC 가 수거해간다. ex) `WeakHashMap` 을 쓰자.

## Item 8. finalizer 와 cleaner 사용을 피하라

이 두개는 C++ 에서의 소멸자가 아니다. 따라서 언제 실행될지 모른다.  자바 9에서는 Finalizer가 deprecated 되었다.

- 안전망 역할로 자원을 반납하고자 하는 경우.
- 네이티브 리소스를 정리해야 하는 경우.

이 두 경우만 쓴느게 그나마 좋다고 한다.

Cleaner 는 별도의 쓰레드를 사용해서 그나마 안전하지만 그래도 쓰지마라..

## Item 9. Try-Finally 보다는 Try-with-Resource를 사용하라

- JDK 1.7 에서 추가된 AutoCloseable 인터페이스를 사용한다.

```
/**
 * @author Josh Bloch
 * @since 1.7
 */
public interface AutoCloseable {
    void close() throws Exception;
}
```

Josh Bloch 는 이펙티브 자바 저자이다.

#### 왜 Close() 가 중요한가?

- 자바는 입출력을 수행하기 위해서 스트림이라 불리우는 데이터를 주고 받는 통로를 사용한다.
- 자원을 얻을 때에는 open() 메서드를 그리고 자원을 반납할 때 close() 메서드를 사용하게 된다.
- 특별한 처리가 없다면 입출력을 수행한는 도중 예외가 발생하면 스트림이 open 된 상태로 남겨진다.
- 자바에는 GC 가 존재하지만 GC 처리가 수행되기 전까지는 성능에 영향을 주게 된다. 따라서 Memory Leak 현상을 막기위해 자원 반납이 중요하다.

### 1. 기존의 try-catch-finally 를 사용한 방법

```
    FileOutputStream out = null; 
    try{
        out=new FileOutputStream("test.txt");
    } catch(FileNotFoundException e){
        e.printStackTrace();
    } finally{
        if(out!=null)
        {   try { 
            out.close(); //close 하다가 예외가 발생할 수 있다. 
            } catch (IOException e) { 
                e.printStackTrace(); 
            } 
        }
    }
```

### 2. try-with-resources 사용한 방법

```
    try(FileOutputStream out = new FileOutputStream("test.txt")) { 
    //로직 구현 
    }catch(IOException e){ 
    e.printStackTrace(); 
    }    
```

- 프로그래머가 보기에도 매우 직관적이고 실수할 경우를 줄여준다.

### 3. AutoCloseable 시에도 예외가 발생하는 경우

```
    javaTest.IOException at 
    javaTest.MyResource.out(ExceptionTest_1.java:27) at 
        Suppressed: javaTest.CloseException at 
            javaTest.FileOutputStream.close(ExceptionTest_1.java:23) at 
```

- Suppressed 라는 문구를 붙여서 기존의 예외 로그랑 같이 넘어오게된다.

#### AutoCloseable 구현

```
class Test implements AutoCloseable{
    public Test(){
    }
    
    @Override
    public void close() throws Exception {
        System.out.println("Test Close Exception");
    }
}

public static void main(String[] args) {
    try(Test test = new Test()){

    }catch(Exception e){

    }
}
```

- AutoCloseable Interface를 implements후 구현해주면됨

