---
layout: single
title: (HTB) Editorial - WriteUP
categories: HTB
tag: [SSRF, RCE, HTB,HackTheBox, Pentest, Pentesting, git, gitpython]
toc: true
author_profile: false
---

해당 문제는 [https://app.hackthebox.com/machines/Editorial](https://app.hackthebox.com/machines/Editorial) 에서 확인할 수 있습니다.

***

# Editorial 시스템 모의 침투
## 수행 내용
1. 정보 수집
2. 도멩인 획득 후 IP와 매칭
3. 사이트 내에 사진 Preview 기능에서 SSRF 취약점 식별
4. SSRF 공격을 통해 내부 5000번 포트에서 API 경로 노출 API 경로 내 파일 안에 계정정보 하드코딩
5. prod 계정의 sudo 권한에서 gitpython 사용부분의 취약점 악용으로 RCE 를 통한 root 경로 내 파일 read
## 정보수집
### Nmap 스캔 - 사용중인 포트 및 배너, 기본 정보 수집

```
nmap -p- -Pn --open --min-rate 5000 -T4 -sV  -oA ./result {target_ip}
```

```
# Nmap 7.93 scan initiated Tue Jul  2 20:19:01 2024 as: nmap -p- --open -n -T 4 -sV -o result.txt 10.10.11.20
Nmap scan report for 10.10.11.20
Host is up (0.30s latency).
Not shown: 65308 closed tcp ports (reset), 225 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jul  2 20:20:29 2024 -- 1 IP address (1 host up) scanned in 88.38 seconds
```

- 22(SSH), 80(http) 포트로 서비스 동작중을 확인함.
- SSH의 경우 Username 열거에 관한 취약점 제외 유의미한 취약점은 없는듯 함.

![그림 1-1](/assets/image/write-up/htb/Editorial/image.png)
- 80 웹 서비스 포트가 동작중이지만, 해당 타겟 IP로 웹 사이트는 접근이 되지 않고 있다.

![그림 1-2](/assets/image/write-up/htb/Editorial/image-1.png)
- 10.10.11.20 에 대해 http://editorial.htb 로 리다이렉트 되며, 현재 10.10.11.20 ip가 제대로 http://editorial.htb 로 매칭되지 않는 것을 확인

![그림 1-3](/assets/image/write-up/htb/Editorial/image-2.png)
- /etc/hosts 파일에서 IP와 도메인 매칭

![그림 1-4](/assets/image/write-up/htb/Editorial/image-3.png)
- 정상적으로 공격 타겟 웹 페이지에 접근할 수 있음

![그림 1-5](/assets/image/write-up/htb/Editorial/image-4.png)
- 해당 사이트의 기능을 살펴보던중 특정 공격 앤드포인트를 탐색함
- 해당 기능은 사용자의 URI 혹은 사진을 업로드시 preview를 통해 미리보기가 가능하며,

![그림 1-6](/assets/image/write-up/htb/Editorial/image-5.png)
- 특정 파일명으로 변환되어 다운로드 또한 가능함

```
http://editorial.htb/static/uploads/5b7f6b31-2af4-4755-bd67-b6ab3c479c5f
```

- 또한 static/uploads 라는 경로가 노출되고 있음.
- 이 때 서버측에서 로직을 통해 사용자에게 사진을 반환 및 Preview하고 있는걸로 보아
- SSRF 공격의 잠재적 앤드포인트가 될 수 있음

![그림 1-7](/assets/image/write-up/htb/Editorial/image-6.png)
- 해당 URI 경로를 로컬 루프백 주소인 127.0.0.1 로 요청시 응답값이 존재하는 것을 확인
- 이 때 서버내의 내부포트로 동작중인 서비스가 존재할 수 있으므로 각 포트별로 부르트포싱 하여 존재 유무 확인

![그림 1-8](/assets/image/write-up/htb/Editorial/image-7.png)
- 5000포트에서 jpg 파일이 아닌 다른 형태의 파일이 식별됨 

![그림 1-9](/assets/image/write-up/htb/Editorial/image-8.png)
- 해당 파일을 확인해보면 json 형식으로 API 경로가 노출되고 있음
- /api/latest/metadata/messages/authors 경로를 확인해보기로함

![그림 1-10](/assets/image/write-up/htb/Editorial/image-9.png)
- dev 유저의 계정 정보가 하드코딩되어 고스란히 노출되고 있어
- dev 유저로의 ssh 접근이 가능함

![그림 1-11](/assets/image/write-up/htb/Editorial/image-10.png)
- dev 계정으로 접근 후 내부 시스템 탐색 결과 app -> .git -> logs -> HEAD 파일 내에 commit 정보가 존재
- "change(api): downgrading prod to dev" 부분 즉 prod 계정을 dev로 다운그레이드 했다고 한다.
- 현재 시스템에 prod계정이 존재하므로 해당 로그를 살펴보기로 함

![그림 1-12](/assets/image/write-up/htb/Editorial/image-11.png)
- 아래쪽 내용을 보게되면 prod 계정 또한 하드코딩된 채로 노출되고 있어
- 이를 통해 prod 계정으로의 횡적이동이 가능하다.

![그름 1-13](/assets/image/write-up/htb/Editorial/image-12.png)
- prod 계정의 sudo 권한이 존재하며, "/usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py *" 명령어 사용이 가능하다
- clone_prod_Change.py 로 인자를 하나 받게되며.
- 입력받은 인자를 가지고 r.clone_from(url_to_clone, 'new_changes', multi_options=["-c protocol.ext.allow=always"]) 를 수행하게 되는데
- 이 때 clone_from 의 url_to_clone 부분에서 취약점이 발생한다.
- CVE-2022-24439 으로 명명된 해당 취약점은 악의적으로 조작된 원격 URL을 clone 명령에 삽입할 수 있으며, 이 취약점을 악용하는 것은 라이브러리가 입력 인수를 충분히 검증 않고 외부에서 git을 호출하기 때문에 가능하다. 이는 ext 전송 프로토콜을 활성화할 때만 해당된다.
- 즉, 본인 권한 이상의 권한을 사용하여 명령을 수행할 수 있게 된다.

```
sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::sh -c cat% /root/root.txt% >% /tmp/root.txt'
```
- 위 명령을 통해 /root/root.txt 파일을 tmp 경로로 가져올 수 있게된다.


