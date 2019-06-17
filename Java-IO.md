# JAVA 입출력 팁

## 입력

### Scanner

```java
Scanner scanner = new Scanner(System.in);
int a = sc.nextInt();
```

Scanner 은 지원해주는 메소드가 많고 사용하기 쉽기 때문에 자주 사용합니다. 하지만 버퍼 사이즈가 1024 이며 많은 작업을 요구할 때는 성능상 저하가 올 수 밖에 없습니다.

### BufferedReader

```java
BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));//선언

String string = bufferedReader.readLine(); // 한줄식 받아옴
StringTokenizer stringTokenizer = new StringTokenizer(string); //토크나이져를 통해 파싱을 한다 지금은 띄어쓰기 단위로 잘라준다
int a = Integer.parseInt(stringTokenizer.nextToken());
```

BufferedReader의 버퍼사이즈는 8192 chars이기 때문에 많은 입력이 있다면 BufferedReader가 성능 상 우위를 가질 수 밖에 없습니다.

여러 줄을 받을 때는

```java
 BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));

        String string;
        while((string=bufferedReader.readLine())!=null){
            // string 한줄
        }
```

버퍼스트림(Buffered Stream)이란 기본 입출력스트림에 버퍼기능을 추가하는 스트림을 의미합니다. 즉, 입력된 데이터가 바로 프로그램으로 전달되지 않고 중간에 버퍼링이 된 후에 전달된다는 의미입니다. 출력도 마찬가지로 버퍼를 거쳐서 간접적으로 출력장치로 전달됩니다. 따라서 중간버퍼를 사용함으로써 시스템의 데이터처리 효율성을 높여줍니다.

## 출력

### System.out.println()

```java
System.out.println("Hello World")
```

디버그를 위한 출력을 할때는 Logger 를 사용해서 사용하는 것이 맞는 방법입니다. 하지만 콘솔 출력을 하기 위해선 기본적으론 System.out.println() 을 사용합니다.

System.out.println() 방식의 출력은 시스템 리소스를 필요 이상으로 잡아먹는다는 한계가 존재합니다.

* JAVA DOC 에 따르면

> System은 Object 클래스를 상속받은 final 클래스이다. 인스턴스화 할수없다.
> out은 PrintStream 의 인스턴스 이다
> println은 PrintStream의 메소드이다.

즉 println은 println -> print -> write() + newLine() 순서로 처리된다고 합니다.

### BufferedWriter 클래스

```java
BufferedWriter bufferedWriter = new BufferedWriter(new OutputStreamWriter(System.out));//선언

bufferedWriter.write("Hello World!");
// write한다고 해서 바로바로 출력되지 않습니다.
// 직접 출력 stream에 반영되는 것이 아니라 성능을 위해 buffer에 저장해두었다가
// BufferedWriter가 flush되거나 close되었을 때 한번에 출력 stream에 반영하기 때문입니다.
bufferedWriter.flush(); 
bufferedWriter.newLine(); // 줄바꿈이 필요할 경우 사용합니다.
bufferedWriter.close(); // 버퍼 닫기
// close는 stream을 닫아버리기 때문에 계속 출력하고자 한다면 flush 사용합니다.
```

## 문자열 포멧팅

### String

```java
String c = a + b;
```

자바에선 String 끼리 + 연산자를 통해 붙이기를 가능합니다. 하지만 이건 내부적으로 Autoboxing, Unboxing 과정을 통하여 concat 메소드를 참조해 사용하기 때문에 매우 느리다고 할 수 있습니다. 따라서 되도록이면 사용을 지양합니다

* String 은 immutable(불변) 한 객체이기때문에 연산을 시행하면 또다른 String 객체를 생성하는 과정을 거쳐서 느립니다.

### StringBuilder

```java
BufferedWriter bufferedWriter = new BufferedWriter(new OutputStreamWriter(System.out));

StringBuilder stringBuilder = new StringBuilder();

stringBuilder.append("첫번째 문장\n"); 
stringBuilder.append("두번 재 문장\n");
bufferedWriter.write(stringBuilder.toString()); // 출력 가능한 String 형태로 변환

bufferedWriter.flush();
bufferedWriter.close();
```

StringBuilder 는 애초에 mutable 한 객체임으로 append 메소드를 통해 붙일 수 있습니다.(insert,delete,replace 등등) 새로운 객체를 생성하는 과정이 아님으로 내부적으로 boxing 과정을 거치지 않아 속도적으로 빠르게 사용가능합니다.

* 출력시에는 String 값이 기본 형태임으로 toString() 을 사용해 출력합니다.

### StringBuffer

```java
BufferedWriter bufferedWriter = new BufferedWriter(new OutputStreamWriter(System.out));

StringBuffer stringBuffer = new StringBuffer();

stringBuffer.append("첫번째 문장\n"); 
stringBuffer.append("두번 재 문장\n");
bufferedWriter.write(stringBuffer.toString()); // 출력 가능한 String 형태로 변환

bufferedWriter.flush();
bufferedWriter.close();
```

StringBuffer 는 StringBuilder 와 동일하지만 thread-safe 한 특징을 가지고 있습니다. 따라서 서버를 구성하고 다중의 사용자가 접근이 가능해야할 경우 StrungBuffer 을 사용해야 합니다. StringBuilder는 내부적으로 synchronization 적용하는 로직이 존재해 약간 느릴 수 있습니다.

* StringBuilder > StringBuffer >>> String

## Reference

https://docs.oracle.com/javase/8/docs/api/java/lang/System.html
https://docs.oracle.com/javase/7/docs/api/java/io/BufferedWriter.html
