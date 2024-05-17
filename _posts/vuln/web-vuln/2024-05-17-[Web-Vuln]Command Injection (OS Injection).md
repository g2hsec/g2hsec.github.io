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

## OS 명령어의 인자 값으로 넘겨줄 때 자주 사용되는 파라미터

{% capture notice-2 %}  <!--notice-2 라는 변수에 다음 텍스트 문단을 문자열로 저장한다.-->  
#### 자주 사용되는 파라미터

* ?cmd={payload}
* ?exec={payload}
* ?command={payload}
* ?execute{payload}
* ?ping={payload}
* ?query={payload}
* ?jump={payload}
* ?code={payload}
* ?reg={payload}
* ?do={payload}
* ?func={payload}
* ?arg={payload}
* ?option={payload}
* ?load={payload}
* ?process={payload}
* ?step={payload}
* ?read={payload}
* ?function={payload}
* ?req={payload}
* ?feature={payload}
* ?exe={payload}
* ?module={payload}
* ?payload={payload}
* ?run={payload}
* ?print={payload}  <!--캡처 끝! 여기까지의 텍스트를 변수에 저장-->

<div class="notice">
  {{ notice-2 | markdownify }} <!--div 태그 사이에 notice-2 객체를 출력하되 markdownify 한다. 즉 마크다운 화-->
</div>

위와 같은 파라미터를 통해 사용자의 입력값을 받는다면 이부분을 놓치지말고 한번은 테스팅을 해볼 필요가 있다.

```
?textfilename=test.txt;id;ls;whoami
```

위와 같이 리눅스 계열이라면 ; 명령어를 통해 추가 결과 값이 반환되는지에 대한 유무를 판단할 수 있다.  OS Injection에서는 메타문자를 사용하여, 원하고자하는 시스템 명령을 실행시킬 수 있다.

| 메타문자            | 기능                                                             | 사용 가능 OS     |
|---------------------|----------------------------------------------------------------|------------------|
| `cmd1 \| cmd2`      | 왼쪽 명령의 결과를 오른쪽 명령의 입력으로 전달                    | *                |
| `cmd1 && cmd2`      | 왼쪽 명령을 성공적으로 실행하면 오른쪽 명령을 실행                | *                |
| `cmd1 || cmd2`      | 왼쪽 명령의 실행에 실패하면 오른쪽 명령을 실행                    | *                |
| `cmd1 ; cmd2`       | 한 줄에 여러 명령을 사용할 때 각각의 명령을 구분하기 위해 사용       | 리눅스/유닉스    |
| `` `cmd` , "cmd" `` | 백틱(`) 사이의 명령을 실행한 결과를 반환                           | 리눅스/유닉스    |
| `$(cmd)`            | $() 안의 명령을 실행한 결과를 반환                                 | 리눅스/유닉스    |
| `cmd > /your_file`  | 왼쪽 명령의 결과를 오른쪽 파일에 기록                              | 리눅스/유닉스    |
| `/your_file < cmd`  | 오른쪽 명령의 결과를 왼쪽 파일에 기록                              | 리눅스/유닉스    |
| `\n, 0x0a (줄바꿈)` | 줄바꿈                                                            | 리눅스/유닉스    |
| `cmd1; cmd2`        | 여러 명령어를 동시에 사용                                          | 리눅스/유닉스    |

# Blind OS Command Injection

**<u style=color:"red">보통 OS Injeciton의 경우 요청에 대한 응답값이 HTTP 응답 내에서 반환되지 않는 경우도 다수 존재한다. 이럴 경우 테스트 과정에서 놓치고 지나가는 경우가 있다.</u>**
<br>
그렇기에 Blind Command Injection 을 통해서 취약점 유무를 판별해야한다.

1. 시간지연
2. 출력 리다이렉트
3. 대역 외(out-of-band) 채널
