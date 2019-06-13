## Java Wrapper Class Cache

Java의 데이터 타입은  기본적으로 두가지로 크게 분류가능하다.

1. Primitive Types

   C 나 C++ 쪽을 상용해 보았다면 바로 이해할 수 있는 타입이다. 8개의 primitive type 을 가지고 있다. byte, short, int, long, float, double, char, boolean 이 있다. binary bits 형식으로 값을 직접 할당해서 가지고 있다.

2. Reference Types

   자바에서 사용하는 Primitive Type 을 제외한 모든 타입입니다. 기본적으로 값을 직접 가지고 있는 것이 아닌 주소값을 가지는 타입들이다. 그 중에서도 원시 데이터 타입과 매칭되는 Reference Type들이 있다. `Wrapper Class` 라고 하는데  아무 생각없이 사용하면 성능 저하 및 원하는 로직이 작동 하지 않을 수 있다. 



Java는 모든 Primitive Type 에 대하여 Wrapper Class 를 제공한다. 또한 JDK 1.5 부터 AutoBoxing 과 AutoUnboxing 을 지원한다. 

![image](https://user-images.githubusercontent.com/32893340/59142855-4c394c00-89ff-11e9-9e7f-b189bce3185a.png)

코드상으론 문제없이 작성 할 수 있다. 컴파일러가 알아서 변환해주기 때문이다.

```java
Integer a = 128; // 컴파일 시 Integer a = Integer.valueOf(128);
int b = a ; // 컴파일 시 int b = a.intValue();
```

그렇다면 Wrapper Class는 Reference Type 임으로 밑의 결과는 당연히 false 임을 알 수 있다. 또한 주소 값을 확인해 보면 다른 것을 확인 가능하다.

```java
Integer num1 = 128;
Integer num2 = 128;

System.out.prinln(a==b); // false , reference Type 임으로 다른 주소값을 가짐
System.out.println(a.equals(b)); // true 내부의 값을 비교하도록 짜여있음
System.out.println(System.identityHashCode(a)); // 460141958
System.out.println(System.identityHashCode(b)); // 1163157884 
// 뒤의 숫자 값은 다를 수 있음
```

### 어 나는 `==` 으로 비교했는데 `true` 가 나온다? 이상한데? 

라고 할 수 있다. 예시로 생각나는게 없어서 이런 개똥같은 코드( [Integer 연산이 들어있고](<https://dzone.com/articles/java-performance-notes-autoboxing-unboxing>), 무한루프의 가능성도 있기 때문) 를 짜보자.

```java 
public static void main(String[] args) {
    Integer maxCount = 20;
    Integer temp = 0;
    while (true){
        System.out.println(temp);
        if(temp == maxCount){
            System.out.println("fin");
            break;
        }
        temp ++;
    }
}
```

20번 째에 끝나도록 짠다고 가정해보자. 근데 이건 문제없이 통과한다. `==` 비교 연산자를 사용했는데 원하는 대로 로직이 진행되었다. 그렇다면 `maxCount` 를 128 로 바꿔서 해보자

```java
public static void main(String[] args) {
    Integer maxCount = 128; // 128 그 이상숫자로 변경
    Integer temp = 0;
    while (true){
        System.out.println(temp);
        if(temp == maxCount){
            System.out.println("fin");
            break;
        }
        temp ++;
    }
}
```

이 코드는 무한루프에 빠진다. 즉 밑의 구문처럼 된것이다.

```java
Integer.valueOf(128) == Integer.valueOf(128) // false 
```

왜 이렇게 될까? 일단 [java.lang.Integer](<https://docs.oracle.com/javase/8/docs/api/java/lang/Integer.html>) 를 알아보자. 그리고 `valueOf` 메소드를 확인해 보면 이런 말이있다.

>valueOf

```
public static Integer valueOf(int i)
```

Returns an `Integer` instance representing the specified `int` value. If a new `Integer` instance is not required, this method should generally be used in preference to the constructor [`Integer(int)`](https://docs.oracle.com/javase/8/docs/api/java/lang/Integer.html#Integer-int-), as this method is likely to yield significantly better space and time performance by caching frequently requested values. This method will always cache values in the range -128 to 127, inclusive, and may cache other values outside of this range.

- Parameters:

  `i` - an `int` value.

- Returns:

  an `Integer` instance representing `i`.

- Since:

  1.5

-128 부터 127 까지는 cache 값 들로 사용되어진다고 한다. 개발시 자주 사용되기 때문에 미리 선언해서 사용한다고 한다.

그럼 어떻게 하는 거지 라고 보면 IDE 에서 Integer 코드를 까보자.  내부에 `IntegerCache` 라는 클래스가 있고 그 내부에 밑의 변수 들이 있다. 

```java
static final int low = -128;	
static final int high;
static final Integer cache[];
```

그리고 static block에 의해서 `IntegerCache` 클래스가 메모리에 로드되는 처음에 초기화가 된다. 즉 저기 low high 범위 안에 있는 애들은 미리 만들어놓은 객체들을 리턴해주기 때문에 주소값이 같음으로  `==` 비교 연산자가 원하는 대로 사용되었던 것이다.

캐싱 방식은 다른 `Wrapper Class` 에도 적용 되어있습니다. 소숫점을 위한 `Float` `Double` 은 적용 되어있지 않다. 또한 `Boolean` 은 `True` , `False` 만 선언해놓으면 됨으로 static 변수로 가지고 있다. 나머지드를 캐싱 방식을 사용한다. 그런데 `Integer` 만 특이하게 JVM -XX:AutoBoxCacheMax 옵션으로 최대 값을 변경 가능하다.
