---
layout: single
title: Flawed Broadcast Receivers
categories: Android
tag: [android, mobile, drozer,insecurebankv2, app, application]
toc: true
author_profile: false
---

# Flawed Broadcast Receivers

## Broadcast Receivers란 무엇인가?

<div class="notice--primary" markdown="1">
애플리케이션의 4대 구성 단위중 하나로서 안드로이드 앱에서 발생하는 시스템 브로드캐스트 메시지를 수신하여 처리하는 구성 요소이다. 브로드캐스트 리시버는 안드로이드 앱의 다른 구성 요소들이나 외부 시스템에서 발생한 이벤트에 반응하여 특정 작업을 수행할 수 있도록 해준다.

> 여기서 브로드캐스트 메시지란 시스템에서 발생하는 이벤트를 알리기 위한 메시지이며, 이러한 메시지를 이벤트 혹인 인텐트(intent)라고도 부른다. 예를 들어 애플리케이션은 브로드캐스트를 시작하여 다른 애플리케이션에 일부 데이터가 디바이스에 다운로드 되어 사용할 수 있음을 알릴 수 있으며, 브로드캐스트 리시버는 이 통신을 가로채 적절한 조치를 취한다.`

```
브로드캐스트 리시버는 안드로이드의 매니페스트 파일(AndroidManifest.xml 의 <receiver></receiver>항목)에 등록되어야 하며, 등록된 브로드캐스트 이벤트가 발생했을 경우 시스템에 의해 호출된다.
``` 

브로드 캐스트 리시버가 정상적으로 작동하려면 우선 브로드캐스트 리시버를 생성 및 등록이 이루어져야한다.
<br>
브로드캐스트 리시버는 BroadcastReceiver 클래스의 서브클래스로 구현되며, 각 메시지를 intent 객체 파라미터로 수신하는 onReceiver() 메서드를 재정의 한다.

![그림 1-1](/assets/image/vuln/mobile-vuln/adnroid-vuln/Flawed%20Broadcast%20Receivers/image.png)

[Example Code]

```java
public class MyReceiver extends BroadcastReceiver {
   @Override
   public void onReceive(Context context, Intent intent) {
      Toast.makeText(context, "Intent Detected.", Toast.LENGTH_LONG).show();
   }
}
```

</div>

어플리케이션에서 선언한 액션을 호출하면 리시버는 해당 액션을 인지하여 작업을 수행하게 하고, 이러한 작업은 브래드캐스트 리시버를 상속 받은 메서드에서 처리하게 된다.
<br><br>
브로드캐스트 리시버 호출 시 발생하는 브로드캐스트가 정상이면 각각의 시스템 이벤트와 다른 어플리케이션에서 발생하는 경우가 있으며, 비정상일 경우 악의적인 어플리케이션에서 발생하거나, 악의적인 사용자에 의해 임의대로 생성할 수 있게 된다.

## Flewed Broadcas Receivers
💡 **<u><span style="background-color: yellow; ">해당 취약점 실습은 "InsecureBank2"라는 모바일 뱅킹서비스 어플리케이션으로 진행합니다.</span></u>** 
{: .notice--primary}

### 취약점 분석

```xml
<receiver
   android:name=".MyBroadCastReceiver"
   android:exported="true" >
   <intent-filter>
         <action android:name="theBroadcast" >
         </action>
   </intent-filter>
</receiver>
```

위 코드는 진단 어플리케이션의 AndroiManifest.xml 파일에 선언된 <receiver> 항목이다.
<br><br>
해당 코드를 보게되면, 브로드캐스트 리시버는 MyBroadCastReceiver이며, exported 가 true로 지정되어 있다. 이는 해당 브로드캐스트 리시버가 다른 어플리케이션에서 사용될 수 있음을 의미한다. 즉, 외부 어플리케이션으로부터 intent를 받을 수 있다. 또한 브로드캐스트 리시버가 처리할 액션은 theBroadcast이다. 즉, 해당 어플리케이션 혹은 다른 어플리케이션에서 해당 액션을 보낼 때 브로드캐스트 리시버가 해당 액션을 수신하여 처리할 수 있게 된다.
<br><br>
이후 Android Device가 부팅될 때마다 브로드캐스트 리시버인 MyBrocastReciver가 이를 가로채고 onRecive() 내부에 구현된 로직이 실행되게 된다.


```java
public class MyBroadCastReceiver extends BroadcastReceiver {
	String usernameBase64ByteString;
	public static final String MYPREFS = "mySharedPreferences";

	@Override
	public void onReceive(Context context, Intent intent) {
		// TODO Auto-generated method stub

        String phn = intent.getStringExtra("phonenumber");
        String newpass = intent.getStringExtra("newpass");

		if (phn != null) {
			try {
                SharedPreferences settings = context.getSharedPreferences(MYPREFS, Context.MODE_WORLD_READABLE);
                final String username = settings.getString("EncryptedUsername", null);
                byte[] usernameBase64Byte = Base64.decode(username, Base64.DEFAULT);
                usernameBase64ByteString = new String(usernameBase64Byte, "UTF-8");
                final String password = settings.getString("superSecurePassword", null);
                CryptoClass crypt = new CryptoClass();
                String decryptedPassword = crypt.aesDeccryptedString(password);
                String textPhoneno = phn.toString();
                String textMessage = "Updated Password from: "+decryptedPassword+" to: "+newpass;
                SmsManager smsManager = SmsManager.getDefault();
                System.out.println("For the changepassword - phonenumber: "+textPhoneno+" password is: "+textMessage);
                smsManager.sendTextMessage(textPhoneno, null, textMessage, null, null);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
        else {
            System.out.println("Phone number is null");
        }
	}

}
```

MyBroadCastReceiver 코드를 보게되면, BroadcastReceiver 객체를 상속받고, phn과 newpass를 인자로받으며, 브로드캐스트가 발생하면 onReceive() 메서드가 실행되게 된다. 해당 메서드를 보게되면, 암호화된 사용자 이름과 패스워드를 가져와서 복호화하여 SMS에 포함될 메시지로 사용하며, 사용자 전화번호로 보내도록 되어있다. 이 때 전화번호가 입력되지 않는 다면 "Phone number is null"을 출력하게 된다.
<br><br>
Drozer을 이용하여 실습을 진행해 보았다.

```
run app.package.attacksurface com.android.insecurebankv2
```

![그림 1-2](/assets/image/vuln/mobile-vuln/adnroid-vuln/Flawed%20Broadcast%20Receivers/image1.png)
- Drozer을 통해 취약점 유/무 확인이 가능하다.
- 해당 내용으로는 content providers exported 가 1로 되어 있다. 이는 외부 어플리케이션에서 접근할 수 있는 서비스의 수를 의미하며, 외부 어플리케이션에서 해당 서비스를 시작할 수 있다. 해당 부분이 공격 표면이 된다.

```
run app.broadcast.info -a com.android.insecurebankv2 -i
```

![그림 1-3](/assets/image/vuln/mobile-vuln/adnroid-vuln/Flawed%20Broadcast%20Receivers/image2.png)
- 위 내용은 브로드캐스트 리시버 정보를 확인할 수 있다.
- 살펴보면 com.android.insecurebankv2패키지의 MyBroadCastReceiver 라는 브로드캐스트 리시버가 포함되어 있다는 것을 알 수 있다. 앞에서 확인한 정보와 일치한다.
- 권한 설정은 되어있지 않다.
- 이후 새로운 브로드캐스트를 생성하여, 브로드캐스트 리시버에 정의된 액션을 수행할 수 있다.

```
run app.broadcast.send --component com.android.insecurebankv2 com.android.insecurebankv2.MyBroadCastReceiver
```
위와 같이 --extra 옵션 없이 임의 브로드캐스트를 생성하여 전송하게 되면 logcat정보에 인자값이 없을 경우 발생하는 문자가 출력된다.

![그림 1-4](/assets/image/vuln/mobile-vuln/adnroid-vuln/Flawed%20Broadcast%20Receivers/image-1.png)


```
run app.broadcast.send --action theBroadcast --extra string phonenumber 1234 --extra string newpass crack

or 

run app.broadcast.send --component com.android.insecurebankv2 com.android.insecurebankv2.MyBroadCastReceiver --extra string phonenumber 1234 --extra string newpass crack
```
위 코드들과 같이 보낼 경우 출력되는 메시지에는 기존에 사용하던 패스워드가 평문으로 포함되어 노출되게 된다. 즉, 계정의 패스워드는 변경되지 않지만 기존의 비밀번호는 노출되게 된다.

<div class="notice">
  <h4>현재 Insecurebank2의 경우에는 onReceive() 메서드에서 SharedPreferences를 사용하는 부분에서 MODE_WORLD_READABLE를 사용하고 있다. 이는 안드로이드에서 더이상 지원하지 않고있으며, API Level 17 부터 사용이 금지되었다. 이로인해 위 명령어는 실행되지 않는다. 이를 실행 시키기 위해서는 MODE_WORLD_READABLE 부분을 MODE_PRIVATE로 변경하고 다시금 빌드를 해야한다.</h4>
</div>
Message

# Referance
- https://www.tutorialspoint.com/android/android_broadcast_receivers.htm