---
layout: single
title:  "HTTP reqeust method 정리"
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
| Request has body             | YES  |
| Successful response has body | Yes |
| Safe                         | `NO` |
| Idempotent                   | `NO` |
| Cacheable                    | 새 정보가 포함되었을 때만 |
| Allowed in HTML forms        | Yes |

