---
layout: post
title:  "Gzip Compression"
date:   2018-02-24
---

<p class="intro"><span class="dropcap">C</span>ompression is a simple, effective way to save bandwidth and speed up your site.</p>

## gzip 압축이란?

gzip 압축은 웹서비스 최적화 방법 중 하나입니다.
파일 압축에 쓰이는 gzip을 사용하여 서버에서 클라이언트로 보내는 데이터를 줄이는 방법입니다.

## 배경

<figure>
	<img src="{{ '/assets/img/2018-02-24-gzip compression/HTTP req res.PNG' }}" alt="">
	<figcaption>Fig1. - HTTP Request & Response</figcaption>
</figure>
위 그림은 일반적인 HTTP Request & Response입니다.
브라우저는 GET으로 서버에게 요청을 보내고 서버는 해당 파일을 찾아 브라우저로 전송합니다.

<figure>
	<img src="{{ '/assets/img/2018-02-24-gzip compression/http compressed.PNG' }}" alt="">
	<figcaption>Fig2. - HTTP Request & Response with Gzip</figcaption>
</figure>
이 그림은 gzip 압축을 사용한 HTTP Request & Response입니다.
브라우저가 서버로 요청할 때 gzip을 지원한다고 보내면, 서버는 해당 파일을 압축하여 브라우저로 전송합니다.

100KB의 데이터를 압축하여 10KB로 전송한다면, 네트워크 bandwith와 다운로드 시간 등을 줄여 페이지 로딩 속도를 향상시킬 수 있습니다.

## gzip 압축 적용하기

gzip 압축을 적용하려면 브라우저와 서버 모두 gzip을 지원해야 합니다.

배포한 웹사이트에서 개발자 도구를 사용하여 request header와 response header를 볼 수 있습니다.

<figure>
	<img src="{{ '/assets/img/2018-02-24-gzip compression/request header gzip.PNG' }}" alt="">
	<figcaption>Fig3. - 브라우저 Request header</figcaption>
</figure>
브라우저가 지원하는지 확인하려면, request header를 살펴보면 됩니다.

<figure>
	<img src="{{ '/assets/img/2018-02-24-gzip compression/response header before.PNG' }}" alt="">
	<figcaption>Fig4. - 서버 Response header</figcaption>
</figure>
서버에서도 gzip으로 압축하여 전송해야 하는데, 기본적으로 encoding이 적용돼있지 않습니다.

<figure>
	<img src="{{ '/assets/img/2018-02-24-gzip compression/파일열기.PNG' }}" alt="">
	<figcaption>Fig5. - httpd.conf file</figcaption>
</figure>
먼저 real 서버에 접속하여 Apache httpd.conf 파일을 수정해야 합니다.

httpd.conf 파일을 vi로 열고 아래 내용들이 주석 처리가 되어있다면 해제합니다.

* LoadModule deflate\_module modules/mod\_deflate\.so
* LoadModule headers\_module modules/mod\_headers\.so
* LoadModule filter\_module modules/mod\_filter\.so

(vi에서 검색은 /로 하면 됩니다.)

<figure>
	<img src="{{ '/assets/img/2018-02-24-gzip compression/복붙내용.PNG' }}" alt="">
	<figcaption>Fig6. - httpd.conf 내부</figcaption>
</figure>
다음으로 아래 내용을 적당한 곳에 붙여 넣습니다.

```
<IfModule mod_deflate.c>

        AddOutputFilterByType DEFLATE text/plain
        AddOutputFilterByType DEFLATE text/html
        AddOutputFilterByType DEFLATE text/xml
        AddOutputFilterByType DEFLATE text/css
        AddOutputFilterByType DEFLATE application/xml
        AddOutputFilterByType DEFLATE application/xhtml+xml
        AddOutputFilterByType DEFLATE application/rss+xml
        AddOutputFilterByType DEFLATE application/javascript
        AddOutputFilterByType DEFLATE application/x-javascript

        DeflateCompressionLevel 9

        BrowserMatch ^Mozilla/4 gzip-only-text/html  # Netscape 4.xx에는 HTML만 압축해서 보냄
        BrowserMatch ^Mozilla/4\.0[678] no-gzip  # Netscape 4.06~4.08에는 압축해서 보내지 않음
        BrowserMatch \bMSIE !no-gzip !gzip-only-text/html  # 자신을 Mozilla로 알리는 MSIE에는 그대로 압축해서 보냄
```

DEFLATE는 zip, gzip에서 사용되는 무솔실 압축 데이터 포맷이자 알고리즘입니다.
DeflateCompressionLevel는 압축 단계로 1~9 사이의 값을 지정할 수 있습니다.
물론 높을수록 더 많이 압축하지만 CPU에 부담을 줍니다.

<figure>
	<img src="{{ '/assets/img/2018-02-24-gzip compression/아파치 재시작.PNG' }}" alt="">
	<figcaption>Fig7. - 아파치 재시작 명령어</figcaption>
</figure>
httpd.conf 파일을 수정한 후 아파치를 재시작하면 끝입니다.

<figure>
	<img src="{{ '/assets/img/2018-02-24-gzip compression/after gzip.PNG' }}" alt="">
	<figcaption>Fig8. - Gzip 적용 후 서버 Response header</figcaption>
</figure>
다시 웹페이지로 돌아가 서버의 response header를 살펴보면 gzip을 지원하는 모습을 볼 수 있습니다.

## 결과

gzip 압축을 적용하기 전후를 비교해보겠습니다.
<figure>
	<img src="{{ '/assets/img/2018-02-24-gzip compression/before gzip.png' }}" alt="">
	<figcaption>Fig9. - Before Gzip website</figcaption>
</figure>
(gzip 압축 적용 전)
<figure>
	<img src="{{ '/assets/img/2018-02-24-gzip compression/http after gzip.png' }}" alt="">
	<figcaption>Fig10. - After Gzip website</figcaption>
</figure>
(gzip 압축 적용 후)

gzip 압축 전후를 비교하면,
276KB - Load 2.40s <=> 159KB - Load 1.03s로
117KB와 1.37s의 로딩 시간 감소를 확인할 수 있습니다.

## 은총알은 없다

하지만 gzip 압축이 무조건 좋은 것은 아닙니다.
gzip의 경우 서버에서 파일을 압축하는 시간과 브라우저에서 파일을 압축 해제하는 시간이 필요합니다.
<figure>
	<img src="{{ '/assets/img/2018-02-24-gzip compression/but.PNG' }}" alt="">
	<figcaption>Fig11. - 은총알은 없다</figcaption>
</figure>
때문에 위처럼 2.2KB < 6.0KB임에도 시간은 gzip을 사용한 경우가 더 오래 걸리는 것을 볼 수 있습니다.
gzip 압축을 해제할 때 서버와 브라우저에 약간의 CPU가 쓰이기 때문입니다.
그래서 통상적으로 2KB 이하의 파일은 압축하지 않는다고 합니다.

이런 문제를 보완하기 위해 미리 gzip으로 압축하여 사용하기도 합니다.
거의 변하지 않는 파일들을 .gz으로 압축 후 서버에 올리고

```
<script type="text/javascript" src="./어떤파일.js.gz"></script>
```

처럼 포함시키면 서버에서 압축하는 과정을 줄일 수 있습니다.

## 참조

[Apache Module mod_deflate](https://httpd.apache.org/docs/2.4/en/mod/mod_deflate.html)
[How To Optimize Your Site With GZIP Compression](https://betterexplained.com/articles/how-to-optimize-your-site-with-gzip-compression/)
[Apache에서 gzip을 이용한 압축전송하기](http://www.playnexacro.com/index.html#show:article)