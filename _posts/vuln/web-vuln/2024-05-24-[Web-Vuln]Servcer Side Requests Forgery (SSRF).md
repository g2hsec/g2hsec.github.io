---
layout: single
title: Servcer Side Requests Forgery (SSRF)
categories: Web-vuln
tag: [Servcer Side Requests Forgery, SSRF, web, web vulnerability, injection]
toc: true
author_profile: false
---

# SSRF 취약점이란?

> SSRF는 클라이언트측의 입력값을 위조시켜 위조된 HTTP 요청을 보내, 일반적으로 외부에서 접근이 불가능한 **<u style="color:red;">서버 내부망에 접근(Access)</u>** 하여, 데이터 유출 및 서버의 기밀성, 가용성, 무결성을 파괴한다. SSRF 의 경우 CSRF 공격과 유사하지만, 공격이 이루어지는 부분이 클라이언트측이냐 서버측이냐의 차이점이 존재한다. 

💡 **<u>SSRF 공격이 성공적으로 Exploit 될 경우 추가 공격으로 이루어지므로, 피해가 크다.</u>** 
{: .notice--primary} 

# 어떻게 발생하는가?
모든 클라이언트측의 요청에 응답을 한 서버에서 다루기에는 무리가 있다. 그러므로 각 서버끼리의 API를 통해 서버끼리 내부적으로 통신을 통해 요청과 응답이 이루어진다. 이때 접근 가능한 내부 IP를 통해 이루어지므로 SSRF 의 경우 루프백(Loopback) 주소인 127.0.0.1 을 사용하여 공격이 주로 이루어진다.

# 공격 표면
주로 URL을 통해 HTTP 요청과 응답에 URL에 관련된 정보가 있을 경우 SSRF 의 잠재적 공격 표면이 될 수 있으며, 특정 웹 사이트의 정보를 불러들어 오는 부분을 유심히 봐야한다.
하지만 외에도 다른 서비스로의 요청을 하는 기능에서도 많이 발견되고 있다.

> SSRF 의 취약점 판별은 OAST를 통한 out-of-band 기술을 통해 쉽게 파악할 수 있다. OAST를 통해 외부로의 접점을 확인할 수 있으며, 외부로의 통신이 막혀 있다면, loaclhost, 사설 IP등을 호출하여 접근 가능 여부를 파악할 수 있다.

특히나 SSRF 공격의 경우, DMZ 뒤쪽에서 동작하기에, ACL 등 보안정책을 우회할 수 있다.
<u sytle="color:red;">보통 SSRF의 경우 서버단에서는 허용된 IP에 대해서만 화이트 리스트 기반을 통해서만 특정 서비스로의 요청이 실행된다.</u>

# 공격 유형
SSRF 를 활용한 공격 방식은 크게 3가지로 분류할 수 있다.
## Port Scan 및 내부 시스템 파일 탈취

<div class="notice">
    SSH 서비스 포트를 스캔할 시 file?=http//[IP]:22  구문 사용<br><br>

    [리눅스/유닉스]
<br>
    file?=http://file:///etc/passwd
<br>
    -- 허용하는 URL 스키마를 사용하여 접근 --
<br>
  <h4 style="color:red;">SSRF 의 경우 프로토콜에 대한 제한을 받지 않기때문에 FTP, SMTP, … 등 과 URI 스킴이 상용 가능하다.</h4>
</div>

