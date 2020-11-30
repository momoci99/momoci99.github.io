---
layout: single
title: '스트림에서 캔버스로, 캔버스에서 스트림으로'
tag:
  - js
  - MediaStream
  - getDisplayMedia
---

# MediaStream

웹 브라우저에서 영상, 음성을 다루게 해주는 기술인 `MediaStream`은 활용도가 매우 다양하다. 오늘은 이 MediaStream을 이용하여 stream to canvas to stream 을 해볼것이다.

## Why?

왜 이런걸 하냐고 물어본다면.. 꽤 오래전부터 이슈가 되었던 문제였는데 MediaStream을 MediaStreamRecord.js를 활용하여 MultiStreamRecording을 할때 화질이 특정 해상도 이하부터 급격하게 떨어지는 문제가 있었다.

문제의 MediaStreamRecord.js가 MultiStreamRecording을 할때의 과정을 살펴보자면..

1. 캔버스 생성
2. 녹화할 stream을 가지고 내부에서 `video element` 생성
3. 생성된 `video element`들을 1에서 생성한 캔버스에 일정한 주기로 `drawImage`를 활용하여 투사
4. 투사된 이미지를 [HTMLCanvasElement.captureStream()](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/captureStream)를 사용하여 `stream` 생성
5. 4에서 생성된 `stream`을 MediaRecorder으로 레코딩한다.

위와 같은 과정을 거치는데 특히 4,5 번 과정에서 퀄리티가 급격하게 떨어진다고 예상하여 실제로 테스트를 해보려고 한다.

## Test Flow

1. [getDisplayMedia](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getDisplayMedia) 를 통해 현재 화면의 스트림을 가지고온다.(1920 1080)
2. 1에서 받은 스트림을 canvas에 drawing
3. 이때 캔버스의 크기를 640 480 /
4. 2의 canvas의 스트림을 캡처하여 다른 video tag에 출력한다.

## Result

예상대로지만 canvas크기가 1280 720 아래부터 drawImage 할때 이미지 퀄리티가 하락하였다..

다시한번 원판 불변의 법칙을 확인할 수 있었다..

### 640 480

_Canvas_

![640_480_canvas](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/videoImage/640_480_canvas.png)

_video_

![640_480_canvas](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/videoImage/640_480_video.PNG)

### 1280 720

_Canvas_

![1280_720_canvas](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/videoImage/1280_720_canvas.png)

_video_

![1280_720_canvas](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/videoImage/1280_720_video.PNG)

### 1920 1080

_Canvas_

![1920_1080_canvas](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/videoImage/1920_1080_canvas.png)

_video_

![1920_1080_canvas](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/videoImage/1920_1080_video.PNG)

### Code

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <h3>Origin</h3>
    <video
      id="screen"
      style="width: 600px; height: 600px"
      autoplay
      controls
    ></video>
    <h3>Canvas</h3>
    <canvas id="canvas"></canvas>
    <h3>Video from canvas stream</h3>
    <video
      id="canvas_video"
      style="width: 600px; height: 600px"
      autoplay
      controls
    ></video>
    <script>
      const screen = document.getElementById('screen')
      const canvas = document.getElementById('canvas')
      startCapture()

      const canvas_video = document.getElementById('canvas_video')

      draw(screen, 0, 0, 640, 480)

      canvas.width = 640
      canvas.height = 480

      canvas_video.srcObject = canvas.captureStream()

      async function startCapture() {
        try {
          captureStream = await navigator.mediaDevices.getDisplayMedia({
            video: true,
            audio: true,
          })
          console.log(captureStream.getVideoTracks()[0].getSettings())
          screen.srcObject = captureStream
        } catch (err) {
          console.error('Error: ' + err)
        }
        return captureStream
      }

      function draw(video, dx, dy, width, heigth) {
        const ctx = canvas.getContext('2d')
        ctx.drawImage(video, dx, dy, width, heigth)
        setTimeout(() => {
          draw(video, dx, dy, width, heigth)
        }, 1000)
      }
    </script>
  </body>
</html>
```
