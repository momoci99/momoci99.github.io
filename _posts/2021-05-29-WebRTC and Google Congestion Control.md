---
layout: single
title: "WebRTC 혼잡 제어 알고리즘 & GCC"
tag:
  - WebRTC
---

WebRTC & Google Congestion Control

# 개요

이번 글에는 WebRTC의 혼잡 제어 알고리즘과 WebRTC에서 널리 쓰이는 Google Congestion Control 알고리즘에 대해 다루어 보았습니다.

# 혼잡 제어 알고리즘?

WebRTC는 네트워크에 트래픽이 많거나, 처리가 지연되는 경우를 대비하여 흐름을 제어하는 알고리즘이 탑재되어 있습니다. 이를 혼잡 제어 알고리즘(Congestion Control Algorithm)이라고 합니다.

## GCC - Google Congestion Control

현재 Google Chrome과 Google Hangout에서 사용되고 있는 실질적인 혼잡 제어 알고리즘입니다. 실질적이라고 표현한 이유는, NADA, SCReAM과 같은 알고리즘이 있지만, 당장 사용되고 있는것은 GCC이기 때문입니다.

GCC는 지연과 패킷 손실률을 파악하여 유동적으로 데이터 흐름을 제어합니다. 이로인해 화질이 갑자기 나빠졌다가, 시간이 지나면서 좋아지거나 하는 이유이죠.

# GCC에 대해 좀 더 알아봅시다.

GCC는 어떤 방식으로 혼잡을 감지하고, 어떻게 혼잡을 제어하는걸까요?

## 혼잡을 감지하고 제어하는 방식

GCC는 회선의 가용 용량을 최대한 활용하고, 대기 대기열을 최대한 줄이려고 합니다.

GCC의 동작 목표는 현재 회선의 가용 용량을 최대한으로 사용함과 동시에, 패킷 대기 대기열을 최대한 줄이려고 동작합니다. 이 목표를 달성하고자 지연 기반 제어와 손실 기반 제어를 수행합니다. GCC는 이 두 가지 방식을 상황에 따라 사용하고 있으나, 지연 기반 방식이 좀 더 세분화 되어있습니다

---

### 1. 지연 기반 제어(Delay-based controller)

GCC는 회선이 포화될때 까지 bitrate를 계속 증가시킵니다. 만약 이 과정에서 회선의 가용 용량을 모두 사용하여 대기 대기열이 발생하는 경우, 대기열이 0이 될 때 까지 bitrate를 감소킵니다. 회선에 다시 여유가 생긴다면 bitrate를 다시 증가시킵니다. 참고로 지연 기반 제어는 사전 필터링, 도착 시간 필터, 과용 감지기, 속도 제어로 구성됩니다.

지연 기반 제어가 동작하여 bitrate를 제어하는 기준을 정리하면 다음과 같습니다.

1. 지연 임계 값 초과(12.5ms)
2. 대기 대기열 발생

### 2. 손실 기반 제어(Loss-based controller)

송신측은 수신측이 보내는 각 정보(지연, 패킷 손실률 등등)를 토대로 자체적으로 사용 가능한 대역폭을 추측합니다. 간단하게 아래의 방식으로 동작합니다.

- 패킷 손실률이 2% 이하일 때 송신 가능 대역폭 추정치 증가.
- 패킷 손실률이 10% 이상일 때 송신 가능 대역폭 추정치 감소.

### 속도 제어

지연 기반 제어와 손실 기반 제어를 바탕으로 최종적으로 대역폭을 제어합니다. 속도 제어는 상태에 따라 증가, 보류, 감소를 수행합니다.

**사용가능한 대역폭 추정 기준**

- 지연 기반 제어에서 추측된 회선 대역폭
- 손실 기반 제어에서 추측된 회선 대역폭

이 두가지 값 중 낮은 값을 기준으로 합니다.

**신호 정의 - 지연 기반 제어, 손실 기반 제어에서 보고**

- 과다 사용 : 지연 기반 제어에서 과다 사용 신호가 검출.
- 일반 : 과다, 여유 신호 미검출
- 미사용 : 회선에 여유가 있는 신호 검출

**상태 정의 - 속도 제어가 현재 수행하는 작업**

- 증가 - 대역폭을 증가시키는 중
- 감소 - 대역폭을 감소시키는 중
- 유지 - 현재 상태 유지중
- 보류 - 대역폭을 증가시키기 전 대기열이 비워질 때 까지 대기 중

![webrtc_status_table.png](https://raw.githubusercontent.com/momoci99/momoci99.github.io/master/assets/img/Etc/webrtc_status_table.png)

과다 사용

- 과다 사용 감지시 속도 보류 상태 → 속도 감소
- 과다 사용 감지시 속도 증가 상태 → 속도 감소
- 과다 사용 감지시 속도 감소 상태 → 유지(계속 속도 감소)

일반 상태

- 일반 상태 감지시 속도 보류 상태 → 속도 증가
- 일반 상태 감지시 속도 증가 상태 → 유지(계속 속도 증가)
- 일반 상태 감지시 속도 감소 상태 → 보류(이후 속도 증가)

미사용 감지

- 미사용 감지시 속도 보류 상태 → 보류(이후 속도 증가)
- 미사용 감지시 속도 증가 상태 → 보류(이후 속도 증가)
- 미사용 감지시 속도 감소 상태 → 보류(이후 속도 증가)

# 정리

이번 글에서는 WebRTC의 혼잡 제어 알고리즘인 Google Congestion Control 알고리즘의 구성, 동작 방식을 알아보았습니다. 참고 자료를 통해 나름대로 정리 하였으나 틀린 부분이 존재할 수 있습니다. 혹시라도 잘못된 부분이 있으시다면 댓글 달아주세요!

# 참고자료

1. WebRTC의 혼잡 제어 및 표준화 관련 이슈 : [Congestion Control for WebRTC: Standardization Status and Open Issues](https://www.researchgate.net/publication/317620792_Congestion_Control_for_WebRTC_Standardization_Status_and_Open_Issues)
2. WebRTC 영상 흐름 제어 구조 - Google Congestion Control : [https://qiita.com/komasshu/items/1cb5d4469595a635c689?fbclid=IwAR2i8ilRnGFnGfK0S_OuNlXFurPyZXM5iTUrse1Ih-wBQiW3uvzK0GXR-zE](https://qiita.com/komasshu/items/1cb5d4469595a635c689?fbclid=IwAR2i8ilRnGFnGfK0S_OuNlXFurPyZXM5iTUrse1Ih-wBQiW3uvzK0GXR-zE)

3. Performance Analysis
   of Google Congestion
   Control Algorithm for
   WebRTC : [https://repository.tudelft.nl/islandora/object/uuid%3A406dae1c-bc39-4973-9f82-0977a7bacdbf](https://repository.tudelft.nl/islandora/object/uuid%3A406dae1c-bc39-4973-9f82-0977a7bacdbf)

4. [IETF GCC 링크](https://datatracker.ietf.org/doc/html/draft-ietf-rmcat-gcc-02)
