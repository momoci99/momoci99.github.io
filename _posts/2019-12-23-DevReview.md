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

### 데이터 분석

- 데이터를 크롤링 하기 앞서 크롤링할 데이터가 어떤 구조인지 파악하였다. 크롤링 대상 페이지는 위에서 언급한 [산림조합중앙회 송이공판 현황 페이지](http://iforest.nfcf.or.kr/forest/user.tdf?a=user.songi.SongiApp&c=1001&mc=CYB_FIF_DGS_SNI_02&pmsh_item_c=01&sply_date=20191108&x=19&y=3)중 [공판현황 자세히 보기](iforest.nfcf.or.kr/forest/user.tdf?a=user.songi.SongiApp&c=1002&sply_date=20191108&pmsh_item_c=01&mc=CYB_FIF_DGS_SNI_02)항목이였다.

- 해당 페이지는 단순한 구조로 공판일자 콤보박스에서 원하는 날짜를 선택하고 `이동` 버튼을 누르면 get 요청이 날아가 데이터를 불러오는 구조였다. `URL`을 분석 해보니 아래와 같았다.

`http://iforest.nfcf.or.kr/forest/user.tdf?a=user.songi.SongiApp&c=1002&sply_date=20191108&pmsh_item_c=01&mc=CYB_FIF_DGS_SNI_02`



