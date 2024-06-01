---
layout: single
title: Vulnerable Activity Components
categories: Android
tag: [android, mobile, drozer,insecurebankv2, app, application, Vulnerable Activity Components]
toc: true
author_profile: false
---

# Vulnerable Activity Components이란?
> 어플리케이션 구성요소중 기본적인 구성 단위중 하나로, 어플리케이션과 사용자간의 상호 작용에 필요한 기능을 제공하는 것을 액티비티 라고 한다. 이러한 액티비티는 AndroidManfiest.xml 파일에 <activity> 태그로 선언된다. 이러한 액티비티는 시스템에서 스택으로 관리하며, 액티비티는 독립적으로 동작하므로 한번의 하나의 액티비티가 실행되며, 요청마다 번갈아 가며 사용된다. 이 때 액티비티가 절절하게 선언되지 않는다면 로직을 우회하여 악의적인 사용자는 본인의 권한을 넘어서는 특정 액티비티에 접근이 가능하다. 


```java
'''
<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@android:style/Theme.Holo.Light.DarkActionBar">
        <!--
        android:theme="@style/AppTheme"-->
        <activity
            android:name=".LoginActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity
            android:name=".FilePrefActivity"
        android:label="@string/title_activity_file_pref"
        android:windowSoftInputMode="stateVisible|adjustResize|adjustPan">
    </activity>
'''
```

💡 **<u><span style="background-color: yellow; ">해당 취약점 실습은 "InsecureBank2"라는 모바일 뱅킹서비스 어플리케이션으로 진행합니다.</span></u>** 
{: .notice--primary}

## 취약점 분석

### Drozer를 통한 분석

![그림 1-1](/assets/image/vuln/mobile-vuln/adnroid-vuln/Vulnerable%20Activity%20Components/image.png)
<br>
드로저를 통해 현재 노출된 액티비트에 대해 확인이 가능하다.<br>
위 내용으로는 현재 5개의 액티비티가 노출되어 있다. 또한 액티비티 사용에 대해 권한이 설정되어 있지 않다.<br>
즉, exported 속성이 true로 부적절하게 설정되어 있다는 것을 알 수 있다.

![그림 1-2](/assets/image/vuln/mobile-vuln/adnroid-vuln/Vulnerable%20Activity%20Components/image-1.png)
<br>
현재 어플리케이션에서는 로그인을 하지 않은 상태이다.
![그림 1-3](/assets/image/vuln/mobile-vuln/adnroid-vuln/Vulnerable%20Activity%20Components/image-2.png)
<br>
이때 드로저를 이용하여 ChangePassword 액티비티를 강제로 실행시켜보았다.<br>
원래의 경우 해당 액티비티는 로그인이 우선 진행되어야 해당 액티비티에 접근이 가능하다.

![그림 1-4](/assets/image/vuln/mobile-vuln/adnroid-vuln/Vulnerable%20Activity%20Components/image-3.png)
<br>
그 결과 로그인 과정 없이 패스워드 변경 페이지로 이동된 것을 볼 수 있다.<br>
이를 통해 특정 사용자의 계정명을 알고 있을 경우 악의적인 사용자는 임의의 패스워드루 강제 변경이 가능하다.

```java
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_change_password);

        // Get Server details from Shared Preference file.
		serverDetails = PreferenceManager.getDefaultSharedPreferences(this);
		serverip = serverDetails.getString("serverip", null);
		serverport = serverDetails.getString("serverport", null);

		changePassword_text = (EditText) findViewById(R.id.editText_newPassword);
		Intent intent = getIntent();
		uname = intent.getStringExtra("uname");
		System.out.println("newpassword=" + uname);
		textView_Username = (TextView) findViewById(R.id.textView_Username);
		textView_Username.setText(uname);

        // Manage the change password button click
		changePassword_button = (Button) findViewById(R.id.button_newPasswordSubmit);
		changePassword_button.setOnClickListener(new View.OnClickListener() {

			@Override
			public void onClick(View v) {
				// TODO Auto-generated method stub
				new RequestChangePasswordTask().execute(uname);
			}
		});
	}
```

해당 코드는 ChangePassword.java 코드의 일부이다 해당 내용을 보면, username을 우선 전달받아 textView_Username를 통해 화면에 표시한다. 즉 username은 입력이 아닌 로그인된 사용자의 username을 받아오는 것을 알 수 있다.

![그림 1-5](/assets/image/vuln/mobile-vuln/adnroid-vuln/Vulnerable%20Activity%20Components/image-4.png)
<br>
드로저를 통해 해당 username을 입력된 상태로 ChangePassword 액티비티를 실행시킬 수 있다.

![그림 1-6](/assets/image/vuln/mobile-vuln/adnroid-vuln/Vulnerable%20Activity%20Components/image-5.png)
<br>
이와 같이 입력되지 않던 username 부분이 jack 사용자로 고정되고 패스워드 변경이 가능하다.

![그림 1-7](/assets/image/vuln/mobile-vuln/adnroid-vuln/Vulnerable%20Activity%20Components/image-6.png)
<br>
변경 또한 성공적으로 된 것을 알 수 있다.