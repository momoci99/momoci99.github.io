---
layout: single
title:  "GitPage에 구글 애널리틱스 적용"
tag : 
    - GoogleAnalytics
---


GitPage의 셋팅도 슬슬 마무리 되어가고 있어 구글 애널리틱스를 적용하기로 하였다. 적용에는 큰 어려움이 없어 금방 적용할 수 있었고 현재 이상없이 잘 동작하고 있다.

- 적용과정


![img1](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/google_analytics/img1.png)

우선 구글 애널리스틱에 접속한다. 이때 구글 계정은 필수이다.
[링크](https://www.google.com/analytics/web/?hl=ko&pli=1)


</br>

![img2](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/google_analytics/img2.png)
계정명을 입력한다.

</br>


![img3](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/google_analytics/img3.png)
계정명을 입력한 후 `다음` 버튼을 클릭하여 다음과정으로 넘긴다.

</br>

![img4](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/google_analytics/img4.png)
애널리틱스에서 측정될 대상 유형을 정한다. GitPage는 `웹`에 해당하니 웹을 선택하고 다음으로 넘긴다.

</br>

![img6](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/google_analytics/img6.png)
웹사이트 명과 URL를 입력한다. 이때 URL에는 GitPage 주소를 입력한다. 보고시간대는 한국으로 설정한다.

</br>

![img7](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/google_analytics/img7.png)
약관 동의 항목인데 이미지에는 미국으로 나오지만 한국으로 설정하여 체크할 수 있도록하자. 한국으로 설정해야 약관 내용이 한국어로 나온다.


</br>

![img8](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/google_analytics/img8.png)
약관에 동의하였다면 정상적으로 `추적 코드`가 생성된다 이 추적코드는 아래의 `_config.yml 파일에 들어가게 된다.
</br>

![img9](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/google_analytics/img9.png)
나는 minimal-mistakes 라는 테마를 사용하고 있고 해당 테마에서 제공하는 설정법을 따라 추적 코드 및 관련 설정을 진행하였다. 구글 애널리틱스를 지원하는 테마의 경우 추적 코드를 넣을 수 있게 설정하는 부분이 존재한다. 


</br>


![img10](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/google_analytics/img10.png)
정상적으로 작업이 끝나 모니터링이 잘 수행되고 있는 상태이다.

## 마무리
애널리틱스를 적용하는데 큰 어려움 없이 금방 적용할 수 있었다. 좀더 사용해보면서 기능을 익혀가야하겠다. 그리고 애널리틱스 적요이 완료되었으므로 구글 블로거에 있던 글들도 이동시켜야겠다.
