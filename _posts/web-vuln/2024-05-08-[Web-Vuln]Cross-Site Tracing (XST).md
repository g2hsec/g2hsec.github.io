---
layout: single
title: (Web-Vuln) Cross-site Tracing (XST)
categories: Web-vuln
tag: XST, XSS, web, web vulnerability, Cross-site Tracing]
toc: true
author_profile: false
---
# 해당 취약점은 무엇인가?
> XSS 공격의 종류중 하나로서, XSS 공격을 방어하거 위해 만들어진 쿠키 속성 중 하나인 HTTPOnly 속성을 우회할 수 있는 기법이다.

## HTTPOnly??
- HTTPOnly는 XSS 공격을 방지하기 위해 쿠키 값이 JavaScript를 통해 접근 할 수 없도록 한다.

# 어떻게 발생하는가?
- 요청 메서드중 하나인 TRACE 메서드가 허용되어 있을 경우 발생할 수 있다.

## TRACE를 통한 XST
 - TRACE 메서드의 경우 클라이언트가 서버에게 요청을 재전송하라는 의미로 사용되며, 주로 디버깅 용도로 사용되었다. 이는 클라이언트가 보낸 요청 메시지를 서버측에서 그대로 클라이언트 측으로 되돌려주게 되는데 이 때 요청값에 쿠키 세션값들이 존재하여, 세션탈취의 취험이 존재한다.

 ## 공격 앤드포인트 탐색
 XST 취약점 또한 XSS 의 한 종류로서 Stored Based XSS와 유사한 성격을 띄고 있다. XSS 공격에 취약한 앤드포인트가 존재할 때 악성 스크립트가 담긴 서버로의 AJAX 요청을 포함하여 서버 데이터베이스 내에 저장 한다. 이 때 희생자가 해당 악성스크립트(게시글로 가정)을 확인 할 경우 서버로의 TRACE를 통한 AJAX요청을 보내고 다시금 서버로 부터 응답을 받을 때 공격자가 의도한 스크립트가 실행되게 된다.

### 공격 대상 TRACE 사용 유/무 체크
대상 시스템에 TRACE 메서드가 실행중인지 확인할 필요가 있다.
아래 3가지 방법을 통해 확인할 수 있다.

```
1. nmap <대상 ip> --script http-methods --script-args http-method.test-all='/<대상ip>'

2. nikto -h <대상 ip>

3. curl -c -X -OPTIONS <대상 IP>
```

## 공격 POC Code

```javascirpt
<script type="text/javascript">
<!--
    function sendTrace () {
	var xmlHttp = new ActiveXObject("Microsoft.XMLHTTP");
	xmlHttp.open("TRACE", "http://foo.bar",false);
	xmlHttp.send();
	xmlDoc=xmlHttp.responseText;
	alert(xmlDoc);
    }
//-->
</script>
<INPUT TYPE=BUTTON OnClick="sendTrace();" VALUE="Send Trace Request">
```

위 코드는 주로 사용되는 XMLHTTP를 이용한 AJAX요청을 보내는 POC 코드이다.
<br>
POC코드에서는 alert()함수를 통해 개념증명을 보이지만, 이 부분을 document.location와 같이 공격자 서버로의 세션 전송 구문을 넣게되면 응답받은 값의 쿠키값을 공격자의 서버로 전송하게 될 것이다.

<div style="background-color:#ffffcc; padding:10px; border: 2px solid #ff6600; border-radius: 10px; text-align: center; color: black;">
TRACE메서드는 알반적으롣 ㅐ부분의 웹 서버에서 기본적으로 비활성화 되어있으며, 현재 사용되는 대부분의 웹 브라우조는 XHR(XMLHttpRequedst)에서 TRACE를 실행하지 못하도록 설정되어 있다. 이로 인해 AJAX를 이용한 XST공격은 사실상 대부분의 웹 브라우저에서는 불가능하만 일부 취약환경에서는 사용이 가능하다.
</div>
<br>

# 대응방안
1. TRACE 메서드를 사용하지 않아야 한다.

# Reference
- https://webhack.dynu.net/?idx=20161111.001&print=friendly