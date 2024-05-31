---
layout: single
title: Encoding Obfuscation
categories: Web-vuln
tag: [Encoding Obfuscation, Encoding, web, web vulnerability]
toc: true
author_profile: false
---

💡 **<u>해당 다양한 인코딩을 통한 난독화 기법은 portswigger을 상당부분 참고하였습니다.</u>** 
{: .notice--primary} 

> 다양한 취약점을 식별하여 해당 취약점에 대한 Exploit을 진행 할 때 Payload에서 문자열 혹은 dot(.), / 등이 필터링 되어 있는 경우고 종종 다수 존재한다. 이 때 이를 우회할 수 있도록 인코딩을 통해 난독화를 이용할 수 있다.

클라이언트와 서버는 다양한 인코딩 방식을 사용하여 서로 시스템간의 데이터를 전송한다. 전달받은 데이터는 디코딩 되어 백엔드단에서 처리되된다.
<br><br>
일반적으로 Query 매개변수는 일반적으로 서버측에서 URL 디코딩 되며,
<br>
HTML 요소의 텍스트 콘텐츠는 클라이언트측에서 HTML 디코딩 된다.

# URL 인코딩을 통한 난독화

Percent-encoding(퍼센트 인코딩) 이라고도 불리며, URL을 통해 GET형식으로 전달할 때 수행되는 인코딩 방식이다.<br>
URL 인코딩의 경우 대체 문자의 ASCII값에 대한 16진수 표현을 '%' 기호와 함께 사용된다.
자동으로 인코딩 되는 특수 문자는 아래 표와 같다

<div class='notice'>
: -> %3A<br>
/ -> %2F<br>
? -> %3F<br>
# -> %23<br>
[ -> %5B<br>
] -> %5D<br>
@ -> %40<br>
! -> %21<br>
$ -> %24<br>
& -> %26<br>
' -> %27<br>
( -> %28<br>
) -> %29<br>
* -> %2A<br>
+ -> %2B<br>
, -> %2C<br>
; -> %3B<br>
= -> %3D<br>
% -> %25<br>
(공백) -> %20 OR +

</div>
이 외에 특수 문자 혹은 일반 문자들도 필요하지는 않지만, 인코딩은 가능하다.
<hr>
모든 URL 기반으로 Query 매개변수를 통해 관련 변수에 할당되기 전 서버측에서는 자동으로 URL 디코딩을 수행하게 된다.
<br>
간혹 WAF 와 같은 장비가 사용자의 입력값을 URL 디코딩을 수행하지 않는 경우가 존재한다. 이럴 경우 블랙리스트 기반 필터 정책을 URL 인코딩으로 인코딩하여 전송하게되면, 필터링을 우회할 수 있다.

<div class='notice'>
EX) SELECT -> %53%45%4C%45%43%54
</div>

# Double URL Encoding 을 통한 난독화

일부 몇몇 서버는 수신받은 URL을 확인 후 두 번의 URL 디코딩을 수행한다. 

<div class='notice'>
../../etc/passwd -> %252E%252E/%252E%252E/etc/passwd
</div>

모든 보안 메커니즘이 입력을 확인 할 때 입력값을 Double Decoding 하지는 않는다. 그러므로 

<div class='notice'>
/?payload=%3Cimg%20src%3Dx%20onerror%3Dalert(1)%3E
(<img src=x onerror=alert(1)>)
</div>

위와같이 단일 인코딩을 통해 XSS PoC 를 주입할 경우 WAF 단에서
차단되어 백엔드까지 해당 payload가 전달되지 않는다.
하지만 Double Encoding를 통해 PoC를 작성하여 보내게 된다면?

<div class='notice'>
/?payload=%253Cimg%2520src%253Dx%2520onerror%253Dalert(1)%253E
</div>

단일 디코딩을 수행 하더라도 아직 인코딩된 상태로 남아있기에
WAF는 해당 payload를  제대로된 필터링을 식별하지 못하고 백엔드 서버측으로 전달되고 이후에 백엔드 측에서는 Double Decoding를 통해 payload가 성공적으로 주입되게 된다.

# HTML 인코딩 을 통한 난독화

HTML 문서에서 브라우저가 마크업의 일부로 잘못 해석하지 않도록 특정 문자를 이스케이프 처리하거나 인코딩 되어 표현되어야 한다.
요소의 텍스트 내용 혹은 속성 값과 같은 HTML 내의 특정 위치에서
브라우저는 문서를 구문 분석할 떄 자동으로 디코딩작업을 수행한다.
이를 활용하면 클라이언트 측 공격에 대한 페이로드를 난독화 하여 서버 측 검증을 우회할 수 있다
<br><br>
예를들어 < 표시는 아래와 같이 표현이 가능하다.

<div class='notice'>
< -> &#X61;
</div>

이와 같이 서버측에서 alert() 라는 Payload를 명시적으로 필터링 하고 있다면 이를 HTML 인코딩 하여 아래와 같이 Payload를 구성할 수 있다.

<div class='notice' markdown="1">
\<img src='' onerror="&#38;#61;lert(1)"\>
</div>

위와 같이 Payload를 작성해서 요청을 보낼 경우 서버측 필터링 로직을 우회하고 브라우저가 페이지를 렌더링 할 때 사입된 Payload를 디코딩하고 실행하게된다.

> 이러한 HMTL 인코딩은 10진수 혹은 16진수 코드 포인트를 사용하여 참조를 제공한다.

<div class='notice' markdown="1">
&#38;#58; === &#38;#x3a; === '<'
</div>

여기서 신기한 점은 HTML인코딩을 사용할 때 코드 포인트에 숫자 0 을 임의 개수로 포함할 수 있는데 이렇게 0을 포함하여 WAF및 기타 필터링을 우회할 수 있다.  

<div class='notice' markdown="1">
\<a href="javascript&#00000000000058;alert(1)"\>Check here\</a\>
</div>

# XML 인코딩 을 통한 난독화

XML 구문에서도 HTML인코딩과 유사하게 숫자 이스케이프 시퀀스를 사용하여 인코딩하게된다. 이 때 XML 입력을 통해 데이터를 전달하는 로직이 존재한다면 여러 취약점을 우회할 수 있다.

> 특징은 XML인코딩을 통한 우회는 HTML과 달리 브라우저에 의한 클라이언트측에서 디코딩 되는것이 아니라 서버 자체에 의해 디코딩되어 WAF및 기타 필터를 우회할 수 있다.


<div class='notice' markdown="1">
\<test\><br>
&emsp;\<list\><br>
&emsp;&emsp;1<br>
&emsp;\</list\><br>
&emsp;\<mode\><br>
&emsp;&emsp;100 &#53;ELECT * FROM information_schema.tables<br>
&emsp;\</mode\><br>
\</test\>
</div>

위와 같은 XML 요청 코드가 존재할 때 mode부분에서 SQL 인젝션을 주입하게된다. 이 때 select 문자열 필터링을 우회하기위해 인코딩을 사용할 수 있다.

# Unicode 를 통한 난독화

유니코드의 이스케이프 시퀀스 문자에 대한 4자리로 구성된 16진수 코드의 접두사로 구성되어 있다 

<div class='notice' markdown="1">
\u003a === U+003A === ':'
</div>