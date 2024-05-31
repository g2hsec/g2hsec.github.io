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