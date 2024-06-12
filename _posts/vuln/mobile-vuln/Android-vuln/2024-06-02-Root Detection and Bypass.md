---
layout: single
title: Root Detection and Bypass
categories: Android
tag: [android, mobile, drozer,insecurebankv2, app, application, Root Detection and Bypass]
toc: true
author_profile: false
---

# Root Detection and Bypass이란?
> 루팅이란 안드로이드 시스템 권한을 얻는 과정 즉, 슈퍼 유저의 권한으로 하드웨어의 다양한 조작이 가능한 것 을 일컫는다. 안드로이드의 경우, 독립적으로 동작하므로, 루팅이 되지 않는다면 어플리케이션에서 할 수 있는 행위에 제한이 생긴다. 이렇기에 안드로이드 어플리케이션에서 침투 테스트시 가장 먼저 하는 일이 루팅된 장치인 환경에서 앱을 실행할 수 있는지에 대한 확인이다.

## 루팅 감지
su 시스템 경로에서 바이너리를 확인함으로써, 루팅되었는지에 대한 여부가 확인 가능하다.

<div class="notice">
1. /system/bin/su
2. /system/xbin/su
3. /sbin/su
4. /system/sbin/su
5. /system/bin/.ext/.su
6. /system/xbin/mu
7. /system/app/superuser.apk
8. /data/data/com.noshufou.android.su
</div>

위 경로들을 통해 체크할 수 있다.

💡 **<u><span style="background-color: yellow; ">해당 취약점 실습은 "InsecureBank2"라는 모바일 뱅킹서비스 어플리케이션으로 진행합니다.</span></u>** 
{: .notice--primary}

```java
    void showRootStatus() {
        boolean isrooted = doesSuperuserApkExist("/system/app/Superuser.apk") || doesSUexist();
        if (isrooted) {
            this.root_status.setText("Rooted Device!!");
        } else {
            this.root_status.setText("Device not Rooted!!");
        }
    }

    private boolean doesSUexist() {
        Process process = null;
        try {
            process = Runtime.getRuntime().exec(new String[]{"/system/bin/which", "su"});
            BufferedReader in = new BufferedReader(new InputStreamReader(process.getInputStream()));
            if (in.readLine() != null) {
                if (process != null) {
                    process.destroy();
                }
                return true;
            }
            if (process != null) {
                process.destroy();
            }
            return false;
        } catch (Throwable th) {
            if (process != null) {
                process.destroy();
            }
            return false;
        }
    }
    
    private boolean doesSuperuserApkExist(String s) {
        File rootFile = new File("/system/app/Superuser.apk");
        Boolean doesexist = Boolean.valueOf(rootFile.exists());
        return doesexist.booleanValue();
    }

```
위 코드는 PostLogin 페이지의 루팅을 체크하는 부분이다.<br>
doesSUexist() 함수는 su 프로세스가 존재하는지 확인하고 showRootStatus와 doesSuperuserApkExist함수의 경우 superuser.apk 파일 존재를 확인한다. 그 후 존재할 경우 루팅 유.무를 출력하고 이다. 