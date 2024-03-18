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
![그림1-1](/assets/image/log_poisoning/image.pngimage.png)
해당 access.log 파일에는 사용자의 User-Agent 값이 담기개 된다.
이 떄 악의 적인 사용자가 User-Agent 에 악성 코드를 삽입할 수 있다.

만약 해당 타겟시스템의 서버측 언어가 PHP 로 동작할 경우
```<?php system($_GET['cmd']);?> ```와 같이 php의 시스템 명령이 가능한 코드를 삽입하게 되면, 서버에서 User-Agent 헤더를 해석하는 과정에서 php 코드로 인식하여 이를 실행하게 된다.
즉 ***<span style="color:red"> RCE 취약점 까지 이어질 수 있는 치명적인 취약점이라고 볼 수 있따.. </span>*** 