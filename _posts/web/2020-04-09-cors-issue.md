---
title: "CORS Issue"
date: 2020-04-09
categories:
  - web
tags:
  - "#cors"
---

## CORS (Cross-Origin Resource Sharing) 교차 출처 리소스 공유

CORS란 서로 다른 Origin에서 자원에 접근할 수 있는 권한을 부여하도록 브라우저에 알려주는 체제라고 볼 수 있다.  

- Origin(출처) (도메인, 프로토콜, 포트)가 모두 같을 경우 Same-Origin, 하나라도 다를 경우 Cross-Origin이다.  
  
HTTP 요청에 대해서 HTML(HTML link태그, css, img태그 src에서의 리소스 접근)은 기본적으로 Cross-Origin요청이 가능한 정책을 따른다.     
그러나 script 태크 내에 있는 HTTP 요청 (XmlHttpRequest, Fetch API)에 대해서는 기본적으로 Same-Origin 정책을 따른다.  

대규모 웹 서비스가 늘어나며, 외부 호출에 대한 니즈가 점점 많아짐에 따라 W3C는 조금 더 안전하게 브라우저와 서버간에 교차 통신을 할 수 있도록 CORS라는 정책을 내놓았다.  

## CORS 동작 방식

CORS의 기본적인 동작은 서버가 한 origin으로부터 요청을 받게되면 응답할 때 HTTP 헤더에 특정 헤더를 추가함으로써 브라우저는 이 origin이 특정 리소스를  
읽을 수 있는 권한이 있는지 없는지를 알게 된다.  추가적으로 GET 이 외의 메소드나, POST 메소드에서 특정 MINE Type은 서버 데이터에 사이드 이펙트를  
발생시킬 수 있기 때문에 기본적으로 브라우저는 Preflight Request 방식으로 요청할 수 있도록 규제한다.

- Simple Requests

- 조건  

1. HTTP Method가 GET, POST, HEAD 이셋 중에 하나로 요청한 경우 Simple Request 방식으로 요청  
2. Fetch 표준 정책에서 정의한 Forbidden Header Name 이라는 헤더 목록(클라에서 자동으로 넣음)과 CORS-safelisted request header 라는 헤더 목록(클라에서 수동으로 넣을 수 있음)  
    이외에 다른 커스텀 헤더, 권한과 관련된 헤더가 없는 경우 Simple Request 방식으로 요청한다.  
3. HTTP Method가 POST인 경우 Content-Type 헤더를 수동으로 지정할 수 있는데 application/x-www-form-urlencoded, multipart/form-data, text/plain 이 세 가지 값에 포함되는 경우   
    Simple  Request 방식으로 요청한다.  

- 과정  

1. 브라우저는 Request가 Cross-Origin 요청인 것을 판단하여 요청에 "Origin: https://foo.example" 헤더를 추가하여 외부 서버로 보낸다.  
2. Cross-Origin으로부터 온 요청인 것을 안 서버는 Origin 헤더를 확인하며, 그 값이 허용이 되었는지 안되었는지를 확인한다.  
    그리고 "Access-Control-Allow-Origin: [서버에서 설정한 값]" 을 응답 헤더에 추가하여 클라이언트로 보낸다.  
3. 브라우저가 응답을 받으면, "Access-Control-Allow-Origin"헤더의 값을 찾아 해당 Origin의 값이 허용이 됐는지를 판단하고,  
    만약 허용이 되었다면 리소스에 접근할 수 있도록 하며, 그렇지 않다면 리소스에 접근할 수 없는 에러를 던진다.  

- Preflight Request

1.  브라우저는 Request가 Cross-Origin 요청인 것을 판단하여 아래와 같이 요청에 "Origin: https://foo.example" 헤더를 추가한다.  
    또한 브라우저는 POST 방식이며, Content-Type이 application/x-www-form-urlencoded, multipart/form-data, text/plain에 포함되지 않기 때문에  
    Prefight Request 방식으로 보내야 한다는 것을 알고 있다. 그래서 브라우저는 요청에 아래와 같이 헤더 정보를 추가하여 외부 서버로 보낸다.  

- Origin: https://foo.example  
- Access-Control-Request-Method: POST  
- Access-Control-Requset-Headers: X-PINGOTHER, Content-Type  

2. 서버는 Preflight Request 방식으로 날아온 요청에 대한 API에 아래와 같은 정보를 헤더에 추가해서 응답한다.  

- Access-Control-Allow-Origin: https://foo.example  
- Access-Control-Allow-Methods: POST, GET, OPTIONS  
- Access-Control-Allow-Headers: X-PINGOTHER, Content-Type  
- Access-Control-Max-Age: 86400  

3. 응답받은 헤더 정보를 통해서 브라우저는 실제 요청을 외부 서버로 보낼지 말지를 판단하게 된다.  
    위 예시에서 받은 응답은 해당 API는 Cross-Origin에 대해서 POST방식과 커스텀 헤더인 X-PINGOTHER 그리고 Content-Type을 허용한다고 하였으므로,  
    브라우저는 실제 요청을 외부 서버로 보낼 수 있다.

- Preflight Request 방식은 많은 리소스를 잡아 먹는다. 그렇기 때문에 서버에서 "Access-Control-Max-Age" 헤더 정보를 통해서 Preflight Request 를 캐싱함으로써 그 효율을 높힐 수 있다.  

- 참고로, 외부 서버로 요청할 때 withCredentials를 true로 세팅하여 보냈다면, 외부 서버로 부터의 응답 헤더에 Access-Control-Allow-Origin의 값에 WildCard가 오면 안 된다. 무조건 명시적으로 Origin의 URL정보가 있어야 한다.

* 출처: https://vvshinevv.tistory.com/60 [왜 모르는가?]


 

