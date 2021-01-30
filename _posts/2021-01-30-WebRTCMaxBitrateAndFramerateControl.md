---
layout: single
title: "MediaStreamTrack.applyConstraints 써보기"
tag:
  - MediaTrack
  - ApplyConstraints
  - RTCRtpSender
  - WebRTC
---

WebAPI의 [MediaStreamTrack.applyConstraints()](https://developer.mozilla.org/en-US/docs/Web/API/MediaStreamTrack/applyConstraints)에 관한 글입니다.

# 개요

WebRTC의 영상 품질은 WebRTC에 내장된 엔진에서 결정됩니다. 이를 BWE(BandWidth-Estimation)라고 합니다. 그렇기에 영상 통화 도중에 품질을 임의로 제어하는것에는 제약이 따릅니다만 아예 방법이 없는것은 아닙니다. 이번 글에서는 WebRTC의 영상 통화 도중에 품질을 제어하기 위해 `MediaStreamTrack.applyConstraints()` 및 `RTCRtpSender.setParameters()`를 활용하여 framerate및 bitrate를 활용하여 테스트하였습니다.

- [MediaStreamTrack.applyConstraints()](https://developer.mozilla.org/en-US/docs/Web/API/MediaStreamTrack/applyConstraints)
- [RTCRtpSender.setParameters()](https://developer.mozilla.org/en-US/docs/Web/API/RTCRtpSender/setParameters)

# 테스트 방법

시그널링 서버 없이 로컬 환경에서 임의의 peer to peer 커넥션을 생성하여 framerate 및 bitrate를 제어합니다. 제어후 webrtc-internals에서 수신측의 inbound-rtp 항목의 frame 관련 항목 및 byte 관련 항목을 확인해봅니다.

# 관련 코드

```js
//송신측의 bandwidth를 제어하는 코드
pc1.getSenders().forEach((sender) => {
  if (sender.track.kind === "video") {
    const parameters = sender.getParameters();
    if (bandwidth === "unlimited") {
      delete parameters.encodings[0].maxBitrate;
    } else {
      parameters.encodings[0].maxBitrate = bandwidth * 1000;
    }

    sender
      .setParameters(parameters)
      .then(() => {
        console.log("bandwidth set success");
        bandwidthSelector.disabled = false;
      })
      .catch((e) => console.error(e));
  }
});
```

```js
//송신측의 framerate를 제어하는 코드
frameSelector.disabled = true;

let frameRate = frameSelector.options[frameSelector.selectedIndex].value;

if (frameRate === "unlimited") {
  frameRate = 60;
}

//pc1의 스트림
localStream
  .getVideoTracks()[0]
  .applyConstraints({
    frameRate: {
      max: frameRate,
    },
  })
  .then(() => {
    frameSelector.disabled = false;
    console.log("set framerate success");
  })
  .catch((e) => {
    console.log(e);
  });
```

# 결과
## framerate
- 30 to 20, 30 to 10 모두 이상없이 동작하였습니다.
- stream 자체의 framerate을 제어하다보니 송신측이 보고있는 영상의 프레임이 떨어지는 영향이 있었습니다.
- framerate이 낮아지면서 frameWidth, frameHeight가 상승하는 현상이 관측되었습니다.

## bitrate
- unlimited to 2M, unlimited to 1M 모두 이상없이 동작하였습니다.
- 인코딩 정보를 수정하는것이기 때문에 송신측은 영향을 안받고 수신측에서 송신측이 보내는 영상을 받을 때 영향이 있었습니다.
- bitrate 자체가 낮아지면 framerate도 함께 함소하는것을 확인하였습니다.


