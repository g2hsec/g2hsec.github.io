---
layout: single
title: (bWAPP)Directory Traversal - Files
categories: bwapp
tag: [Directory Traversal,Path Traversal, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> Directory Traversal 혹은 path traversal은 ./(현재 디렉토리) 또는 ../(상위 디렉토리로 이동)를 나타내는 문자들이 제대로 필터링되지 않아 파일 시스템 API로 전달되어, 허용된 디렉토리가 아닌 숨겨진 디렉토리 및 파일에 대한 무단 access를 가능하게 하는 취약점이며, 보통 File 명과 같이 사용자의 입력값이 전달되어, 찾아오는 로직에서 많이 발견된다.

## Low-Level

![그림 1-1](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Directory%20Traversal%20-%20Files/image.png)
- 따로 특정 기능이 존재하는 거 같지는 않다.

```
/bWAPP/directory_traversal_1.php?page=message.txt
```

- URL를 보게되면 특정 페이지를 page파라미터를 통해 Include 하는 듯한 것을 알 수 있다.

```
/bWAPP/directory_traversal_1.php?page=../
```

- 위와 같이 경로이동 문자를 시도해보았다.

![그림 1-2](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Directory%20Traversal%20-%20Files/image-1.png)
- 파일이 존재하지 않는다고 출력된다.
- 즉 디렉터리가 아닌 파일명만을 인자로 받고 있다.

```
/bWAPP/directory_traversal_1.php?page=../../../../../etc/passwd
```

- 개념증명을 위해 최상된 디렉터리 아래 etc/passwd 파일 출력을 시도했다.

![그림 1-2](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Directory%20Traversal%20-%20Files/image-2.png)
- 성공적으로 passwd 파일이 노출 되었다.
- 이를 통해 다양한 파일을 무단 열람할 수 있다.

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

- High Level에서는 변수를 통해 특정 경로만을 허용하도록 하도록 하여 보안대책을 구현하였다.