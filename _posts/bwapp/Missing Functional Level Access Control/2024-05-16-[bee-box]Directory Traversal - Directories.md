---
layout: single
title: (bWAPP)Directory Traversal - Directories
categories: bwapp
tag: [Directory Traversal,Path Traversal, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> Directory Traversal 혹은 path traversal은 ./(현재 디렉토리) 또는 ../(상위 디렉토리로 이동)를 나타내는 문자들이 제대로 필터링되지 않아 파일 시스템 API로 전달되어, 허용된 디렉토리가 아닌 숨겨진 디렉토리 및 파일에 대한 무단 access를 가능하게 하는 취약점이며, 보통 File 명과 같이 사용자의 입력값이 전달되어, 찾아오는 로직에서 많이 발견된다.

## Low-Level

![그림 1-1](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Directory%20Traversal%20-%20Directories/image.png)
- 여러 파일들이 존재한다.

```
/bWAPP/directory_traversal_2.php?directory=documents
```
- URL을 보게되면 directory 파라미터에 디렉터리 명을 Include 하는 것 처럼 보인다.

```
/bWAPP/directory_traversal_2.php?directory=../
```

- 위와 같이 ../라는 경로 이동 문자를 삽입하였다.

![그림 1-2](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Directory%20Traversal%20-%20Directories/image-1.png)
- 경로 이동문자가 정상적으로 인식되면서
- 상위 경로에 존재하는 디렉터리 및 파일들이 그대로 노출되고 있다.

![그림 1-3](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Directory%20Traversal%20-%20Directories/image-2.png)
- 존재하는 파일을 선택하면 해당 파일의 내용 또한 확인이 가능한 걸 볼 수 있다.

## Medium-Level 

```php
function directory_traversal_check_2($data)

{
    $directory_traversal_error = "";  
    if(strpos($data, "../") !== false ||

       strpos($data, "..\\") !== false ||

       strpos($data, "/..") !== false ||

       strpos($data, "\..") !== false ||

       strpos($data, ".") !== false)
    {



        $directory_traversal_error = "Directory Traversal detected!";

    

    }
    return $directory_traversal_error;
}
```

- Medium Level의 필터링 코드를 보게되면 strpos함수를 통해 경로이동에 사용되는 문자들을 필터링 하여 보안대책을 구현하였다.

## High-Level 

```php
function directory_traversal_check_3($user_path,$base_path = "")

{
    $directory_traversal_error = "";   
    $real_base_path = realpath($base_path);

    $real_user_path = realpath($user_path);
    if(strpos($real_user_path, $real_base_path) === false)

    {
        $directory_traversal_error = "<font color=\"red\">An error occurred, please try again.</font>";
    }
    return $directory_traversal_error;
}
```

- High Level에서는 변수를 통해 Document 경로만을 허용하도록 하도록 하여 보안대책을 구현하였다.