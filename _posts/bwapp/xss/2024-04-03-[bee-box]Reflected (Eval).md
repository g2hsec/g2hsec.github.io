---
layout: single
title: (bWAPP)XSS - Reflected (Eval)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/xss/Revlected%20(Eval)-archive/image.png)
- data() 함수를 통한 사용자가 해당 페이지 접근시 실시간 시간을 알려준다.
- 해당 시나리오의 경우 data 함수와 eval() 함수를 사용한당.
- eval함수는 사용하기에 따라 매우 취약하게 작동할 수 있다.

```
/bWAPP/xss_eval.php?date=Date()
```

- Request 방식은 GET 으로 전송되며 date 변수에 담겨 날라가게 된다.

```php
   <script>

    eval("document.write(<?php echo xss($_GET["date"])?>)");

    </script>
```

- 제공되는 소스코드를 보게되면 eval 함수를 사용하여 date 파라미터로 받은 입력값을
- echo 명령어를 사용하여 document.write 즉 페이지 상에 출력시킨다.
- 즉 이 때 입력값을 조작하면 스크립트 실행이 가능하다.

```
/bWAPP/xss_eval.php?date=alert(1)
```

![그림 1-2](/assets/image/bwapp/xss/Revlected%20(Eval)-archive/image-1.png)
- 성공적으로 alert창이 실행된다.

## Level - Medium & High

>medium level의 경우 해당 시나리오에서는 eval 함수를 사용한 스크립트 실행이므로 htmlspecialchars() 함수에서 필터링 하는 어떠한 문자도 필터링되지 않는다.

```php
<?php

    if(isset($_GET["date"]))
    {    

        if($_COOKIE["security_level"] == "2")
        {

            if($_GET["date"] != "Date()")
            { 

                echo "<p><font color=\"red\">Invalid input detected!</font></p>";        

            }

            else
            {

    ?>
```

- 위 소스코드는 High level의 필터링 소스코드이다.
- 해당 level 에서는 화이트리스트 기반 방식으로 Date() 를 제외한 어떠한 입력값도 허용하지 않고 있다.