---
layout: single
title: "Front End Stream Record Library"
tag:
  - javascript
  - RecordRTC.js
  - MediaStreamRecord.js
---

# 개요

요즘 Front-End 에서 Stream을 녹화하는 작업을 하고있다. 이번 글에는 `MediaDevices.getDisplayMedia()` 함수와 `MediaDevices.getUserMedia()` 에서 발생하는 Stream을 저장하는 과정 및 관련 라이브러리에 대해 다루려고한다. 이에 대해 MDN에서 잘 정리해둔 문서가 있으니 한번 보고 오는것도 좋을것같다. [Recording_a_media_element](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream_Recording_API/Recording_a_media_element)

# Stream From Media

Front-End에서 Stream을 만드는 방법은 지금까지 내가 알고있는 방법으로는 3가지가 있다.

## 1. getUserMedia

[MDN 링크](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia)

클라이언트에 연결된 장치(마이크, 카메라 등등)를 통해 스트림을 생성한다. 이 스트림을 통해 외부 장치로부터 Video, Audio, 혹은 둘다 가지고 올 수 있다.

## 2. getDisplayMedia

[MDN 링크](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getDisplayMedia)

브라우저 혹은 다른 어플리케이션의 화면을 캡처하여 스트림을 생성한다.

## 3. captureStream

[MDN 링크](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/captureStream)

[HTMLMediaElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement)를 통해 스트림을 생성한다. `getUserMedia`, `getDisplayMedia`와 다른점이 있다면 외부 장치가 아닌 HTML내에 존재하는 MediaElement에서 스트림을 생성한다.

# Recording Stream

기본적으로 스트림을 Recording하는것은 전적으로 브라우저 내장 함수인 [MediaRecorder](https://developer.mozilla.org/en-US/docs/Web/API/MediaRecorder)을 사용한다. 밑에 소개할 두 라이브러리인 `RecordRTC.js`, `MediaStreamRecord.js` 모두 내부적으로는 `MediaRecorder`를 활용하고 있기에 내부 코드도 비슷하다. 애초에 [muaz-khan](https://github.com/muaz-khan)라는 개발자분이 두 라이브러리에 깊게 관여해있기 때문이다.

# RecordRTC.js

[RecordRTC.js](https://recordrtc.org/)

오픈소스 기반 스트림 Record 라이브러리이다.

```js
let stream = await navigator.mediaDevices.getUserMedia({
  video: true,
  audio: true,
});
let recorder = new RecordRTCPromisesHandler(stream, {
  type: "video",
});
recorder.startRecording();

const sleep = (m) => new Promise((r) => setTimeout(r, m));
await sleep(3000);

await recorder.stopRecording();
let blob = await recorder.getBlob();
invokeSaveAsDialog(blob);
```

내부적으로는 `MediaRecorder`를 사용하고 있지만 부가적인 Record관련 라이브러리를 사용할 수 있어 제약 사항이 덜한편이다. 그리고 관련 저장소가 계속 관리되고 있는 상태이다.

# MediaStreamRecord.js

[MediaStreamRecorder](https://github.com/streamproc/MediaStreamRecorder)

RecordRTC와 동일하게 오픈소스 기반 Record 라이브러리지만, 관리가 안되고 있고 소스코드를 잘 살펴보면 시간이 지나 API의 코덱 지원 스펙이 늘어났음에도 불구하고 반영이 안되는 문제가 있다.

```js
var mediaConstraints = {
  audio: true,
  video: true,
};

navigator.getUserMedia(mediaConstraints, onMediaSuccess, onMediaError);

function onMediaSuccess(stream) {
  var mediaRecorder = new MediaStreamRecorder(stream);
  mediaRecorder.mimeType = "video/webm";
  mediaRecorder.ondataavailable = function (blob) {
    // POST/PUT "Blob" using FormData/XHR2
    var blobURL = URL.createObjectURL(blob);
    document.write('<a href="' + blobURL + '">' + blobURL + "</a>");
  };
  mediaRecorder.start(3000);
}

function onMediaError(e) {
  console.error("media error", e);
}
```

## 두 라이브러리의 차이

### 있는듯 하면서 없는듯한 차이

### Blob 처리방식

결론만 말하면 Blob을 처리하는 방식에 있다. RecordRTC.js는 Timeslice된 Blob을 파일로 만들면 재생이 불가능하다. 반면 MediaStreamRecorder.js는 Timeslice된 Blob을 파일로 만들게되면 재생이 가능한 차이가 있다.

여기에 muaz-khan이 남긴 코멘트를 본다면 좀더 확실히 알 수 있다. [링크](https://github.com/muaz-khan/RecordRTC/issues/134#issuecomment-228486551)

### 지속적인 지원

RecordRTC는 비교적 활발하게 이슈가 처리되고 있지만 MediaStreamRecorder.js는 안타깝게도 마지막 업데이트가 3년전일정도로 오래된 라이브러리가 되었다. 거기에 NPM에 올라온 버전은 현재 GitHub 버전과 불일치하는 코드이기 때문에 사용상에 주의가 필요하다.
