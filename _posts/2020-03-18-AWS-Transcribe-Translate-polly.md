---
layout: single
title:  "AWS Transcribe / Translate / Polly"
tag : 
    - AWS
---


# AWS Transcribe / Translate / Polly

## 0. 개요

Remote 2.0 에서 추가 계획인 실시간 음성 번역 기능 개발과 관련하여 
AWS가 제공하는 솔루션에 대한 내용입니다.

## 1. 큰 그림

    USER -> Amazon Transcribe -> Amazon Translate -> Amazon Polly -> USER

- USER가 입력한 오디오 데이터를 Amazon Transcribe로 전송
- Amazon Transcribe는 입력받은 오디오를 컴퓨터가 읽을수 있는 json 데이터로 변환
- 변환된 json 데이터는 Amazon Translate로 전송
- Amazon Translate는 입력받은 json 데이터를 분석하여 번역
- 번역된 결과물을 Amazon Polly로 전송
- Amazon Polly는 전달받은 결과물을 TTS로 발성

## 2. 큰 그림에 대한 세부 설명(예제)

![https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2019/06/06/real-time-translator-2.gif](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2019/06/06/real-time-translator-2.gif)

서비스 상호 작용의 흐름은 다음과 같음.

[아래 내용을 실제 구현한 예제](https://d399h0fue2zb49.cloudfront.net/voice-translator.html)

- `Amazon CloudFront` 를 사용하여 사이트에 대한 액세스를 허용하면 페이지에 대한 HTTPS 링크를 얻을 수 있음.
- `Amazon S3`에 브라우저에 기록 된 입력 오디오 파일을 저장.
- 입력 오디오 파일을 S3에 저장하고 Lambda 함수를 호출. 함수 입력에서 이전에 Amazon S3에서 저장 한 오디오 파일의 이름을 제공하고 소스 및 대상 언어 매개 변수를 전달.
- `Amazon Transcribe`를 사용하여 오디오를 텍스트로 변환
- `Amazon Translate`를 사용하여 문자를 한 언어에서 다른 언어로 번역.
- `Amazon Polly`를 사용하여 새로운 번역 된 텍스트를 음성으로 변환.
- `Lambda` 함수를 사용하여 출력 오디오 파일을 S3에 다시 저장 한 다음 파일 이름을 페이지로 반환 (JavaScript 호출). 오디오 파일 자체를 반환 할 수 있지만 간단하게하기 위해 S3에 저장하고 이름 만 반환하면됨.
- 번역 된 오디오를 사용자에게 자동으로 재생.

## 3. 각 솔루션 특징

### 3-1. Amazon Transcribe

> 음성을 텍스트로 자동 변환

[링크](https://aws.amazon.com/ko/transcribe/)

### 특징

- 음성을 텍스트로 변환해주는 솔루션
- 텍스트로 변환하기 위해 음성파일이 `반드시` **S3**에 저장되어 있어야함.
- 거의 실시간 오디오 스트리밍(한국어는 안됨)을 텍스트로 변환할 수 있음. ([지원언어](https://docs.aws.amazon.com/transcribe/latest/dg/what-is-transcribe.html))
- [Amazon Real-time transcription 예제](https://console.aws.amazon.com/transcribe/home?region=us-east-1#realTimeTranscription)
- 여러명이 동시에 말했을때 이를 구분할 수 있음.
- 필터링할 단어를 지정하여 특정 단어(보안 사항)를 제거할 수 있음.
- 변환된 결과물은 아래와 같이 `json` 형식으로 출력됨.
- 16kHz 및 8kHz 오디오 스트림과 WAV, MP3, MP4 및 FLAC를 비롯한 여러 오디오 인코딩을 지원
```json
    {"Transcript": {"Results": [{"Alternatives": [{"Items": [{"Content": "See","EndTime": 0.2199,"StartTime": 0.0699,"Type": "pronunciation"},생략...{"Content": "how","EndTime": 0.6099,"StartTime": 0.2299,"Type": "pronunciation"},{"Content": "much","EndTime": 0.7999,"StartTime": 0.6199,}],"Transcript": "See how much a transcribed Rachel test copy of the speech into your time. Choose started dreaming and talk."}],"EndTime": 7.05,"IsPartial": false,"ResultId": "e6be6579-af08-4447-b084-b68debc496a1","StartTime": 0.0699}]}},
```
- 한글도 지원
```json
    {"jobName": "TestKor","accountId": "002568105033","results": {"transcripts": [{ "transcript": "안녕하세요 만나서 반갑습니다" }],"items": [{"start_time": "0.04","end_time": "0.89","alternatives": [{ "confidence": "1.0", "content": "안녕하세요" }],"type": "pronunciation"},{"start_time": "0.89","end_time": "1.53","alternatives": [{ "confidence": "1.0", "content": "만나서" }],"type": "pronunciation"},{"start_time": "1.53","end_time": "2.65","alternatives": [{ "confidence": "1.0", "content": "반갑습니다" }],"type": "pronunciation"}]},"status": "COMPLETED"}
```
### 3-2. Amazon Translate

> 딥 러닝을 기법을 이용한 번역 솔루션

[링크](https://ap-northeast-2.console.aws.amazon.com/translate/home?region=ap-northeast-2#translation)

### 특징

- 딥 러닝 기법을 사용하여 자연스러운 번역을 추구함.
- Amazon Translate에서 처리되는 모든 컨텐츠는 암호화됨.
- `json` 형식으로 요청 및 응답함.

**요청**
```json
    {"Text": "Coronaviruses are a group of related viruses that cause diseases in mammals and birds. In humans, coronaviruses cause respiratory tract infections that can be mild, such as some cases of the common cold (among other possible causes, predominantly rhinoviruses), and others that can be lethal, such as SARS, MERS, and COVID-19. Symptoms in other species vary: in chickens, they cause an upper respiratory tract disease, while in cows and pigs they cause diarrhea. There are yet to be vaccines or antiviral drugs to prevent or treat human coronavirus infections.","SourceLanguageCode": "en","TargetLanguageCode": "ko"}
```
**응답**
```json
    {"TranslatedText": "코로나 바이러스는 포유류와 조류에 질병을 일으키는 관련 바이러스 그룹입니다. 인간에서 코로나 바이러스는 (다른 가능한 원인, 주로 라이노 바이러스 중) 감기의 일부 경우와 같이 경미하게 될 수있는 호흡기 감염을 일으키고 사스, MERS 및 COVIDD-19와 같이 치명적일 수 있습니다. 다른 종의 증상은 다양합니다. 닭에서는 상부 호흡기 질환을 일으키고 소와 돼지에서는 설사를 일으 킵니다. 인간 코로나 바이러스 감염을 예방하거나 치료하기 위해 백신이나 항 바이러스제가 아직 없습니다.","SourceLanguageCode": "en","TargetLanguageCode": "ko"}
```
[실시간 번역 예제 링크](https://ap-northeast-2.console.aws.amazon.com/translate/home?region=ap-northeast-2#translation)

### 3-3. Amazon Polly

> 딥 러닝을 사용하여 텍스트를 음성으로 전환

- 실시간 스트리밍 제공
- 자연스러운 음성
- 음성 저장 및 재배포가 자유러움

[링크](https://aws.amazon.com/ko/polly/)

## 3. 각 솔루션별 요금 정책

- Amazon Transcribe
    - 월별 변환된 오디오 시간(초)을 기준으로 사용한 만큼 비용을 지불
    - 초당 0.0004 USD의 비율로 월별 청구
    - 사용량은 1초 단위
    - 요청당 15초의 최소 요금이 청구됨
    - 24시간 거의 풀로 사용하면 5만원 정도 부과되는걸로 예상
    - 월 사용량이 1백만분(하루종일 풀로 사용하는경우) 별도 문의 필요

[요금 정책 링크](https://aws.amazon.com/ko/transcribe/pricing/?nc=sn&loc=3)

- Amazon Translate
    - 처리한 텍스트 수를 기준으로 사용한만큼 지불함
    - 문자 1백만개에 15$

[요금 정책 링크](https://aws.amazon.com/ko/translate/pricing/)

- Amazon Polly
    - 변환되는 텍스트 문자 수에 따라 비용이 부과됨.
    - 신경명 TTS와 표준 TTS 비용이 다르게 부과됨(신경망이 더 비쌈)
    - 문자 1백만개 (23시간 8분)

[요금 정책 링크](https://aws.amazon.com/ko/polly/pricing/?nc=sn&loc=4)

## 4. 결론

- 각각의 솔루션은 나쁘지 않다고 판단됨.
- 허나, 번역을 위해서는 각각의 솔루션을 거치면서 json 파일을 전달함
- 이는 결과적으로 지연을 발생시킴.
- 3가지 솔루션을 유기적으로 묶어 지연을 감소시킬 수 있는 방법(양방향 스트리밍이라던지..)이 필요함.

## 5. 기타

[hyperconnect-Azar](https://hyperconnect.com/tech/rtc/) : 실시간 음성 -> 텍스트 번역 제공