---
layout: single
title:  "HTTP reqeust method 정리(작성중)"
tag : 
    - Http
    - GET
    - POST
---

# HTTP request method

HTTP request는 자주 사용하지만 간혹 헷갈리고 까먹는 부분이 종종 생겨 이참에 정리를 해보고자 한다. 그전에 먼저 method의 성격에 대해 기술한다.

#### 아래의 내용은 HTTP/1.x 에 대하여 기술함.

- Request has body

요청문 내에 본문이 있는지 여부이다. HTTP 요청 메시지의 구조에는 크게 `Start Line`, `Headers`, `Body` 로 나누어져 있고 이중에 Body가 존재하는지를 표현한다.
예를들어 `GET`, `HEAD`, `DELETE`, 또는 `OPTIONS` 요청에는 일반적으로 Body가 존재하지 않는다.


- Successful response has body
성공적인 요청의 응답에 본문이 있는가에대한 여부이다. 요청문에도 그렇듯 응답문에도 일정한 구조가 있는데 `Status Line`, `Headers`, `Body` 이다. 이중에 Body의 존재 유무를 나타내며 `HEAD` 요청은 유일하게 Body가 없다.

- Safe
안전의 의미는 `요청`이 서버의 상태를 변화시키는가에 대한 유무이다. 쉽게 정리하면 만약 요청을 처리하면서 `a = a + 10` 이런식으로 서버내의 어떤 상태를 변경시킨다면 Safe하지 않다.

- Idempotent
직역하면 `멱등성`을 의미하는데, 요청을 수십 수만번을 해도 동일한 결과를 가진다면 멱등성을 가진다고 할수 있겠다. 안전한 함수는 멱등성을 가지지만 안전하지 않은 함수는 멱등성을 가질수도, 가지지 않을수도 있다. 코드로 설명하지면 `a = 10` 으로 표현할수있다. 아무리 호출해도 a 변수에는 10이라는 정수가 할당되기 때문디ㅏ.


- Cacheable
HTTP request가 캐싱이 되는지에 대한 여부를 나타낸다. HTTP caching는 사설 브라우저 캐시 혹은 공유 프록시 캐시에서 이루어지며 캐시는 생명주기를 가진다. 좀더 자세한 내용은 링크를 참조. [Caching](https://developer.mozilla.org/ko/docs/Web/HTTP/Caching)

- Allowed in HTML forms
html의 `<form>` 요소로 사용될 수 있는지 여부를 나타낸다. `GET` 과 `POST` 요청만이 `<form>` 요소에 사용될 수 있다.


## GET

| 항목                         | 유무|
|------------------------------|-----|
| Request has body             | `No`  |
| Successful response has body | Yes |
| Safe                         | Yes |
| Idempotent                   | Yes |
| Cacheable                    | Yes |
| Allowed in HTML forms        | Yes |

`GET` 은 특정한 리소스를 가지고 오도록 요청하며 오로지 데이터를 가져오는데에만 사용된다.

Request 내의 body는 별도로 존재하지 않고 파라미터가 URL에 붙어서 문자열 형태로 전달된다.


예를 들어 URL이 아래와 같다면

> http://www.songistock.net/GetGroupPrice?target=AREA&dateOpt=2019-12-30


서버내의 호출되는 함수명이 GetGroupPrice이고 파라미터명은 target, dateOpt이며 파라미터 값은 각각AREA, 2019-12-30이 되는것이다.

GET에 대해 좀더 기술하자면 GET은 body가 없는 구조이지만 임의로 body를 붙혀보내면 어떻게 될까? 이에 대해 아래의 블로거 분이 잘 정리해주셨다.

- [HTTP GET 요청에 body를 붙여서 보내면 어떤 일이 벌어질까?](https://libsora.so/posts/http-get-request-with-body-and-http-library/)

- [HTTP 요청에 body를 붙여서 보내면 어떤 일이 벌어질까? part 2](https://libsora.so/posts/http-request-with-body-and-java-httpurlconnection/)

그리고 GET은 모든 파라미터가 URL에 담겨지므로 URL의 길이 제한도 고민해볼만한 부분이다. 이에대해 찾아보니 이에 대해 흥미로운 답변을 stackoverflow에서 찾을수 있었다.

[What is the maximum length of a URL in different browsers?](https://stackoverflow.com/questions/417142/what-is-the-maximum-length-of-a-url-in-different-browsers)

가장 많은 reputation을 받은 답변을 참고하여 정리하면 다음과 같다.

- RFC 2616에 정의된 표준에는 길이 제한이 없다.
- 만약 서버가 감당못할 길이의 URI가 들어온다면 (Request-URI Too Long) 를 리턴해야한다.
- RFC 7230에 분리된 표준에는 8000(octets)바이트 까지는 처리할 수 있어야한다고 명시되어있따.

즉 서버는 서버 자신이 감당할수 있는 최대한으로 처리를 해야한다는 뜻이되겠다.

하지만 현실은 그렇지 않다. 브라우저마다 감당할수있는 URL의 길이가 각각 다르고 서버에서도 처리할수있는 길이가 별도로 있기 때문이다.

위의 링크에 걸린 답변을 참고하여 결론을 내리면 2000 chars 내의 수준으로 적절히 제한하는게 맞는것같다.

## POST

| 항목                         | 유무|
|------------------------------|-----|
| Request has body             | Yes  |
| Successful response has body | Yes |
| Safe                         | `NO` |
| Idempotent                   | `NO` |
| Cacheable                    | 새 정보가 포함되었을 때만 |
| Allowed in HTML forms        | Yes |

- `POST` 메서드는 데이터를 서버로 보내는 방법 중 하나이다. request의 body 타입은 Content-Type 헤더에 따라 결정된다. 예를 들어

> application/x-www-for 
    
    POST / HTTP/1.1
    Host: foo.com
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 13

    say=Hi&to=Mom

POST 요청이 위와 같을때(Content-Type: application/x-www-form-urlencoded) html form 을 통해 POST 요청을 한것을 알수 있고 body의 데이터가 `key`=`value` 형태를 가지고 있는것을 알수있다.

> multipart/form-data

    POST /test.html HTTP/1.1 
    Host: example.org 
    Content-Type: multipart/form-data;boundary="boundary" 

    --boundary 
    Content-Disposition: form-data; name="field1" 

    value1 
    --boundary 
    Content-Disposition: form-data; name="field2"; filename="example.txt" 

    value2

이번에는 Content-Type: multipart/form-data; 이다. body의 데이터가 `key`:`value` 형태인것을 알수있다.

이 Content-Type에 따라 body 데이터 형식이 달라지니 유의해야하고 서버에서 처리할때도 유의하는것이 좋다.

- 동작의 유사성으로 `PUT`과 비슷하나 `멱등성`에서 차이가 있다. PUT은 몇번을 호출하던 같은 결과를 가지고 오지만 POST는 호출할때마다 그 결과가 달라진다. 

- POST request는 보통 HTML form 을 통해 서버에 데이터를 전송하고 서버상에 변경사항을 만든다. 예를들어 HTML form에 입력한 데이터는 서버에 전송되고 해당 데이터가 서버에서 적절히 처리되고 변경된다. 

*예제코드*

```html
<form action="/my-handling-form-page" method="post">
    <div>
        <label for="name">Name:</label>
        <input type="text" id="name" />
    </div>
    <div>
        <label for="mail">E-mail:</label>
        <input type="email" id="mail" />
    </div>
    <div>
        <label for="msg">Message:</label>
        <textarea id="msg"></textarea>
    </div>
    
    <div class="button">
        <button type="submit">Send your message</button>
    </div>
</form>

```

- POST는 body에 파라미터가 저장된다 따라서 GET과는 달리 파라미터가 URL에 노출되는 일이 없다. 이는 GET과 POST의 주요 차이점 중 하나이다. 여기서 궁금증이 하나가 떠오르는데 POST request에 GET 처럼 URL에 파라미터를 추가해서 보내는 방식은 어떠한가 대한것이다.

물론 이에 대해 stackoverflow에 질문이 올라와있다. [HTTP POST with URL query parameters — good idea or not? [closed]
](https://stackoverflow.com/questions/611906/http-post-with-url-query-parameters-good-idea-or-not) 의견이 매우매우 분분하지만 결국 다 하는소리가 `가능은 한데 하지마라` 가 주된 의견이다.

이 글에서 나온 의견들을 찬찬히 살펴보면 GET과 POST는 왜 구분되어서 사용되어야 하는지, 두 request method의 차이점까지 드러난다. 

- 서버에서 두가지 방식이 혼용된 요청을 받았을때 그걸 적절히 처리한다는 보장이 없다.
- 프로그래머에게 혼란을 주게 된다.
- GET은 멱등성이 있고 POST는 멱등성이 없다. 이 두가지 방식을 혼용하면 멱등성이 명확하지 않아 혼란만 준다.

정리하면 GET과 POST는 구분되어 사용되어야한다. 이렇게 정리할 수 있겠다.

- POST 방식은 request body에 데이터가 전달되는데 이때 body의 한도는 얼마인가? 라는 의문이 또 든다. 또 구글링해본 결과, GET의 URI 제약이 표준에 없는것처럼 마찬가지로 제한이 없고 브라우저 및 서버에 따라 결정된다.

[Can HTTP POST be limitless?](https://stackoverflow.com/questions/2880722/can-http-post-be-limitless/55998160#55998160)

한번 읽어보면 좋을것같다.



## PUT

## DELETE

## CONNECT

## OPTIONS

## TRACE

## PATCH