---
layout: single
title: Log Injection
categories: Web-vuln
tag: [Log Injection, log, web, web vulnerability, injection]
toc: true
author_profile: false
---
# 해당 취약점은 무엇인가?

> 사용자의 입력값으 로그파일에 기록될 때 발생 할 수 있는 취약점으로 공격자가 로그 기록을 조작하여 데이터 탈취 및 서비스중단까지도 발생할 수 있다. 

## 일어날 수 있는 피해는?

- 로그 항목 위조 : 임의로 삽입된 입력값이 로그파일에 삽입되어, 로그항목을 위조한다 공격자가 해당 시스템에 침투 후 보안 침해를 숨기거나 피해랄 악화시킬 수 있다. 이 경우 로그파일에 대한 신뢰를 떯어 뜨리고 보안 사고 조사를 어렵게 만든다.
- 로그파일 손상 : 로그파일에 기록되는 입력값에 매우 많은양의 데이터를 삽입시켜 로그파일을 손상시킬 수 있다.
- 정보 탈취 : 로그파일에 저장된 민감 정보를 탈취할 수 있다.

## 어떻게 발생하는가?

> 사용자의 입력값이 로그파일에 추가되는 로직이 구현되어 있을 경우 일어날 수 있다.

1. 신뢰할 수 없는 소스에서 데이터가 응용 프로그램에 입력된다.
2. 응용 프로그램 또는 시스템 로그 파일에 데이터가 작성된다.

```javascript
String val = request.getParameter("val");
	try {
  		int value = Integer.parseInt(val);
	}
	catch (NumberFormatException nfe) {
  		log.info("Failed to parse val = " + val);
	}
```

- 입력값이 정수값이 아닌 경우 해당 문제점을 나타내는 오류코드와 입력값이 함께 로깅되는 코드가 존재한다.
- 정수값이 아닌 “test” 라는 문자열을 입력할 경우 아래와 같이 출력 된다.


```
INFO: Failed to parse val
INFO: test
```

- 공격자가 입력값을 test%0d%0a%0d%0aINFO: success login 을 입력할 경우 아래와같은 로그가 생성되게 된다.

```
INFO: Failed to parse val
INFO: test

INFO: success login
```

> 이때 공격은 CRLF(\r\n) 문자를 이용하여 이루어진다.

- 로그 인젝션에 취약한 서버측 코드는 아래 사이트에서 확인이 가능하다.<br>
https://www.notion.so/Log-Injection-b7b4ee265f62490d889e72bb7dd58464?pvs=4#f0e218f80168433c85ba414f862042dc

# 대응책

- 입력 필터링 - 사용자 입력값을 로그파일에 기록 전 악성코드 및 데이터를 필터링하여 사전에 차단한다.
- 코드 검토 - 로그를 기록하는 코드에 대한 오류를 검사한다.
- 보안 모니터링 - 로그 파일과 시스템을 모니터링하여 의심스러운 활동이 있는지 확인한다

# References
https://vulncat.fortify.com/ko/detail?id=desc.dataflow.abap.log_forging#Ruby