---
layout: single
title: PHP Type Juggling Magic Hash
categories: Web-vuln
tag: [php, Type Juggling, Magic Hash, web Vulnability]
toc: true
author_profile: false
---

# Type Juggling 이란?
> PHP는 Python, Javascript와 같은 언어와 동일하게 동적 언어이다. 이는 프로그램이 실행되는 동안 변수 유형을 검사하게되는데, PHP에는 Type Juggling라는 기능이 존재한다. 다양한 유형의 변수를 비교하는 동안 PHP는 공통적이고 비교 가능한 유형으로 변환시킨다. 즉 정수 7과 문자열”7”을 비교할 때 7 == “7” 은 True를 반환하게 된다. 다른 경우로 “7 apple” == 7 이 또한 True를 반환하게된다. 이는 == 를 통한 비교를 수행하기 떄문에 발생하는 취약점이다.

- PHP 에서는 두 값을 비교하는 방식으로 == 와 === 이 존재한다.

![그림 1-1](/assets/image/vuln/web-vuln/PHP%20Type%20Juggling%20Magic%20Hash/image.png)

위 그림을 보게되면, == 방식을 사용하여 비교하는 경우  
<br>
매우 느슨하게 두 값을 비교하며, 다양한 경우에 True를 반환하게 된다.

![alt text](/assets/image/vuln/web-vuln/PHP%20Type%20Juggling%20Magic%20Hash/image-1.png)

위 그림의 ===을 사용한 경우를 보면 매우 엄격하게 유형과 데이터를 검사하는 걸 볼 수 있다.

<hr>
PHP 에서는 문자열과 정수를 비교할 때 문자열을 숫자로 변환한 다음 숫자 연산을 수행하게 된다.<br>
즉, 일반적인 문자열은 정수 0 으로 변환된다.<br>
그 후 문자열에서 숫자가 발견될 경우 해당 숫자의 정수로 변환된다.<br>

- **True** → “0000” == int(0)
- **True**→ “0e12: == int(0)
- **True**→ “1abc” == int(1)
- **True**→ “0abc”== int(0)
- **True**→ “abc” == int(0)
- **True**→ “356abc” == int(356)

```php
if ($_POST['secret'] == $passString) {login()}
```

위와 같은 검증 로직이 있다고 가정해보자, passString 라는 변수에 담겨져 있는 값과<br>
사용자가 입력한 값이 ==(Loose Comparisons)로 비교할 때 입력값으로 정수 0 을 입력하게되면,<br>
PHP 에서는 두 값을 비교하기 위해 문자열을 정수 0 으로 변환시켜 비교를 수행한다. 그럼 위 코드에서는 True를 반환하게되고, login() 함수를 호출하게 되는 것이다.<br>

```
이는 PHP 버전별로 반환값이 상이할 수 있다.
```

PHP에서는 Array(배열)은 null 값으로 반환되는데 == 연산자를 통해 null == 0 은 True를 반환한다.

# Reference
- https://owasp.org/www-pdf-archive/PHPMagicTricks-TypeJuggling.pdf