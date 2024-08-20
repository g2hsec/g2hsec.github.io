---
layout: single
title: UltraTech - WriteUP
categories: THM
tag: [THM,TryHackMe, Pentest, Pentesting]
toc: true
author_profile: false
---

해당 문제는 [https://tryhackme.com/r/room/ultratech1](https://tryhackme.com/r/room/ultratech1) 에서 확인할 수 있습니다.

***

# UltraTech 시스템 모의 침투
## 수행 내용
1. 정보 수집
2. robots.txt 및 sitemap 발견
3. restAPI사용을 통한 숨겨진 페이지 및 Command injection 취약점 발견
4. 취약한 해시함수로 해시화된 계정정보 및 패스워드 획득
5. 획득한 계정정보로 로그인한 사용자가 속한 그룹이 docker그룹을 통한 docker 명령어로 root 권한 획득
## 정보수집
### Nmap 스캔 - 사용중인 포트 및 배너, 기본 정보 수집

```
nmap -p- --max-retries 1 -n --open --min-rate 5000 -T4 -sV -sC -A -oA ./ultra {target_ip}
```

```
Starting Nmap 7.93 ( https://nmap.org ) at 2024-04-09 07:46 EDT
Warning: 10.10.237.105 giving up on port because retransmission cap hit (1).
Nmap scan report for 10.10.237.105
Host is up (0.42s latency).
Not shown: 33212 closed tcp ports (conn-refused), 32319 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc668985e705c2a5da7f01203a13fc27 (RSA)
|   256 c367dd26fa0c5692f35ba0b38d6d20ab (ECDSA)
|_  256 119b5ad6ff2fe449d2b517360e2f1d2f (ED25519)
8081/tcp  open  http    Node.js Express framework
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
|_http-cors: HEAD GET POST PUT DELETE PATCH
31331/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: UltraTech - The best of technology (AI, FinTech, Big Data)
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 83.48 seconds
```

- 21(ftp), 22(ssh), 8081(http), 31331(http) 서비스 동작중
- 두 개의 포트로 웹 서비스가 구동중인 것을 확인 했다.
- 우선적으로 웹 서비스부터 탐색을 시도했다.
- 8181 포트로 동작중인 웹 서비스는 node.js express framework를 사용한것으로 보이며
- 31331 포트로 동작중인 웹 서비스는 AI와 빅데이터를 처리하는 웹서비스로 보인다.

### 특정 API 이용 확인

![그림 1-1](/assets/image/write-up/thm/thm_UltraTech/image.png)
- 8081 서비스로 접속하면 UltraTech API v0.1.3 을 사용한다고 나온다.

![그림 1-2](/assets/image/write-up/thm/thm_UltraTech/image-1.png)
- 우선 8081 포트로 동작중인 사이트에 디렉터리부르트포싱 결과 auth 라는 경로가 존재하는 것을 확인함

![그림 1-3](/assets/image/write-up/thm/thm_UltraTech/image-2.png)
- 해당 경로로 접근시 로그인을 진행해야하는 듯한 문자가 출력됨

![그림 1-4](/assets/image/write-up/thm/thm_UltraTech/image-3.png)
- 8081 포트 및 31331 포트에 대한 디렉터리 부르트 포싱 결과 추가적인 유의미한 결과는 획득하지 못하였다.

### robots.txt 및 sitemap 식별

![그림 1-5](/assets/image/write-up/thm/thm_UltraTech/image-4.png)
- 31331 포트에 robots.txt 경로가 확인되었으며, sitemap이 존재함.

![그림 1-6](/assets/image/write-up/thm/thm_UltraTech/image-5.png)
- siteamp 화깅ㄴ 결과 3개의 경로가 확인됨
- index.html과 what.html은 31331 포트의 웹 페이지상에 메인 페이지와 about 페이지이며
- 추가적인 partners.html 경로를 획득

![그림 1-7](/assets/image/write-up/thm/thm_UltraTech/image-6.png)
- 숨겨진 페이가 확인됨
- id 및 pw는 확인 불가

```html

<!DOCTYPE html>
<html lang='en'>
<head>
	<meta class="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
	<title>UltraTech | Authentication</title>
	<link rel='stylesheet' href='css/style.min.css' />
</head>
<body>
	<!-- navbar -->
	<div class="navbar">
		<nav class="nav__mobile"></nav>
		<div class="container">
			<div class="navbar__inner">
				<a href="#" class="navbar__logo">UltraTech</a>

				<div class="navbar__menu-mob"><a href="" id='toggle'><svg role="img" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512"><path fill="currentColor" d="M16 132h416c8.837 0 16-7.163 16-16V76c0-8.837-7.163-16-16-16H16C7.163 60 0 67.163 0 76v40c0 8.837 7.163 16 16 16zm0 160h416c8.837 0 16-7.163 16-16v-40c0-8.837-7.163-16-16-16H16c-8.837 0-16 7.163-16 16v40c0 8.837 7.163 16 16 16zm0 160h416c8.837 0 16-7.163 16-16v-40c0-8.837-7.163-16-16-16H16c-8.837 0-16 7.163-16 16v40c0 8.837 7.163 16 16 16z" class=""></path></svg></a></div>
			</div>
		</div>
	</div>
	<!-- Authentication pages -->
	<div class="auth">
		<div class="container">
			<div class="auth__inner">
				<div class="auth__media">
					<img src="./images/undraw_selfie.svg">
				</div>
				<div class="auth__auth">
					<h1 class="auth__title">Private Partners Area</h1>
					<p>Fill in your login and password</p>
					<form method='GET' autocompelete="new-password" role="presentation" class="form">
						<label>Login</label>
						<input type="text" name="login" id='email' placeholder="your login">
						<label>Password</label>
						<input type="password" name="password" id='password' placeholder="&#9679;&#9679;&#9679;&#9679;&#9679;&#9679;&#9679;&#9679;&#9679;" autocomplete="off">
						<button type='submit' class="button button__accent">Log in</button>
						<a href=""><h6 class="left-align" >Forgot your password?</h6></a>
					</form>
				</div>
			</div>
		</div>
	</div>
	<script src='js/app.min.js'></script>
	<script src='js/api.js'></script>
</body>
</html>
```

### Rest API 사용 확인 

- 해당 페이지의 소스코드이다.
- 맨 아래 api.js 가 보이며, 특정 API를 사용하는 것 같다.

```javascript
(function() {
    console.warn('Debugging ::');

    function getAPIURL() {
	return `${window.location.hostname}:8081`
    }
    
    function checkAPIStatus() {
	const req = new XMLHttpRequest();
	try {
	    const url = `http://${getAPIURL()}/ping?ip=${window.location.hostname}`
	    req.open('GET', url, true);
	    req.onload = function (e) {
		if (req.readyState === 4) {
		    if (req.status === 200) {
			console.log('The api seems to be running')
		    } else {
			console.error(req.statusText);
		    }
		}
	    };
	    req.onerror = function (e) {
		console.error(xhr.statusText);
	    };
	    req.send(null);
	}
	catch (e) {
	    console.error(e)
	    console.log('API Error');
	}
    }
    checkAPIStatus()
    const interval = setInterval(checkAPIStatus, 10000);
    const form = document.querySelector('form')
    form.action = `http://${getAPIURL()}/auth`;
    
})();
```


## 취약점 분석
### 8081 포트로의 Rest API 사용 및 숨겨진 페이지 내에 Command Injection 취약점 식별

- Rest API 를 사용하는 걸로 확인되며,
- getAPIURL 부분을 보면 포트가 8081 즉, 8081포트로 동작중인 웹 사이트와의 데이터 전송이 이루어진다.
- 또한 8081 포트로 동작하는 웹 사이트에 ping 경로가 존재하며 ip 파라미터를 통해 특정 호스트가 동작중인지 확인하는 api가 구현되어 있는 듯 하다.
- 이 때 ping 경로에 ip 파라미터에서 command injection취약점이 존재할 가능성이 있으므로, 해당 구간을 테스트 해보았다.

![그림 1-8](/assets/image/write-up/thm/thm_UltraTech/image-7.png)
- 실제 페이지가 존재하였으며, 호스트네임에 대한 결과를 출력한다.

![그림 1-9](/assets/image/write-up/thm/thm_UltraTech/image-8.png)
- ; 를 통한 Command Injection 을 진행 하였으나 출력되는 문자열로 짐작을 해보면
- ; 문자열이 시스템 내에서 명령어의 구분자로 사용되지 않고 하나의 문자로 합쳐지는 듯 하다.

![그림 1-10](/assets/image/write-up/thm/thm_UltraTech/image-9.png)
- `(백틱) 을 통한 추가 우회를 시도하였더니 정상적으로 id 명령어에 대한 출력값이 나왔다.
- 이를 통해 Command Injection 취약점이 존재한다는 것을 확인할 수 있다.

### 취약한 해시함수인 md5 를 통해 해시화된 계정정보 패스워드 확인

![그림 1-11](/assets/image/write-up/thm/thm_UltraTech/image-10.png)
- ls 결과 utech.db.sqlite 라는 데이터베이스 파일이 존재하는 것을 확인할 수 있다.

![그림 1-12](/assets/image/write-up/thm/thm_UltraTech/image-11.png)
- 해당 데이터베이스 파일에는 해시화된 특정 계정정보들이 저장되어 있는 듯 하다.

![그림 1-13](/assets/image/write-up/thm/thm_UltraTech/image-12.png)
- r00t 계정과 admin 계정으로 보이며
- 두 계정의 해시화된 패스워드는 md5로 저장되어 있는 듯 하다
- md5는 취약한 해시 알고리즘으로 복호화가 가능하다.


## 내부 침투
### r00t 사용자에 대한 패스워드 크랙 성공 및 SSH 접근 성공

![그림 1-14](/assets/image/write-up/thm/thm_UltraTech/image-13.png)
- hashcat 결과 r00t 유저의 패스워드는 n100906 로 크랙에 성공 했다.

![그림 1-15](/assets/image/write-up/thm/thm_UltraTech/image-14.png)
- admin 유저 또한 mrsheafy 라는 패스워드로 크랙에 성공 했다.

![그림 1-16](/assets/image/write-up/thm/thm_UltraTech/image-15.png)
- ssh 서비스가 동작 중이었으며 r00t 유저로 ssh 접근이 가능했다.
- admin 유저로는 ssh 접근이 불가능 했다.
- admin 유저는 데이터베이스 유저 목록 중 하나(관리자?)로 추정되며 해당 서버에는 admin 이라는 유저가 존재하지 않았다.

![그림 1-17](/assets/image/write-up/thm/thm_UltraTech/image-16.png)
- 여러 경로를 순회 하면서 정보들을 수집했다.
- lp1 이라는 유저의 사용자 홈 디렉터리에 접근이 가능했으며
- 숨겨진 파일중 .sudo_as_admin_successful 파일이 존재했다.
- 해당 파일은 sudo 명령어를 사용하여 관리자 권한으로 명령어를 실행할 때 생성되는 임시 파일이다.
- 하지만 해당 파일 내에 존재하는 데이터는 없었다.

## 권한 상승
### r00t 유저가 속한 docker 그룹을 이용한 docker 명령어를 악용하여 root 권한 획득

![그림 1-18](/assets/image/write-up/thm/thm_UltraTech/image-18.png)
- 해당 r00t 유저가 속해있는 그룹에 docker 그룹이 존재하고 해당 서버는 docker 위에서 동작중이라는 것을 알 수 있다.
- 이 떄 docker 명령어를 통해 root 사용자의 권한을 획득할 수 있다.