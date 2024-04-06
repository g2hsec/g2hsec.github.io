---
layout: single
title: (bWAPP)Reflected (PHP_SELF)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

![그림 1-1](/assets/image/bwapp/xss/XSS%20-%20Reflected%20(PHP_SELF)-archive/image.png)
- First name과 Last name 값에 사용자의 입력값을 받고있다.


![그림 1-2](/assets/image/bwapp/xss/XSS%20-%20Reflected%20(PHP_SELF)-archive/image-1.png)
- 두개의 사용자의 입력값을 합쳐 페이지 내에 출력되고 있다.

```php
    <?php

    if(isset($_GET["form"]) && isset($_GET["firstname"]) && isset($_GET["lastname"]))
    {   

        $firstname = $_GET["firstname"];
        $lastname = $_GET["lastname"];    

        if($firstname == "" or $lastname == "")
        {

            echo "<font color=\"red\">Please enter both fields...</font>";       

        }

        else            
        { 

            echo "Welcome " . xss($firstname) . " " . xss($lastname);   

        }

    }

    ?>
```

- 위 로직을 보게되면 아무런 필터링 없이 입력값으 고스란히 출력하고 있으며 이로 인해 악성 스크립트가 실행될 수 있다.

```
 <script> alert(1)</script>
```

![그림 1-3](/assets/image/bwapp/xss/XSS%20-%20Reflected%20(PHP_SELF)-archive/image-2.png)
- alert가 실행된 걸 볼 수 있다.
