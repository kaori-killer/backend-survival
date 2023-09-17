# HTTP Client

## 키워드

* TCP/IP 통신
* TCP와 UDP
* Socket과 Socket API 구분
* URI와 URL
* 호스트(host)
  * IP 주소
  * Domain name
  * DNS
* 포트(port)
* path(경로)
* Java text blocks
* Java InputStream과 OutputStream
* Java try-with-resources

## 최종 목표

HTTP 에 대해서 간단히 알아보고 직접 구현해보자.

> 웹의 핵심 요소인 HTTP는 웹 개발자에게 있어서 필수적으로 알아야 하는 지식이다. 그러나 상당수의 개발자들이 HTTP가 어떠한 원리로 동작하는지 제대로 알지 못하고 있다. 이번 과정을 통해 HTTP 통신을 처리하는 웹 서버가 의외로 간단한 원리로 구현되어 있다는 걸 알게 될 것이다. 직접 간단한 웹 서버를 구현해보면서, HTTP를 제대로 이해하고 동시에 Spring Web MVC가 어떤 편의를 제공하는지 절실하게 느껴보자.

## 현재 목표

TCP/IP 통신에 대해 가볍게 알아보자. 그리고 HTTP Client 를 가볍게 구현해보자.

## TCP/IP 통신 이란?

TCP/IP 는 TCP 프로토콜을 의미하는 게 아니다. [인터넷 프토토콜 스위트](https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%EB%84%B7\_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C\_%EC%8A%A4%EC%9C%84%ED%8A%B8)를 의미한다. 그래서 인터넷 프로토콜 스위트에서 제일 많이 쓰이는 TCP 와 IP 의 이름을 따서 TCP/IP 로 이름 지은 것이다.

이미지를 보면 HTTP 는 TCP 와 IP 를 기반으로 올라간다.

> 물론 최근에는 UDP 를 TCP 와 비슷하게 구현한 상태로 그 위에서 HTTP 3.0 을 구현하긴 했다. 하지만 우리는 HTTP 1.1 로 가장 기본적인 것을 쓸 것이기 때문에 TCP 를 쓴다고 생각하면 된다.

![](https://velog.velcdn.com/images/jeong\_lululala/post/37c04248-05fc-4574-893b-185a85c53fcd/image.png)

## 전송 계층의 대표적인 프로토콜

이전 게시글에서 다음과 같이 \[전송 계층] (https://ko.wikipedia.org/wiki/%EC%A0%84%EC%86%A1\_%EA%B3%84%EC%B8%B5)을 정리하고 넘어왔다.

* 4계층 - 전송 계층 → TCP, UDP ⇒ Port number

전송 계층에는 여러 프로토콜도 있지만 TCP, UPD 가 대표적인 프로토콜이다. 이에 대해 알아보자.

### TCP 와 UDP 란?

TCP 와 UDP 는 **연결 여부**를 가지고 비교할 수 있다. 연결을 하고 안하고의 차이로 특징이 나뉜다.

TCP 는 마치 전화기같다. 연결이 필요하고, 연결한 후에 여러 개의 메시지를 보내면 순서를 보장한다. 이는 전화를 연결해서 우리가 말을 하면 어순이 바뀌지 않고 순서대로 전달되는 것과 같다. 그리고 확실하게 도착하는 것을 보장해서 전달한다.

반대로 UCP 는 마치 편지같다. 연결을 하지 않고 데이터를 보낸다. 순서가 보장되지 않는다. 확실하게 도착하는 것을 보장해서 전달하지 않는다.

* TCP
  * 전화같다.
  * 연결이 필요하다.
  * 전달 및 순서를 보장한다.
* UDP
  * 편지같다.
  * 연결하지 않고 데이터를 보낸다.
  * 전달 및 순서를 보장하지 않는다.

그래서 웹에서 쓰려면 TCP 를 써야 한다. (물론 위에서 말했듯이 최근에는 UDP 를 TCP 처럼 구현하기도 한다.)

## 전송 프로토콜을 쓰기 위해서, Socket 이란?

TCP 와 UDP 같은 프로토콜을 쓰러면 Socket 이 필요하다.

이때, Socket과 Socket API를 구분해야 헷갈리지 않는다. Socket API 안에 Socket 이 있는 것이다.

Socket 과 Socket API 에 대해 알아보자.

> 비트코인은 프로토콜 이름인데, 이 프로토콜 안에서 쓰이는 코인 이름도 비트코인으로 불리고 있는 것과 비슷하다. 헷갈리지 않게 주의하자.

## Socket 과 Berkeley Sockets (Socket API) 란?

이란?

* [네트워크\_소켓](https://ko.wikipedia.org/wiki/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC\_%EC%86%8C%EC%BC%93)
* [Berkeley\_sockets](https://en.wikipedia.org/wiki/Berkeley\_sockets)

전자의 Socket 은 우리가 일반적으로 알고 있는 Socket 이다.

Socket 은 엔드 포인트(종착역)인데, 맞닿는 부분이라고 할 수 있다. 그래서 서버에서의 엔드 포인트와 클라이언트의 엔드 포인트가 연결된다.

이때, 엔드 포인트를 Socket API 를 이용해서 연결 시킨다. Socket API 는 Socket 을 프로그래밍하기 위한 API 이다. 그래서 Socket API 를 이용해서 Socket 을 다룬다고 할 수 있다.

Socket 에 대해 더 자세히 알아보자.

Socket은 기본적으로 파일과 유사하게 다룰 수 있다. 그래서 Socket 은 유닉스의 파일 디스크립터와 비슷하다고 볼 수 있다. (유닉스에서는 파일을 읽고 쓰고 닫는 것을 돕는 Handler 가 있다. 이를 파일 디스크립터 라고 한다.)

Java에서는 I/O에 대한 것들을 전부 통일한다. 예를 들어, 키보드 입력, 화면 출력, 파일 입출력 등 모두 Stream(Java 8에서 도입된 Stream API가 아님에 주의!)으로 다룰 수 있다.

## TCP 통신 순서는?

TCP 프로토콜을 Socket API 를 이용해서 쓰려고 한다. 근데 TCP 는 어떻게 통신할까? 이에 대해 먼저 알아보자.

1. 서버는 접속 요청을 받기 위한 소켓을 연다. → Listen
2. 클라이언트는 소켓을 만들고, 서버에 접속을 요청한다. → Connect
3. 서버는 접속 요청을 받아서 클라이언트와 통신할 소켓을 따로 만든다. → Accept
4. 소켓을 통해 서로 데이터를 주고 받는다. → Send & Receive ⇒ 반복!
5. 통신을 마치면 소켓을 닫는다. → Close ⇒ 상대방은 Receive로 인지할 수 있다.

아래는 단계별로 부가 설명을 적어봤다.

**1단계:** 서버는 '접속 요청을 받기 위해서' 소켓을 열고 기다리고 있는다.

**2단계:** 클라이언트도 소켓을 만든다. 그리고 서버 소켓에 요청을 한다.

**3단계:** `서버의 요청을 받는 소켓` 이 `클라이언트의 요청한 소켓` 의 접속 요청을 받는다. 그 다음에 `서버에서 해당 클라이언트와 통신할 소켓` 을 만들어서, `클라이언트의 요청한 소켓` 과 연결한다.

서버는 여러 클라이언트의 요청을 받아서 처리하기 때문에, `요청을 받는 소켓` 과 `해당 클라이언트와 통신할 소켓` 이 따로 필요하다.

**4단계:** 1, 2, 3 단계는 4 단계를 위한 준비단계라고 할 수 있다.

**5단계:** 서버와 클라이언트 둘 중 누구든 통신을 Close 할 수 있다.

***

👇 단계를 말로만 하니까 복잡하다. 클라이언트만 봐보자. 👇

## HTTP 클라이언트를 간단하게 만들어보자

HTTP 클라이언트를 간단하게 만들어보자.

이는 Java Gradle 이 세팅되어 있다는 전제로 진행하기 때문에 다음과 같이 세팅해준다.

```bash
sojeongyoo@Sojeongui-MacBookAir http-client % gradle init

Select type of project to generate:
  1: basic
  2: application
  3: library
  4: Gradle plugin
Enter selection (default: basic) [1..4] 2

Select implementation language:
  1: C++
  2: Groovy
  3: Java
  4: Kotlin
  5: Scala
  6: Swift
Enter selection (default: Java) [1..6] 3

Generate multiple subprojects for application? (default: no) [yes, no] no

Select build script DSL:
  1: Kotlin
  2: Groovy
Enter selection (default: Kotlin) [1..2] 2

Select test framework:
  1: JUnit 4
  2: TestNG
  3: Spock
  4: JUnit Jupiter
Enter selection (default: JUnit Jupiter) [1..4] 4

Project name (default: http-client): client
Source package (default: client): com.ahastudio.http.client
Enter target version of Java (min. 7) (default: 18):
Generate build using new APIs and behavior (some features may change in the next


> Task :init
To learn more about Gradle by exploring our Samples at https://docs.gradle.org/8.3/samples/sample_building_java_applications.html
```

HTTP 클라이언트는 2, 4, 5 단계를 처리하면 된다.

> 2. 클라이언트는 소켓을 만들고, 서버에 접속을 요청한다. → Connect
> 3. 소켓을 통해 서로 데이터를 주고 받는다. → Send & Receive ⇒ 반복!
> 4. 통신을 마치면 소켓을 닫는다. → Close ⇒ 상대방은 Receive로 인지할 수 있다.

## 1️⃣ Connect

호스트에 Connect 요청을 한다. 호스트는 IP 주소 또는 도메인 이름을 사용할 수 있다. 원래 도메인 이름을 쓰게 되면, 도메인의 경우 DNS 를 활용해서 IP 를 얻는 등 여러가지로 복잡하다. 하지만 다행히도 자동으로 내부적으로 알아서 처리해준다.

### 1. IP 주소와 포트 번호만 알면, 서버에 접속할 수 있다

우리는 IP 주소와 포트 번호만 알면, 서버에 접속할 수 있다.

IP 주소를 알면 해당 기기에 접근할 수 있고,

포트 번호를 알면 해당 기기가 여러 프로그램을 쓰고 있을 때 '해당 포트 번호는 내가 처리할게 라고 하는 서버 프로그램'을 찾을 수 있다.

HTTP의 기본 포트 번호는 80 이다.

```java
int port = 80;
```

2단계에서 클라이언트는 소켓을 만들고 서버에 요청한다고 했지만, 자바에서는 "어차피 바로 연결할 것이기 때문에" 객체 생성이지만 여기서 바로 서버에 접속 요청을 한다. 실패하면 ConnectException 예외 발생시킨다.

```java
Socket socket = new Socket(host, port);
```

## 2️⃣ Request

연결을 했으면 이제 요청을 해야 한다. 요청 메시지를 만들고, TCP로 전송하자.

### 1. 요청 메시지를 만들자

HTTP 요청 메시지는 앞에서 정리했듯이 다음과 같은 구조를 가진다.

> Start line Headers 빈 줄 Body

그래서 다음과 같은 형태로 요청 메시지를 만들자. 후자가 최신 자바의 문법 형태라서 더 많이 쓴다.

⚠️ 마지막에 빈 줄을 넣는 걸 잊으면 안 된다. ⚠️

```java
GET http://example.com/ HTTP/1.1
(빈 줄)


또는

GET / HTTP/1.1
Host: example.com
(빈 줄)
```

Java 코드로 바꾸면 다음과 같다.

```java
String message = """
	GET / HTTP/1.1
	Host: example.com

	""";

또는

String message = "" +
	"GET / HTTP/1.1\n" +
	"Host: example.com\n" +
	"\n";
```

> """ 는 java 에서 여러 줄을 쓰기 위한 표현이다.
>
> URL 을 작성할 때 'example.com' 이 아닌 'example.com/' 으로 작성하자. 끝에 '/' Path 가 생략되도 웹 브라우저에서 자동으로 붙여주지만, 원래 있어야 완전한 URL 이다.

### 2. TCP로 전송하자

이제 HTTP 요청 메시지를 TCP 로 전송하면 된다.

어떻게 전송할 수 있냐면, 소켓에서 Output Stream을 얻어서 쓸 수 있다.

```java
OutputStream outputStream = socket.getOutputStream();
outputStream.write(message.getBytes());
```

> OutputStream 의 특징인데, 전송할 때 String 가 아닌 Byte Array 로 보내야 한다.

문자열을 직접 전송하고 싶다면 Writer를 쓰면 된다. **(이렇게 하는 것을 추천)**

```java
OutputStreamWriter writer = new OutputStreamWriter(socket.getOutputStream());

writer.write(message);
writer.flush();
```

> ⚠️ 내부적으로 버퍼가 있기 때문에 write 를 하면 버퍼에 계속 쌓인다. flush를 이용해서 지워야 한다. flush 를 잊지 말고 사용하자.

## 3️⃣ Response

접속했고 요청했으면, 이제 응답을 받아야 한다.

### 2. 응답을 받아 보자

어떻게 응답을 받을 수 있냐면, 소켓에서 Input Stream을 얻어서 쓸 수 있다.

```java
InputStream inputStream = socket.getInputStream();
```

참고로 Stream 의 특징은 다음과 같다.

Byte 배열을 준비하고, 여기로 데이터를 읽어온다. 만약, 응답 데이터가 우리가 준비한 배열보다 크다면, 반복해서 여러 번 읽는 작업이 필요하다. 이때 준비한 배열을 Chunk(한번에 처리하는 단위)라고 한다.

일단, 지금은 단순하게 하기 위해 여기서는 엄청 큰 배열을 준비해서 한번만 읽어보자.

```java
byte[] bytes = new byte[1_000_000];
int size = inputStream.read(bytes); // 얼마나 읽었는지 값을 리턴해준다.
```

1\_000\_000 Byte 를 잡았는데, 분명 다 쓰지 않았을 것이다. 실제 데이터 크기만큼 Byte 배열을 자르고, 문자열로 변환해 출력하자.

```java
byte[] data = Arrays.copyOf(bytes, size); // size 만큼 잘라진 게 나온다.
String text = new String(data);

System.out.println(text);
```

요청과 Write 와 마찬가지로, 응답에서는 문자열 처리를 위해서 Reader를 쓰면 훨씬 편하다 **(이렇게 하는 것을 추천)**

```java
InputStreamReader reader = new InputStreamReader(socket.getInputStream());

CharBuffer charBuffer = CharBuffer.allocate(1_000_000);

reader.read(charBuffer);

charBuffer.flip();

System.out.println(charBuffer.toString());
```

> CharBuffer.allocate(1\_000\_000) 는 Char Buffer 를 미리 1\_000\_00 을 할당해주는 것이다.
>
> ⚠️ 그리고 toString 으로 CharBuffer 를 읽기 전에 flip 하는 것을 잊지 않아야 한다. ⚠️

## 4️⃣ Close

### 직접 Close 하기

더이상 할 게 없으니 이제 close하면 된다.

```java
socket.close();
```

### 자동 Close 하기

만약 직접 Close 를 하고 싶지 않다면, try-with-resources를 써도 된다.

try 블록을 나가면 자동으로 Close 가 된다.

```java
try (Socket socket = new Socket(host, port)) {
	// Request
	// Response
}
```

## 전체 코드

```java
package com.ahastudio.http.client;

import java.io.*;
import java.net.*;
import java.nio.*;

public class App {
    public static void main(String[] args) throws IOException {

        App app = new App();
        app.run();
    }

    private void run() throws IOException {
        System.out.println("Hello World");

        // 1. Connect
        try (Socket socket = new Socket("example.com", 80)) {

            System.out.println("Connect!");

            // 2. Reqeust
            String message = "" +
                    "GET / HTTP/1.1\n" +
                    "Host: example.com\n" +
                    "\n";

            OutputStream outputStream = socket.getOutputStream();
            Writer writer = new OutputStreamWriter(outputStream);

            writer.write(message);
            writer.flush();

            System.out.println("Request!");

            // 3. Response
            InputStream inputStream = socket.getInputStream();
            Reader reader = new InputStreamReader(inputStream);

            CharBuffer charBuffer = CharBuffer.allocate(1_000_000);

            reader.read(charBuffer);

            charBuffer.flip();

            String text = charBuffer.toString();

            System.out.println(text);
        }

        // 4. Close
        System.out.println("Complete!");
    }
}
```

## 실행 결과

```bash
10:53:35 AM: 실행 중 'run'...

> Task :app:compileJava UP-TO-DATE
> Task :app:processResources NO-SOURCE
> Task :app:classes UP-TO-DATE

> Task :app:run
Hello World
Connect!
Request!
HTTP/1.1 200 OK
Accept-Ranges: bytes
Age: 161705
Cache-Control: max-age=604800
Content-Type: text/html; charset=UTF-8
Date: Sun, 17 Sep 2023 01:53:35 GMT
Etag: "3147526947"
Expires: Sun, 24 Sep 2023 01:53:35 GMT
Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
Server: ECS (laa/7AA2)
Vary: Accept-Encoding
X-Cache: HIT
Content-Length: 1256

<!doctype html>
<html>
<head>
    <title>Example Domain</title>

    <meta charset="utf-8" />
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
        
    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
        background-color: #fdfdff;
        border-radius: 0.5em;
        box-shadow: 2px 3px 7px 2px rgba(0,0,0,0.02);
    }
    a:link, a:visited {
        color: #38488f;
        text-decoration: none;
    }
    @media (max-width: 700px) {
        div {
            margin: 0 auto;
            width: auto;
        }
    }
    </style>    
</head>

<body>
<div>
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples in documents. You may use this
    domain in literature without prior coordination or asking for permission.</p>
    <p><a href="https://www.iana.org/domains/example">More information...</a></p>
</div>
</body>
</html>

Complete!

BUILD SUCCESSFUL in 468ms
2 actionable tasks: 1 executed, 1 up-to-date
10:53:35 AM: 실행이 완료되었습니다 'run'.
```

## 아하! 포인트

Socket 프로그래밍을 처음 해봤다. Socket 프로그래밍보다 로우 레벨을 만지는 일은 없다고 하신다. 그럼 거의 밑바닥을 보게 된 것이다. 별거 없지만 어려웠다.

받은 HTML 내용을 그랙픽적으로 표현하면 웹 브라우저가 되고, 파싱을 하면 크롤러가 되는 등. 여러가지로 쓸 수 있다는 점이 신기했다.

## 다음에는?

HTTP 서버도 직접 만들어보자.
