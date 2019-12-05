---
layout: single
title:  "GitHub 저장소 정리 1"
---



이제야 깃헙 페이지 정비가 어느정도 끝나서 글을 작성할 수 있게 되었다. 정비 관련 내용은 별도의 글을 작성하기로 하고 이번에는 GitHub 저장소 정비에 관한 내용을 작성하기로 한다.



![gitRepoPic](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/gitRepo/gitRepoPic.PNG)
#### contributions 내역을 보니 사하라 사막이 떠오를 지경이다

우선 저장소 정비를 하게된 이유는 현재 내가 만든 저장소중에 관리가 안되는 저장소가 많아 contributions 내역에 나무를 심기전에 정리를 할 필요가 있기 때문이다. 그리고 방치해둔 저장소가 많아 보안 알림이 계속 뜨는 문제도 있기 때문이다.


![securityNoti](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/gitRepo/securityNoti.PNG)
#### 관리가 하도 안되서 보안 이슈가 누적되어있다

우선 가장 먼저 정비할 저장소는 인프런에서 제공하는 React강의를 따라 만든 영화 정보 제공 웹 사이트이다. 이 저장소의 문제를 파악한 결과 아래와 같은 문제가 있었다.

[movie_app](https://github.com/momoci99/movie_app)

- 보안 이슈 발생
- 서비스 동작 안함

### 보안 이슈 해결하기

우선 해당 저장소에서 접수되는 보안 알림은 전부 라이브러리의 취약점 관련으로 의존성 업데이트를 하는것으로 쉽게 처리하였다. 

`yarn upgrade`

![warning](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/gitRepo/warning.PNG)

물론 엄청난 Warning 메시지는 덤이다. 그동안 라이브러리들이 죄다 업데이트 되었고 beta버전이 정식 버전으로 변경되었기 때문이다.

의존성 관련 업데이트가 모두 완료된 후 보안 취약점 관련 경고 메시지는 더 이상 볼 수 없었다. 하지만 그보다 더 심각한 문제가 남아있었다.


### 서비스 동작 안함

이게 가장 큰 문제이다. 잽싸게 개발자 도구를 띄워보니 CROS 문제가 발생하여 영화 정보를 가져올 수 없는 상태였다. (물론 개발자 도구를 띄우기 전에 api의 동작성을 확인하였다.)


![crosError](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/gitRepo/crosError.PNG)

##### 문제의 에러메시지


CROS는 `Cross Origin Resource Sharing` 의 약어로 웹 브라우저의 현재 웹 페이지가 다른 서버의 자원을 호출하는 행위를 말한다. 

이와 관련하여 아래 링크에 좋은 내용이 있어 올린다.

[CORS, Preflight, 인증 처리 관련 삽질-김형준](https://www.popit.kr/cors-preflight-%EC%9D%B8%EC%A6%9D-%EC%B2%98%EB%A6%AC-%EA%B4%80%EB%A0%A8-%EC%82%BD%EC%A7%88/)

우선 내가 겪은 문제의 원인은 CROS가 아니였다. 결론만 말하자면 api링크가 변경되어 발생한 것으로 이 문제의 원인과 해결은 인프런에서 찾을 수 있었다.

하지만 CROS 관련 내용을 찾아보면서 관련 지식을 알게 되었고 수집한 정보를 바탕으로 내가 겪고 있는 문제의 원인이 CROS와는 관련이 없다는것을 알 수 있었다.

왜냐하면 해당 api의CROS 문제가 발생하였다면 당연히 이 api를 사용하고있는 다른 웹 서비스의 장애가 발생하였을 것이고 관련 정보가 올라와야하는데 찾아볼 수가 없었기 때문이다. 단 하나의 링크를 제외하고는..

[인프런 링크-fetch problem](https://www.inflearn.com/questions/5362)

2019-12-05 기준 맨 마자막 답변을 보면 링크가 변경되었다는것을 볼 수 있다. 그리고 위의 답변을 보면 그동안 api 주소가 몇번 변경된것도 알 수 있다.

결국 만든 웹 서비스가 호출하고있는 api의 주소를  수정하여 문제를 해결할 수 있었다.

그리고 이번 저장소 외 다른 저장소들도 하나하나 정리하거나 private 처리를 진행할 것이다. 

끝
