---
layout: single
title: " contenteditable 간단 사용후기"
tag:
  - HTML
  - DOM
---

## 개요

최근에 contenteditable를 사용하여 코멘트 작성 및 유저 멘션 기능을 개발하였습니다. 개발 과정에서 삽질을 많이하여 기록으로 남겨 놓기 위해 이번 글을 작성합니다.

## contenteditable?

웹페이지에서 유저로부터 텍스트를 입력받는 방법에는 보통 input, textarea를 주로 사용합니다. 이 외에 다른 방법이 있는데, div, span, p, label처럼 일반적인 엘리먼트에 텍스트를 입력할 수 있게 해주는 요소가 contenteditable입니다. [https://developer.mozilla.org/ko/docs/Web/HTML/Global_attributes/contenteditable](https://developer.mozilla.org/ko/docs/Web/HTML/Global_attributes/contenteditable)

## **커서가 자꾸 처음으로 되돌아가요**

react에서 contenteditable을 사용하다보면 필연적으로 해당 요소의 innerHTML, innerText 혹은 textContent에 다른 값을 재할당해야하는 경우가 발생합니다. (저도 해당 이슈 때문에 고민을 많이 했었습니다) 가령 입력된 값을 파싱하여 다른 값으로 치환 후 재할당 하거나, 다른 엘리먼트를 추가하는 경우를 예로 들 수 있습니다.

재할당 하는 경우, react는 리렌더링을 수행하고 이로 인해, 맨 처음 위치로 되돌아가거나 없어지는 현상이 발생합니다. 구글링을 해보면 동일 사례가 많이 있다는걸 알 수 있습니다.

이 문제를 해결하기위해 스택 오버 플로에 달린 여러 답변과 chatGPT의 의견을 종합해보니 아래의 결론을 얻었습니다.

- 현재 커서 위치를 어딘가에 저장
- 데이터 갱신 후 저장했던 커서 위치를 원상 복구.

**커서(Caret)의 위치 알아내고 지정하기(Selection & Range)**

web api 중 Selection과 Range를 사용하면 비교적 간단하게 현재 커서 위치 및 유저가 drag를 통해 텍스트를 선택한 위치를 알 수 있습니다.

- Selection ([mdn 링크](https://developer.mozilla.org/en-US/docs/Web/API/Selection))
- Range ([mdn 링크](https://developer.mozilla.org/en-US/docs/Web/API/Range))

아래 블로그에서 도움을 많이 받아 같이 보시면 좋을 것 같아 링크를 걸어둡니다.

[https://happy-playboy.tistory.com/entry/ContentEditable에서-커서Caret-활용하기1-자바스크립트](https://happy-playboy.tistory.com/entry/ContentEditable%EC%97%90%EC%84%9C-%EC%BB%A4%EC%84%9CCaret-%ED%99%9C%EC%9A%A9%ED%95%98%EA%B8%B01-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8)

사용하면서 겪었던 몇가지 이슈 사항이 있어서 간단하게 정리합니다.

- 클릭하지 않은 새로운 창에서 Selection 객체의 anchorNode는 null입니다.
- Range의 setStart, setEnd에 selectNodeContents 의 범위를 벗어나는 값을 전달하면 에러를 throw합니다.
- contenteditable 이 false인 textNode 내부에 커서 포지션 지정을 시도하면 커서가 사라집니다.

## 기타

**innerHTML, innerText, textContext 는 한 가족입니다.**

당연한 이야기지만, 엘리먼트의 innerHTML, innerText, textContent는 독립된 속성이 아니라 하나가 변경되면 나머지 속성 모두 영향을 받습니다. 처음에 이 부분을 제대로 몰라서 값이 바뀌는지 삽질을 했었습니다. 이 문제를 해결하기위해 유저가 입력하는 내용을 따로 저장하기 위해 보이지 않는 별도의 contenteditable 엘리먼트를 생성하여 유저에게 노출되는 contenteditable 엘리먼트의 갱신을 최소화 하였습니다.

**dangerouslySetInnerHTML**

리액트에서 엘리먼트의 innerHTML을 다루기 위한 방법입니다. 손쉽게 html string을 렌더링 할 수 있는 방법입니다. 다만 XSS 공격에 취약해지는 단점이 있습니다. [https://www.npmjs.com/package/sanitize-html](https://www.npmjs.com/package/sanitize-html) 와 같은 라이브러리를 사용하여 의도하지 않은 요소가 들어오는것을 걸러낼 필요가 있습니다.

[https://ko.reactjs.org/docs/dom-elements.html#dangerouslysetinnerhtml](https://ko.reactjs.org/docs/dom-elements.html#dangerouslysetinnerhtml)

사족

- 해당 기능 개발 과정에서 운좋게 naver deview 2023 예약에 성공하여, 스마트 에디터를 개발하면서 겪었던 이슈들을 들을 수 있는 기회가 있었습니다. [https://deview.kr/2023/sessions/573](https://deview.kr/2023/sessions/573) 저에게 정말 큰 도움이되어서 같이 보시면 좋을 것 같아 남겨봅니다.
