---
layout: single
title:  "React toy 프로젝트 개발 회고"
tag : 
    - DevReview
---

# React Toy Project - Songi Stock 

이때까지 배운 Reactjs를 실제로 활용하여 작은 프로젝트를 진행하였다. 개발이 어느정도 마무리되어 AWS EC2 인스턴스에 Deploy 및 도메인 연결까지 진행되었다. 진행하면서 얻은 경험과 기록하고싶은 내용들을 여기에 정리하고자 한다.


긴 글에는 3줄 요약

- 내가 불편해서
- 기술스택을 활용하여
- 송이 통계 사이트 만듬 끝. [사이트 링크](http://www.songistock.net)

## 아이템 선정 사유

우리 가족은 해마다 가을철에 [송이버섯](https://ko.wikipedia.org/wiki/%EC%86%A1%EC%9D%B4)을 채취한다. 채취한 송이버섯은 여러 경로로 판매하게 되는데 이때 중요한 요소가 바로 `가격`이다. 송이버섯의 수매 가격은 그날그날 달라진다. 수매가에 따라 공판장 / 개인 판매비율을 결정하는데 수매가는 [산림조합중앙회](http://iforest.nfcf.or.kr/forest/user.tdf?a=user.songi.SongiApp&c=1001&mc=CYB_FIF_DGS_SNI_02&pmsh_item_c=01&sply_date=20191108&x=19&y=3)에서 볼수 있다. 

그 외에 아래의 링크에서도 관련 정보를 조회해 볼 수있다.

- [임산물 유통정보 시스템](https://www.forestinfo.or.kr/public/mjif/selectMjifPrinfoGradeList.do?pageNo=11110400)

- [임산물 가격정보 시스템](https://fps.kofpi.or.kr/fnt/price/forest/area/list.do)

링크를 들어가보면 알겠지만 보기가 좀 불편했다. 특정 날짜의 송이 가격 정도야 쉽게 알 수 있겠지만 내가 원하는것은 년, 월, 일 단위로 가격의 변동을 한눈에 보고 싶었다. 가격 변동을 보면 언제 어느 날짜에 가격이 상승/하락 했는지 바로 알수있고 이날에는 송이 품질을 결정하는 요소가 어떠했는가를 추적하기 쉽기 때문이다. (날씨, 습도, 온도)

송이 이야기가 좀 길었던것같다. 요약하면 내가 불편하면 내가 만들면 되지라는 생각으로 만들게 되었다.


### 사용된 기술 스택과 관련된 코멘트

#### Visual Studio Code 1.41.1
    
- 가장 자주 사용하는 도구이다. 코딩 관련 도구중에는 손꼽히는 도구라고 생각한다. 지금 이글도 VSCode로 작성중이다.


#### JetBrain PyCharm 2019.2.3
- Visual Studio Code도 좋지만 PyCharm도 이에 버금가는 강력한 IDE라고 생각한다. Python 코드를 작성하는데 사용하였다.

#### Python 3.7
- 무언가 가볍고 빠르게 프로그램을 개발하고 싶다면 역시 Python만한게 없는것같다. 참고로 Python2는 올해(2019)까지가 마지막이기 떄문에 Python 3를 사용하였고 송이 공판현황 사이트 크롤링 및 트위터 업데이트 코드를 작성하였다.

- 이번 프로젝트에 Python을 깊게 사용하지는 않았지만 덕분에 가벼운 수준의 Phyton 사용 경험을 습득할 수 있었다.

#### Javascript (Ecmascript 2015 - ES6)
    
- 현 세대에서 웹페이지를 구성하는데 javascript는 거의 필수요소가 되었다. 시작은 미미하나 그 끝은 창대하리라는 말이 떠오를 정도로 javacript는 눈부시게 발전하였고 사용 영역은 웹을 벗어난지는 이미 오래이다. ReactJS는 javacript lib이기 때문에 React를 사용하고 싶다면 반드시 사용해야하는 언어이다.

    
#### React JS
- 이 프로젝트를 시작하게 된 원인중 70%를 차지하고 있다. React JS로 개발을 꼭 해보고 싶었고 실제로 사용하게 되었다. 비록 매우 깊게 다루지는 못하였지면 여러 새로운 개념들을 배울 수 있었고 앞으로도 계속 다루고 싶고 사용할 예정이다.


#### NodeJS v12.14.0 LTS(with Koa)


- [Chrome V8 JavaScript 엔진](https://v8.dev/)을 기반으로한 런타임인 NodeJs를 백엔드에 사용하였다. 여러 장점이 있지만 javascript로 개발을 할 수 있다는점이 내가 NodeJs를 채택한 사유이고 또 NodeJS로 개발을 해보고 싶었기 때문이다.



#### MongoDB 4.0 to 4.2.2

- NoSQL기반 DB이다. NodeJS 및 Python과의 궁합도 좋고 또 사용해보고 싶어서 사용하게 되었다. 

- 사용해보고싶어서 사용하긴 했는데 NoSQL에 대한 개념을 알고 실제 쿼리를 작성하는데 많은 시간이 소요되었다. 만약 MongoDB를 사용하지 않고 RDBMS인 `MSSQL`이나 `MySQL`을 사용하였다면 개발기간이 절반으로 줄어들었을것이다.

- 비록 익숙해지는데 많은 노력과 시간이 들었지만 NoSQL 기반 DB의 개념을 조금이나마 이해할 수 있어 큰 도움이 되었다.

#### AWS(Amazon Web Services) with EC2, Route 53
- 아마존에서 제공하는 매우 강력한 클라우드 컴퓨팅 플랫폼이며 이것또한 꼭 써보고싶어 사용하게 되었다. 마침 프리티어 기간이 아직 남아 있어서 비용 부담도 거의 없었다. EC2 인스턴스를 구성하였고 도메인을 구입하여 인스턴스와 도메인을 연결하여 웹페이지를 구성할 수 있엇다.


## 개발과정 with 삽질

### 데이터 분석 & 크롤링

- python3.7

데이터를 크롤링 하기 앞서 크롤링할 데이터가 어떤 구조인지 파악하였다. 크롤링 대상 페이지는 위에서 언급한 [산림조합중앙회 송이공판 현황 페이지](http://iforest.nfcf.or.kr/forest/user.tdf?a=user.songi.SongiApp&c=1001&mc=CYB_FIF_DGS_SNI_02&pmsh_item_c=01&sply_date=20191108&x=19&y=3)중 [공판현황 자세히 보기](iforest.nfcf.or.kr/forest/user.tdf?a=user.songi.SongiApp&c=1002&sply_date=20191108&pmsh_item_c=01&mc=CYB_FIF_DGS_SNI_02)항목이였다.

해당 페이지는 단순한 구조로 공판일자 콤보박스에서 원하는 날짜를 선택하고 `이동` 버튼을 누르면 get 요청이 날아가 데이터를 불러오는 구조였다. 파라미터를 확인 해보니 아래와 같았다.

![songi_url_query](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/dev_review_songi_stock/songi_url_query.PNG?raw=true)

다른 파라미터는 어떤 역할을 하는지 잘 모르겠지만 `sply_date` 가 조회할 날짜를 의미하는것은 바로 알 수 있었다. 쉽게 생각해서 url에 `sply_date` 파라미터에 날짜만 변경하여 요청을 계속 보내면 원하는 데이터를 얻을 수 있을것 같았고 실제로 요청을 날리자 정상적으로 데이터를 받아 MongoDB의 DB에 저장할 수 있었다. [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/)을 이용하였으며 결과를 파싱후 MongoDB에 저장될 데이터 스키마도 최대한 단순하게 만들었다.

```py
# 데이터 스키마
    songiDic = {
        "_id":"",# DB _id
        "SALE_DATE": "", #공판 일자
        "AREA": "", #지역명
        "UNION":"", #조합명
        "BEFORE_SUMMARY_KG": 0, #전일까지의 수량
        "BEFORE_SUMMARY_PRICE" : 0, #전일까지의 가격
        "TODAY_KG" : 0, #금일 수량
        "TODAY_PRICE" : 0, #금일 가격
        "SUM_KG" : 0, #누계 수량
        "SUM_PRICE" : 0, #누계 가격
        "CLASS_A_KG": "", #1등품 수량
        "CLASS_A_PRICE": "", #1등품 가격
        "CLASS_B_KG": "", #2등품 수량
        "CLASS_B_PRICE": "", #2등품 가격
        "CLASS_C1_KG": "",#3등(생장정지품) 수량
        "CLASS_C1_PRICE": "",#3등(생장정지품) 가격
        "CLASS_C2_KG": "", #3등(개산품) 수량
        "CLASS_C2_PRICE": "", #3등(개산품) 가격
        "CLASS_D_KG": "", #등외품 수량
        "CLASS_D_PRICE": "",#등외품 가격
        "CLASS_F_KG": "", #혼합품 수량
        "CLASS_F_PRICE": "", #혼합품 가격
        "LAST_UPDATED" : ""#최종변경일시
    }

```


### 백엔드 코드 작성 Nodejs

- Koa

Nodejs 특성상 javascript로 코드를 작성할 수 있어 비교적 수월하게 코드를 작성할 수 있었다. 다만 미들웨어로 [Koa](https://koajs.com/)를 사용하였다. Expresss또한 자료가 많았지만 `Koa`로 갈아타고 있는 추세라고 하여 Koa를 활용하였다. 사실 이전에도 Express 로 간단한 코드를 몇번 작성한적이 있어 Express가 더 익숙한것인 사실이나 koa를 한번 사용하고 싶었다.

koa와 Express 둘다 겉으로는 비슷한 역할을 수행하나 라우팅이라던지 파라미터 구성, 들어온 요청을 처리하는 방식에 차이가 있었다. 물론 koa API 문서를 참고하여 잘 활용할 수 있었다.

아래 블로그도 참조하였다

[express-vs-koa](https://velog.io/@noyo0123/express-vs-koa-qyk24lsozz)

- async await

Nodejs를 사용하면서 [CallBack Hell](http://callbackhell.com/)이란 말을 자주 접하게 된다. 이는 Nodejs의 특성에 기인한것인데 일반적인 동기 방식과 달리 Nodejs는 비동기 이벤트 주도 JavaScript 런타임이다. 따라서 비동기 이벤트를 처리하기 위해 es6의 `Promise`가 등장 하기 전까지는 CallBack 함수를 주렁주렁 달거나 별도의 lib를 사용해야 했다. 나도 Promise까지만 사용해봤고 그 이후에 등장한 async await는 접해볼 길이 없었는데 이번 기회를 통해 사용하게 되었다.

- 같이 보면 좋은 블로그

[Node.js 이벤트루프 제대로 이해하기](https://tk-one.github.io/2019/02/07/nodejs-event-loop/)

[js-async-await](1https://joshua1988.github.io/web-development/javascript/js-async-await/)

[google-async-functions](https://developers.google.com/web/fundamentals/primers/async-functions?hl=ko)


### MongoDB

NoSQL 기반의 MongoDB는 평소에 이름만 알았지 실제 프로그램 개발에 사용하는것은 이번이 처음이였다. 우선 SQL 기반 RDBMS인 MySQL, MSSQL에 익숙해져 있던 터라 NoSQL 방식의 쿼리를 작성하는데 매우 큰 애를 먹었다. 만약 RDBMS를 사용했다면 개발 기간이 크게 단축되었을것이다.(..) 그래도 MongoDB의 워크벤치라고 할수있는 MongoDB Compass로 쿼리를 미리 구성하고 적용하는것으로 원하는 쿼리를 작성할 수 있었다.

- [Aggregation](https://docs.mongodb.com/manual/aggregation/)

### ReactJS

크게 삽질한 부분은 React 16.8에 추가된 [Hooks](https://ko.reactjs.org/docs/hooks-intro.html) 였다. `setState`를 대신하여 상태관리를 자동으로 해주게 되었다. 아직 제대로 이해하고 있는것은 아니지만 기존 생명주기 관련 함수들 대신 `useEffect()`가 대체하게 되고 필요에따라 DOM을 업데이트하게 되는것으로 간단하게 이해하고 개발했다. 개발 시간을 늘린(!) 주요 사항중 하나였지만 덕분에 상태관리에 대한 개념을 익힐 수 있었다.

#### AWS

최초에는 계획에 없었고 라즈베리파이에 서버를 올릴 계획이었으나 AWS도 한번 사용하고 싶은 마음에 EC2 인스턴스를 생성하고 도메인까지 구입하였다. 의외로 크게 삽질할 요소는 없었다. AWS 자체 안내서가 매우 잘 되어있기 때문이다.

도메인의 경우 아래 그림과 같이 가격대에 맞게 구입이 가능하다. 나는 제일 저렴한 11$(1년)의 .net 을 구입하였다.

![domain_buy](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/dev_review_songi_stock/domain_buy.PNG?raw=true)

도메인과 인스턴스를 연결하기 위해서는 [route 53](https://aws.amazon.com/ko/route53/)에서 연결하려는 인스턴스와 도메인간의 쿼리를 생성하면 된다.

![route53](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/dev_review_songi_stock/route53.PNG?raw=true)



## 마무리

toy project로 시작해서 어떻게 잘 deploy시킨것같다. 물론 아직 버그나 개선 사항이 많이 남아 있어 조금씩 잡아나갈 예정이다. 아쉬운점으로는 모바일 지원이 안된다는점이다. 좀더 열심히 해야겠다.