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
: -> %3A
/ -> %2F
? -> %3F
# -> %23
[ -> %5B
] -> %5D
@ -> %40
! -> %21
$ -> %24
& -> %26
' -> %27
( -> %28
) -> %29
* -> %2A
+ -> %2B
, -> %2C
; -> %3B
= -> %3D
% -> %25
(공백) -> %20 OR +

</div>