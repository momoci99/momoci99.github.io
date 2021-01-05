---
layout: single
title: 'WebRTC'
tag:
  - WebRTC signaling and ICE connection establish
---

WebRTC 시그널링 및 ICE 세션 수립 과정을 간단하게 정리하였습니다.

### 참고자료

[Signaling_and_video_calling#Signaling_transaction_flow](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Signaling_and_video_calling#Signaling_transaction_flow)

### 시그널링 트랜잭션 flow

![https://media.prod.mdn.mozit.cloud/attachments/2016/01/27/12363/9d667775214ae0422fae606050f60c1e/WebRTC%20-%20Signaling%20Diagram.svg](https://media.prod.mdn.mozit.cloud/attachments/2016/01/27/12363/9d667775214ae0422fae606050f60c1e/WebRTC%20-%20Signaling%20Diagram.svg)

1. Naomi (Caller) Web App
   1. `RTCPeerConnection` 생성
   2. `getUserMedia()` 로 웹캠 및 마이크 접근시도
   3. 접근 성공 : 로컬 스트림을 `RTCPeerConnection.addStream()` 로 추가함.
2. Naomi (Caller) Web Browser
   - 협상 할 준비가되었으므로 발신자에게 시작하도록 요청함.
3. Naomi (Caller) Web App
   - handleNegotiationNeededEvent()
   1. RTCPeerConnection.createOffer() 를 통해 SDP offer를 생성
   2. 수행 완료 : `RTCPeerConnection.setLocalDescription()` 을 호출하여Naomi의 호출 종료 설명을 설정.
   3. 수행 완료 : Priya에게 시그널링 서버를 통해 sdp offer를 'video-offer' 유형의 메시지로 전달

---

ICE 레이어에서 Priya에게 ICE 후보를 보냄

---

1. Signaling Server
   - `video-offer` 메시지를 받고 Priya에게 전달

---

1. Priya (Callee) Web App
   - handleVideoOfferMsg()
   1. `RTCPeerConnection` 생성
   2. 수신받은 SDP offer를 통해 `RTCSessionDescription` 생성
   3. `RTCPeerConnection.setRemoteDescription()`을 호출하여 WebRTC에게 Naomi's의 설정값을 알려줌
   4. `getUserMedia()` 로 웹캠 및 마이크 접근시도
   5. 접근 성공 : 로컬 스트림을 `RTCPeerConnection.addStream()` 로 추가함.
   6. 수행 완료 : `RTCPeerConnection.createAnswer()` 를 호출하여 Naomi에게 전달할 SDP 응답을 생성
   7. 수행 완료 : `RTCPeerConnection.setLocalDescription()` 호출하여 생성 된 응답과 일치하게 Priya의 연결 끝 구성
   8. 수행 완료 : Naomi에게 시그널링 서버를 통해 sdp offer를 'video-answer' 유형의 메시지로 전달

---

1. Priya (Callee) Web Browser
   - ICE 레이어에서 Naomi에게 후보를 보냄

### ICE 후보 교환 과정

- 양쪽 peer가 시그널링 서버를 경유하여 ICE 협상이 완료될때까지 서로의 ICE 후보를 교환함

![https://mdn.mozillademos.org/files/12365/WebRTC%20-%20ICE%20Candidate%20Exchange.svg](https://mdn.mozillademos.org/files/12365/WebRTC%20-%20ICE%20Candidate%20Exchange.svg)

1. Naomi(Caller) - Web Browser
   - SDP 문자열로 표현되는 ICE 후보를 생성
2. Naomi(Caller) - Web Browser
   - handleICECandidateEvent()
   - 후보를 받고 이를 'new-ice-candidate' 메시지로써 시그널링 서버를 경유하여 Priya's 의 클라이언트로 전달함.
3. Signaling Server
   - 'new-ice-candidate' 메시지를 받고 Priya로 전달
4. Priya(callee) - Web App
   - handleNewIceCandidateMsg()
   1. 제공받은 후보내의 SDP를 사용 `RTCIceCandidate` 객체 생성
   2. `RTCPeerConnection.addIceCandidate()` 를 사용하여 Priya의 ICE layer에 후보를 전달.

---

1. Priya (Callee) - Web Browser
   - SDP 문자열로 표현되는 ICE 후보를 생성
2. Priya (Callee) - Web Browser
   - handleICECandidateEvent()
   - 후보를 받고 이를 'new-ice-candidate' 메시지로써 시그널링 서버를 경유하여 Naomi의 클라이언트로 전달함.
3. Signaling Server
   - 'new-ice-candidate' 메시지를 받고 Naomi로 전달
4. Naomi (Caller) - Web App
   - handleNewIceCandidateMsg()
   1. 제공받은 후보내의 SDP를 사용 `RTCIceCandidate` 객체 생성
   2. `RTCPeerConnection.addIceCandidate()` 를 사용하여 Naomi의 ICE layer에 후보를 전달.

---

둘다 후보에 동의할때까지 양쪽의 ICE layer에서 이 과정이 반복됨.
