---
layout: single
title: 'MediaSteram API'
tag:
  - javascript
  - MediaStream
---

# 개요

최근 프로젝트에서 `WebAPI`중 하나인 `MediaSteram API`를 다루게 되면서 알게된 특이사항을 정리한 글이다.

## MediaStream API

`MediaSteram API`는 오디오와 비디오 데이터 스트리밍을 지원하는 WebRTC 관련 API이다. 이 API를 적절하게 활용하여 웹 브라우저상에서 영상 및 오디오를 처리할 수 있게 된다.

## MediaStream의 몇가지 특징

- 하나의 `입력`과 하나의 `출력` 을 가진다.
- Internet Explorer는 지원이 불가능하다
- 로컬 스트림과 비 로컬 스트림으로 나누어진다. (아래에서 좀더 다루겠다.)

  - 로컬 스트림

    - `MediaDevices.getUserMedia()`로 생성한 스트림
    - `MediaDevices.getDisplayMedia()` 로 생성한 스트림

  - 비 로컬 스트림
    - `<video>`,`<audio>` 와 같은 미디어 요소로 생성된 스트림
    - `Web RTC RTCPeerConnection`으로 획득한 스트림
    - `Web Audio API`의 `MediaStreamAudioSourceNode`로 생성된 스트림

## 스트림 받아오기

웹 브라우저상에서 스트림을 얻을 수 있는 방법은 여러가지이지만, 역시 주로 사용하는 방법은 `MediaDevices.getUserMedia()`를 통해 로컬 MediaStream을 가지고 오는것이다.

```js
async function getMedia(constraints) {
  let stream = null

  try {
    stream = await navigator.mediaDevices.getUserMedia(constraints)
    /* 스트림 사용 */
  } catch (err) {
    /* 오류 처리 */
  }
}
```

물론 아래와 같이 `RTCPeerConnection`을 통해 리모트 MediaStream을 받아올 수 있다.

```js
//RTCPeerConnection을 통해 리모트 스트림을 화면에 출력하는 코드 샘플
const remoteStream = new MediaStream()
const remoteVideo = document.querySelector('#remoteVideo')
remoteVideo.srcObject = remoteStream

peerConnection.addEventListener('track', async (event) => {
  remoteStream.addTrack(event.track)
})
```

## 스트림에 트랙을 추가하여 직접 생성하기

스트림에 video, audio 트랙을 추가하여 직접 스트림을 만들 수도 있다.

```js
const customStream = new MediaStream()
customStream.addTrack(audioStream.track)
customStream.addTrack(videoStream.track)
```

## autoplay와 로컬 스트림, 비 로컬 스트림

위에서 로컬 스트림, 비 로컬 스트림이라고 구분 지은 이유는 스트림을 video tag를 통해 출력시킬때 관련이 있기 때문이다. 크롬은 로컬 스트림 이외에는 기본적으로 autoplay를 막는다.

조건이 조금 복잡하지만 정리하면 아래와 같다.(크롬 한정)

- Muted 속성이 있으면 항상 허용
- 아래 상황에서는 Muted가 아니더라도 autoplay 허용
  - 사용자의 조작이 있을때(클릭, 탭 등)
  - 데스크탑에서는 MEI 값이 특정 수치를 넘었을 때(이전에 재생을 했을때)
  - 홈 화면에 사이트를 추가했거나 데스크탑에 PWA를 설치했을 때
- 최상위 프레임은 자동 재생 권한을 iframe에게 위임하여 사운드와 함께 자동재생을 허용할 수 있다.

아래 링크에 좀더 상세히 설명되어 있다. [Autoplay Policy Changes](https://developers.google.com/web/updates/2017/09/autoplay-policy-changes)

## MediaStreamTrack

`MediaStreamTrack`은 스트림내의 하나의 트랙을 나타내는 인터페이스이다. 명시적으로 audio, video 트랙이 존재하지만 다른 종류의 트랙도 존재할 수 있다.

### 참고하면 좋은글

[Capturing Audio & Video in HTML5](https://www.html5rocks.com/ko/tutorials/getusermedia/intro/)

[MDN-MediaDevices.getUserMedia()](https://developer.mozilla.org/ko/docs/Web/API/MediaDevices/getUserMedia)

[https://webrtc.org/getting-started/remote-streams](https://webrtc.org/getting-started/remote-streams)

[MEI](https://developers.google.com/web/updates/2017/09/autoplay-policy-changes#mei)
