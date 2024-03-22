---
layout: single
title: (Web-Vuln)Open redirect (redirect for PHP header() vulnerability)
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

해당 능에서는 시스템 함수를 사용한다는 것을 짐작할 수 있다.
이 
때 적절한 입력 값 검증이 이루어지지 않는다면, 시스템의 메타문자들을 이용하여 앞 명령어에 이어서 추가 명령어를 실행할 경우 RCE 취약점으로 연계가 가능하다.