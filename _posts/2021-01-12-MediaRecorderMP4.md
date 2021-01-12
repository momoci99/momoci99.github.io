---
layout: single
title: "Media Recorder mp4"
tag:
  - MediaRecorder
  - WindowMediaPlayer
---

MediaRecorder, mp4, WindowMediaPlayer에 관한 글입니다.

# 개요

[MediaRecorder](https://developer.mozilla.org/en-US/docs/Web/API/MediaRecorder)는 `MediaStream`을 녹화하는 Web API입니다. 최근 회사에서 이 API를 활용하여 레코딩 관련 기능을 개발하던중 흥미로운 이슈를 발견하여 정리합니다.

이슈 내용은 MediaRecorder로 녹화된 영상을 mp4로 저장하면 WindowMediaPlayer에서 건너뛰기, 뒤로가기가 안된다는것이였습니다.물론 다른 플레이어(다음 팟 플레이어, KM 플레이어, 곰플레이어 등등..)에서는 문제가 없었습니다.

# 왜이래 이거

처음에는 WindowMediaPlayer자체문제가 아닐까하여 아래의 방법을 시도하였습니다.

1. 설정 초기화
2. DLL 추가

물론 해결이 안되었습니다.

혹시 생성된 파일 헤더에 문제가 있는가싶어 Hex Editor로 MediaRecorder로 생성된 파일과 구글링하여 받은 예제 mp4파일을 열어보니 차이를 확연히 알 수 있었습니다. 이런!

MediaRecorder로 생성한 파일
![WebM](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/Etc/header_webm.PNG)


예제 파일
![mp4](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/Etc/header_mp4.PNG)

분명 mp4로 저장한다고 생각하였는데 실제로는 WebM 컨테이너로 저장된것이 보이네요.

# 원인

```js
//미디어 레코더 시작
mediaRecorder = new MediaRecorder(stream, {
  mimeType: "video/webm; codecs=vp9",
});
mediaRecorder.start();

...

//다운로드 관련 코드
var blob = new Blob(this.recordedChunks, {
  type: "video/mp4",
});

var url = URL.createObjectURL(blob);
var a = document.createElement("a");
document.body.appendChild(a);
a.download = "result.mp4";
a.click();
window.URL.revokeObjectURL(url);
```
원인은 MediaRecorder에 mimeType를 `video/webm`으로 지정하고 녹화종료후 다운로드할때 Blob의 mimeType을 `video/mp4`로 지정하는 부분이 문제였습니다. blob의 mimeType을 `mp4`로 지정하였다 한들 실제 파일의 컨테이너가 변경되는게 아니였기 때문이죠.

이때문에 WebM 컨테이너 파일이지만 파일 형식은 mp4였기때문에 헤더 불일치 문제로 WindowMediaPlayer가 재생은 했지만 제대로 파일을 못읽어서 건너뛰기, 뒤로감기가 동작하지 않는 문제가 있는것이라고 추측됩니다.

# 우회적인 해결방법?
MediaRecorder의 mimeType에 `video/mp4`를 지정하여 해결이 된다면 참 좋겠으나 안타깝게도 MediaRecorder는 mp4 컨테이너를 지원하지 않습니다. 만약 `video/mp4`를 지정하여 레코딩을 시도하면 지원하지 않는 유형이라는 에러를 반환합니다.

근본적인 문제 해결은 어려우니 아래의 방법을 고려해볼수 있겠네요.


방법 1. 브라우저에서 WebM으로 인코딩하고 컨버팅 서버를 두어 mp4로 변환
- 가장 이상적인 방향입니다. 서버에서 ffmpeg로 컨버팅하면 간단합니다.

방법 2. https://github.com/ffmpegwasm/ffmpeg.wasm 로 mp4 컨버팅
- 브라우저에서 변환하는 방법입니다. 별도의 서버가 필요없겠지만 클라이언트의 성능이 일정 이상이어야하며, 무엇보다 도중에 브라우저를 종료하면 컨버팅과정이 모두 날아가버립니다. 


# 결론
- MediaRecorder는 mp4 컨테이너를 지원하지 않습니다.
- 당연하다면 당연하지만 mimeType만 변경한다고 해서 실제 영상 파일의 내용이 바뀌는건 아니라는 부분이네요.




