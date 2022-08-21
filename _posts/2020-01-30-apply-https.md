---
layout: single
title: "[Nginx][HTTPS] Nginx에 HTTPS 적용하기"
tag:
  - Ngix
  - HTTPS
---

드디어! 진행중인 토이프로젝트에 HTTPS를 적용하는데 성공하였다. 역시 삽질이 있었고 이 토이 프로젝트를 마무리 할 수 있게되었다. 이번 글은 HTTPS를 적용하는 과정을 기록하고자 한다.

## 개요

요 근래 구축된 웹사이트에 HTTPS를 적용하지 않은곳을 찾기가 어려울 정도로 HTTPS가 적용되어 있다. 나또한 HTTPS를 토이 프로젝트 웹 사이트의 적용 필요성을 느꼈다.

왜 토이 프로젝트에 HTTPS(TLS) 가 적용되어야하는가?

- 더이상 크롬의 보안 경고문을 보지 않아도 된다.
- HTTP/2 를 적용하여 성능상의 이점을 누릴 수 있다.

보안관련 사항은 이미 널리 알려져 있는 사항이고 나의 경우는 크롬 주소창에 자물쇠 모양이 표시 되지 않는게 매우 불편했다. 이 문제를 해결하기 위해 HTTPS를 적용하기로 하였다.

## 적용 과정

### 0. 사전 정보

토이 프로젝트의 환경은 아래와 같다.

- Ubuntu 18.04.3 LTS
- Nginx 1.14.0
- Mongdb 4.2.3
- NodeJS + Koa
- AWS EC2

### 1. Letsencrypt 설치

본래 HTTPS를 적용하기 위해 SSL 인증서를 유료로 구매하고 인증서 파일을 다운로드 받는등 번거롭고 귀찮은 절차가 있으나 `Letsencrypt` 를 통해 무료로 손쉽게 SSL 인증서 및 관련 작업을 할 수 있다.

참고로 `Letsencrypt`로 발급된 인증서는 아래의 제약을 가진다
이 블로그에 잘 정리되어있으니 참고해보자 [링크](https://kr.minibrary.com/353/#comment-4052078787)

제한사항을 간략히 요약하자면..

- 90일 제한 [why-90-days](https://letsencrypt.org/2015/11/09/why-90-days.html)
  - 90일이 되기전에 갱신을 해야함.
- Asterisk (\*.minibrary.com)을 사용할 수 없음
  - 서브 도메인을 만들어 SSL 인증서를 사용하려면, 해당 도메인을 위한 인증서를 발급받아야함.
- certonly(standalone)한정으로 진행하는 경우 인증서 발급 과정에서 80, 443 포트가 비어있어야함.
  - 갱신하는 과정에서 웹 서비스를 중단해야하는 경우 발상

letsencrypt 설치

> sudo apt install letsencrypt -y

letsencrypt 를 `certonly` 로 구동

> sudo letsencrypt certonly --standalone -d 발급할 도메인 주소

`certonly` 로 진행하는 이유는 나는 단순히 인증만 필요하여 `certonly`로 진행한 것이다. 만약 자동갱신이 필요하다면 [certbot](https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx)을 사용하여 자동갱신 기능을 쓰면 되겠다.

참고로 port 80 포트가 사용중이면 인증서 발급이 안되니 미리 확인하여 열어놓도록 하자.

우선 관리자 알림 기능을 사용하기위해 메일 주소를 입력한다. 이 메일을 통해 인증서 만료 알림 메일이 전송된다.

![apply-https_i1](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/dev_review_songi_stock/apply-https/apply-https_i1.png?raw=true)

사용 조건을 묻는다. 당연히 A를 선택한다.

![apply-https_i2](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/dev_review_songi_stock/apply-https/apply-https_i2.png?raw=true)

인증서 발급이 무사히 완료되었다면 아래와 같이 메시지가 출력된다.

![apply-https_i3](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/dev_review_songi_stock/apply-https/apply-https_i3.png?raw=true)

이것으로 `Letsencrypt` 설치가 완료되었다.

### 2. Nginx 설정

아래와 같이 설정한다. http2 설정도 함께 진행하였다. 참고로 나는 현재 reverse proxy를 사용중이라 `proxy_pass`를 통해 nginx 뒤의 nodeJS 서버에 요청을 전달한다.

```

server {
    listen 80;
    return 301 https://$host$request_uri;
}


server {
 listen 443 ssl http2;
 listen [::]:443 ssl http2;
 server_name songistock.net www.songistock.net;

 ssl_certificate /etc/letsencrypt/live/www.songistock.net/fullchain.pem;
 ssl_certificate_key /etc/letsencrypt/live/www.songistock.net/privkey.pem;

 ssl_protocols TLSv1.2;
 ssl_prefer_server_ciphers on;
 ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';

        location / {
         proxy_pass http://127.0.0.1:3000;

         proxy_set_header        Host $host;
         proxy_set_header        X-Real-IP $remote_addr;
         proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header        X-Forwarded-Proto $scheme;
         proxy_read_timeout      120;
         proxy_http_version 1.1;
         proxy_redirect off;
        }
}

```

### 3.Nginx 재기동 or 기동

상황에 맞게 Nginx를 재기동하거나 기동시킨다.

> service nginx restart

### 4. 잘 적용되었는지 확인

크롬에서 SSL, HTTP/2 둘다 확인가능하고 별도의 확인 사이트에서도 가능하다.

**SSL 적용 확인**

아래 사이트에서 확인하려는 URL를 입력한다.
https://www.sslshopper.com/ssl-checker.html

**HTTP/2 적용 확인**
아래 사이트에서 확인하려는 URL를 입력한다.
[https://tools.keycdn.com/http2-test](https://tools.keycdn.com/http2-test0)

크롬 개발자 도구에서 접속한 URL의 protocol이 h2(HTTP/2)로 표시되는지 확인한다.

![apply-https_i6](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/dev_review_songi_stock/apply-https/apply-https_i6.png?raw=true)

### 끝?

이렇게만 보면 정말 간단하고 쉽게 적용한것처럼 보인다. 실제로 nginx 설정까지해서 2시간으로 마무리 지을 수 있었다. 이후에 발생한 어처구니 없는 문제만 빼고..

Nginx를 재기동 시키고 나서 아래 오류 메시지가 잡혔다. 내용은 Upstream쪽에서 연결을 닫아버렸다는 내용인데.. 이걸 해결하기위해 장장 3일간 별별 삽질을 치루고 나서야 해결할 수 있었다.
![apply-https_i5](https://github.com/momoci99/momoci99.github.io/blob/master/assets/img/dev_review_songi_stock/apply-https/apply-https_i5.png?raw=true)

우선 결론은 AWS EC2 인스턴스가 한번 죽고나서 재기동되었는데 `mongodb` 서버가 재기동 되지않았던것이 원인이였다. DB가 죽어버렸으니 DB관련 로직이 모조리 동작 안하는게 당연했다. `mongodb` 서비스를 기동시키니 모든 문제가 해결되었다.

뭐 삽질 덕분에 Nginx 설정에 대해 자세히 알아볼 수 있었던 기회였고, _무의식적으로 XX는 괜찮을꺼야_ 라는 행위가 얼마나 위험한지 다시한번 깨달을 수 있었다. 무려 3일간 삽질한건 분명 치명적이였지만.

#### 참고한 링크들

링크중에 certboy이 nginx를 지원하지 않는다는 내용이 있는데 지금(2020년)은 잘 지원하니 참고하자.

- https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-with-ssl-as-a-reverse-proxy-for-jenkins

- https://kr.minibrary.com/353/#comment-4052078787

- https://gdtbgl93.tistory.com/97

- https://jootc.com/p/201901062488

- https://docs.nginx.com/nginx/admin-guide/security-controls/securing-http-traffic-upstream/

- https://www.howtoforge.com/how-to-enable-http-2-in-nginx/
