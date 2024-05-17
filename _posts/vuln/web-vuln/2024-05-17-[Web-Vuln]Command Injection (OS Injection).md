---
layout: single
title: Command Injection (OS Injection)
categories: Web-vuln
tag: [OS Injection, Command Injection, web, web vulnerability, injection]
toc: true
author_profile: false
---
# Command Injection이란?

> Command Injection이란 사용자의 입력값을 파라미터로 받아 서버측 백엔드에서 운영체제 명령으로 전달되어 명령을 실행하고 다시 반환될 때 발생하는 취약점으로 해당 취약점이 조재할 경 우 대상 서버를 완전히 제어 및 무력화 할 수 있는 치명적인 취약점이다.

# 공격 표면 

command injection의 경우 일반적으로 공격표면 즉 해당 취약점이 발생하는 구간을 찾기가 매우 어렵다.  예를 들어 아래와 같은 로직이 있다 생각해보자.

```
~/ViewPage/ViewText?textfilename=Test.txt

----------------------
exec('cat'+textfilename) 혹은 system('cat'+textfilename)
```

위와 같이 평범하게 filename 을 입력받아 출력 해주는 로직은 파일을 웹 서버단에 저장 후 일반적으로 출력해줄 수 있지만 위와 같이 시스템 명령어를 사용해 출력해주는 경우도 있다. 이와 같이 테스팅 과정에서 육안으로는 OS Injection 식별이 힘들다.

