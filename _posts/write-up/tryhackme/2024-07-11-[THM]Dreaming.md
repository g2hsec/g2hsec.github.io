---
layout: single
title: Dreaming - WriteUP
categories: THM
tag: [THM,TryHackMe, Pentest, Pentesting, Dreaming]
toc: true
author_profile: false
---

해당 문제는 [https://tryhackme.com/r/room/dreaming](https://tryhackme.com/r/room/dreaming) 에서 확인할 수 있습니다.

***

# Dreaming 시스템 모의 침투
## 수행 내용
1. 정보 수집
2. 디렉터리 부르트 포싱으로 /app 경로 확보
3. pluck CMS 4.7.13 의 CVE가 존재
4. 웹쉘 업로드를 통한 쉘 획득
5. /opt 경로의 test.py 파일 내 Lucien 계정정보 하드코딩 
6. .bash_history 파일 내에 Lucien 계정의 mysql 접속 기록 확보
7. Death 계정권한으로 실행시키는 파일 중 DB 내의 값을 가져와 출력시키는 파일 존재
8. DB 내 특정 테이블 컬럼 값 조작으로 인한 Death 계정으로 파일 실행시 악의적인 시스템 명령 실행으로 일시적인 권한 확보 후 특정 파일에서 Death 계정의 계정정보 확보
9. Death 그룹권한으로 수정가능한 파일 확보
10. Morpheus 으로 지속적으로 실행되는 프로세스 식별
11. 프로세스 내에 실행 파일에 Death 권한으로 수정가능한 파일을 불러와 사용
12. 해당 파일 악의적은 변조 후 리버스 커넥션
13. Morpheus 권한에서의 SUDO 권한 확인시 패스워드없이 root 권한으로 모든 명령 수행 가능
14. sudo -i 를 통한 root 권한 획득
## 정보수집
### Nmap 스캔 - 사용중인 포트 및 배너, 기본 정보 수집

```
sudo nmap -p- --open -sV -T 4 -n -o result  {target_ip}
```

```
Starting Nmap 7.93 ( https://nmap.org ) at 2024-07-10 21:01 EDT
Nmap scan report for 10.10.174.238
Host is up (0.41s latency).
Not shown: 64852 closed tcp ports (reset), 681 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 271.00 seconds
```

- 21(ftp), 22(ssh), 8081(http), 31331(http) 서비스 동작중
- 두 개의 포트로 웹 서비스가 구동중인 것을 확인 했다.
- 우선적으로 웹 서비스부터 탐색을 시도했다.

![그림 1-1](/assets/image/write-up/thm/thm_Dreaming/image.png)
- 기본적인 아파치 기본 페이지가 출력되고있다.

![그림 1-2](/assets/image/write-up/thm/thm_Dreaming/image-1.png)
- 해당 웹 서버에 대한 기술스태에 대해 확인 하였으며, 크게 유의미한 정보는 얻을 수 없다

![그림 1-3](/assets/image/write-up/thm/thm_Dreaming/image-2.png)
- 디렉터리 부르트포싱 결과 
- app 경로를 추가 확인할 수 있었다.

![그림 1-4](/assets/image/write-up/thm/thm_Dreaming/image-3.png)
- 확인 결과 pluck CMS 4.7.13 버전을 사용하고 있다.

![그림 1-5](/assets/image/write-up/thm/thm_Dreaming/image-4.png)
- ADMIN 패널이 존재했으며, 패스워드는 기본 default password로 "password"로 설정되어 있다.
- 또한 pluck CMS 4.7.13 의 CVE가 존재하여, 해당 PoC 를 통해 웹쉘 업로드를 통한 공격이 가능하다.

![그림 1-6](/assets/image/write-up/thm/thm_Dreaming/image-5.png)
<br>

![그림 1-7](/assets/image/write-up/thm/thm_Dreaming/image-6.png)
- 성공적으로 웹쉘을 업로드하여 실행 시킬 수 있다.

![그림 1-8](/assets/image/write-up/thm/thm_Dreaming/image-7.png)
- 여러 경로를 탐색중 opt 경로에 test.py가 존재하였다.
- 해당 파일에 lucien 계정에 대한 계정정보가 하드코딩되어 있다.

![그림 1-9](/assets/image/write-up/thm/thm_Dreaming/image-8.png)
- 성공적으로 lucien 계정으로의 접근이 가능하다.

![그림 1-10](/assets/image/write-up/thm/thm_Dreaming/image-9.png)
- .bash_histroy 파일을 읽었을 때 lucien 계정으로 Database 에 접근한 명령어 이력을 확인할 수 있다.

![그림 1-11](/assets/image/write-up/thm/thm_Dreaming/image-10.png)
- 또한 lucien 계정에 대한 sudo 권한을 확인해보면, death 계정의 권한으로 /usr/bin/python3 /home/death/getDreams.py 명령을 실행시킬 수 있는걸 알 수 있다.

![그림 1-12](/assets/image/write-up/thm/thm_Dreaming/image-11.png)
- sudo 를 통해 명령 실행시 특정 문자열들이 출력되고
- 다른 유의미한 기능은 존재하지 않는 듯 하다.

![그림 1-13](/assets/image/write-up/thm/thm_Dreaming/image-12.png)
- 이전에 발견한 mysql 접속 정보를 토대로 Database에 접근하였으며,
- 특정 DB, TABLE를 발견하여 확인해본 결과
- sudo 명령을 통해 실행시 출력되는 문자열과 동일한 값들이 생성되어 있는 걸 볼 수 있다.
- 즉, 해당 py파일을 실행시키면 DB에서 데이터를 가져와 출력시키는 걸 알 수 있다.
- 이 때 테이블 내에 데이터를 임의 추가 혹은 생성시 악의적인 명령을 추가시켜 출력시키는 과정에서 실행시킬 수 있게 된다.

![그림 1-14](/assets/image/write-up/thm/thm_Dreaming/image-13.png)
<br>

![그림 1-15](/assets/image/write-up/thm/thm_Dreaming/image-14.png)
- 성공적으로 /bin/bash 을 실행시켜 death의 쉘을 획득 하였지만, 명령어의 출력이 나오지 않는다.
- 명령은 성공적으로 실행되므로, getDreams.py에 Other에 대한 읽기 권한을 부여하여 따로 확인이 가능하다.

![그림 1-16](/assets/image/write-up/thm/thm_Dreaming/image-15.png)
- 해당 파일 내에 Death 사용자에 대한 계정정보가 하드코딩 되어 있다.

![그림 1-17](/assets/image/write-up/thm/thm_Dreaming/image-16.png)
- Death 그룹 권한으로 실행 가능한 파일들을 살펴보면,
- /usr/lib/python3.8/shutil.py을 확인할 수 있다.

![그림 1-18](/assets/image/write-up/thm/thm_Dreaming/image-17.png)
- 해당 파일을 보게되면 소유자는 root로 된 것을 알 수 있다.

![그림 1-19](/assets/image/write-up/thm/thm_Dreaming/image-18.png)
<br>

![그림 1-20](/assets/image/write-up/thm/thm_Dreaming/image-19.png)
- UID가 1002인 morpheus 사용자로 해당 명령을 지속적으로 실행시키는 걸 볼 수 있다.

![그림 1-21](/assets/image/write-up/thm/thm_Dreaming/image-20.png)
- 또한 morpheus 계정의 홈 디렉토리에서 restore.py를 보게되면 shutil 파일을 불러와서 사용한다.
- 정리해보면 death 사용자가 그룹으로 shutil.py파일을 수정할 수 있고 restore.py 파일에서 shutil을 불러와 사용하게된다.
- 또한 cron과 같이 지속적으로 morpheus 유저로 restore.py 파일을 실행시키고 있다.
- 즉 shutil 파일을 리버스커넥션을 수행하도록 내용을 변조시켜, morpheus 계정의 쉘을 획득할 수 있다.

![그림 1-22](/assets/image/write-up/thm/thm_Dreaming/image-21.png)
- shutil.py 에 python 리버스 커넥션 코드로 변조 하였으며,

![그림 1-23](/assets/image/write-up/thm/thm_Dreaming/image-22.png)
- 성공적으로 morpheus 계정의 쉘을 획득할 수 있다.

![그림 1-24](/assets/image/write-up/thm/thm_Dreaming/image-23.png)
- 이후 morpheus 계정의 sudo 권한 확인시 root 권한으로의 패스워드 없이 모든 명령 실행이 가능했다.
- sudo -i 를 통해 손쉽게 root 권한 획득이 가능하다.
