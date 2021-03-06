# 메인 컴포넌트3 | Broadcast Receiver

> Broadcast Receiver 특징
- UI가 없이 백그라운드 작업 수행
- 묵시적 활성화의 경우, manifest 파일에 등록없이 직접 객체 생성해서 사용 가능
- 해당 경우에는 intent filter를 직접 생성하여 묵시적 활성화 조건을 설정할 수 있음

> Broadcast Receiver의 생명주기 메서드
- onReceive()
백그라운드에서 자신을 호출할 때를 기다리다가 호출이 되면 메세지가 포함된 인텐트를 파라메터로 전달 <br/>
![image](https://user-images.githubusercontent.com/62331803/90577994-26135c80-e1fd-11ea-8f81-e319a2cb1a75.png)
<br/>

> manifest.xml 확인
```xml
        <receiver
            android:name=".MyReceiver"  
            android:enabled="true"  //리시버 호출 가능 여부
            android:exported="true"> //리시버 추출 가능 여부
        </receiver>
 ```
 <br/>
 
 ## Broadcast Receiver 활용예제 | 명시적/묵시적 호출
 > activity_main
 
 ![image](https://user-images.githubusercontent.com/62331803/90579171-25c89080-e200-11ea-9191-2caee608b10d.png)
 <br/>
 
 > MainActivity.java
 ```java
package com.example.receivertest;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Intent;
import android.content.IntentFilter;
import android.os.Bundle;
import android.view.View;

public class MainActivity extends AppCompatActivity {
    // 묵시적 호출에서 사용할 문자열 상수
    private final String ACTION = "com.example.receivertest.MyAction";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // receiver가 실행이 되면 filter 설정이 되게끔. (자원낭비를 막음)
        MyReceiver2 br = new MyReceiver2();
        IntentFilter filter = new IntentFilter();
        filter.addAction(ACTION);
        registerReceiver(br, filter);
    }

    public void onBtn1(View view){
        Intent intent = new Intent(this, MyReceiver.class);
        intent.putExtra("str","yez");
        intent.putExtra("num","122");
        sendBroadcast(intent);
    }

    public void onBtn2(View view){
        //private final String ACTION = "com.example.receivertest.MyAction";
        Intent intent = new Intent(ACTION);//intent filter에 해당이름으로 등록된 것을 깨우겠다.
        sendBroadcast(intent);
    }
}
 ```
 
 > [명시적 활성화] MyReceiver.java (new -> other-> Broadcast Receiver)
 ```java
 package com.example.receivertest;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class MyReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
        String str = intent.getStringExtra("str");
        int num = intent.getIntExtra("num",0);
        Toast.makeText(context,str+"/num:"+num,Toast.LENGTH_SHORT).show();
    }
} 
 ```

> [묵시적 활성화] MyReceiver2.java 
 ```java
 package com.example.receivertest;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class MyReceiver2 extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
        Toast.makeText(context,"리시버 묵시적 활성화",Toast.LENGTH_SHORT).show();
    }
}
 ```
> 결과화면 <br/>

![image](https://user-images.githubusercontent.com/62331803/90579892-54476b00-e202-11ea-9dbd-35330e237dd3.png)

<br/>

 ## Broadcast Receiver 활용예제2 | SMS 수신
 > 설명
 - manifest 파일에 RECEIVE_SMS permission 지정 <br/>
 
 ![image](https://user-images.githubusercontent.com/62331803/90580796-b1dcb700-e204-11ea-8cc2-d359ffea8e51.png)
 
 - intent filter 설정 <br/>

![image](https://user-images.githubusercontent.com/62331803/90580854-d46ed000-e204-11ea-80b9-773a2ade6c32.png)

 - 코드 <br/>
 
 ![image](https://user-images.githubusercontent.com/62331803/90580844-cf118580-e204-11ea-8146-15aad3cfdd0d.png)
 
  - 애뮬레이터에서 SMS 기능 사용 <br/>
 
 ![image](https://user-images.githubusercontent.com/62331803/90581329-ed2bb580-e205-11ea-9230-a1163b7dfdab.png)
 
 > SMSReceiver.java
 ```java
 package com.example.receivertest;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.os.Bundle;
import android.telephony.SmsMessage;
import android.widget.Toast;

public class SMSReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
        Bundle bundle = intent.getExtras();//intent에 저장된 메세지 정보를 반환. (원시 데이터)
        String msg;
        String sender;

        int i;
        if(bundle!=null){
            Object[] rawData = (Object[]) bundle.get("pdus");//pdus: 메세지를 추출하기 위한 key
            SmsMessage[] sms = new SmsMessage[rawData.length];
            for(i=0;i<rawData.length;i++){
                sms[i] = SmsMessage.createFromPdu((byte[])rawData[i]);//메세지 내용을 추출하여 sms에 저장
            }
            for(i=0;i<sms.length;i++){
                msg = sms[i].getMessageBody();//메세지 내용
                sender = sms[i].getOriginatingAddress();//메세지 송신인
                Toast.makeText(context,msg+" from "+sender, Toast.LENGTH_SHORT).show();
            }
        }

        throw new UnsupportedOperationException("Not yet implemented");
    }
}
 ```

 > manifest.xml
 - permission 추가
 ```xml
     <uses-permission android:name="android.permission.RECEIVE_SMS"/>
```
- SMSReicever intent filter 설정
 ```xml
<receiver
            android:name=".SMSReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="android.provider.Telephony.SMS_RECEIVED"/>
            </intent-filter>
        </receiver>
```

> MainActivity.java
```java
package com.example.receivertest;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.M)
        {
            // For device above MarshMallow
            boolean permission = getSMSPermission();
            if(permission) {
                Toast.makeText(this,"sms 권한 획득", Toast.LENGTH_SHORT).show();
            }
        }
        else{
            Toast.makeText(this,"권한 획득이 필수가 아닌 버전", Toast.LENGTH_SHORT).show();
        }
    }


    public boolean getSMSPermission(){
        boolean hasPermission = ((ContextCompat.checkSelfPermission(this,
                Manifest.permission.SEND_SMS) == PackageManager.PERMISSION_GRANTED) &&
                (ContextCompat.checkSelfPermission(this,
                        Manifest.permission.RECEIVE_SMS) == PackageManager.PERMISSION_GRANTED));
        if (!hasPermission) {
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.SEND_SMS,
                    Manifest.permission.RECEIVE_SMS}, 1);
        }
        return hasPermission;
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode)
        {
            case 1: {
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED)
                {
                    Toast.makeText(this, "sms 권한 부여", Toast.LENGTH_SHORT).show();
                }
            }
        }
    }

}
```

> 결과화면 <br/>

![image](https://user-images.githubusercontent.com/62331803/90582842-b9eb2580-e209-11ea-97a0-71db7d6f6d48.png)


 ## Broadcast Receiver 활용예제3 | SMS 발신
 
 > 설명 <br/>
 
 ![image](https://user-images.githubusercontent.com/62331803/90583837-26ffba80-e20c-11ea-864f-5b05dd0fc49e.png)

<br/>
 - 리시버 2개 필요 : 받는 쪽의 상태값 정보, 보내는 쪽의 상태값 정보 (정상 or 비정상)
<br/>

 > activity_s_m_s.xml <br/>
 
 ![image](https://user-images.githubusercontent.com/62331803/90584253-303d5700-e20d-11ea-909e-d1d7b16c2280.png)
 
 > SMSActivity.java *원래 리소스 파일 대문자 쓰면 안됨..
 ```java
 package com.example.receivertest;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.app.PendingIntent;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.telephony.SmsManager;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

public class SMSActivity extends AppCompatActivity {
    private EditText msg_et;
    private EditText phone_et;
    //각 리시버를 깨울 상수값
    private final static String SEND_ACTION = "SENT";
    private final static String DELIVER_ACTION = "DELIVERED";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_s_m_s);
        msg_et = findViewById(R.id.msg);
        phone_et = findViewById(R.id.phone);

        // intent filter 설정
        SMSSendResult send = new SMSSendResult();
        IntentFilter f1 = new IntentFilter();
        f1.addAction(SEND_ACTION);

        SMSReceiveResult recv = new SMSReceiveResult();
        IntentFilter f2 = new IntentFilter();
        f2.addAction(DELIVER_ACTION);

        registerReceiver(send,f1);
        registerReceiver(recv,f2);

        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.M)
        {
            // For device above MarshMallow
            boolean permission = getSMSPermission();
            if(permission) {
                Toast.makeText(this,"sms 권한 획득", Toast.LENGTH_SHORT).show();
            }
        }
        else{
            Toast.makeText(this,"권한 획득이 필수가 아닌 버전", Toast.LENGTH_SHORT).show();
        }
    }


    public boolean getSMSPermission(){
        boolean hasPermission = ((ContextCompat.checkSelfPermission(this,
                Manifest.permission.SEND_SMS) == PackageManager.PERMISSION_GRANTED) &&
                (ContextCompat.checkSelfPermission(this,
                        Manifest.permission.RECEIVE_SMS) == PackageManager.PERMISSION_GRANTED));
        if (!hasPermission) {
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.SEND_SMS,
                    Manifest.permission.RECEIVE_SMS}, 1);
        }
        return hasPermission;
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode)
        {
            case 1: {
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED)
                {
                    Toast.makeText(this, "sms 권한 부여", Toast.LENGTH_SHORT).show();
                }
            }
        }
    }

    public void onSend(View view){
        String msg = msg_et.getText().toString();
        String phone = phone_et.getText().toString();

        // 둘 중 하나라도 내용이 없다면, 내용입력 받게끔 설정
        if(msg.length() <=0 || phone.length() <=0){
            Toast.makeText(getApplicationContext(),
                    "input the message and phone number!",
                    Toast.LENGTH_SHORT).show();
            return;
        }

        //sms 보내는 쪽 상태확인을 위한 객체
        PendingIntent sendStat = PendingIntent.getBroadcast(this,0,
                new Intent("SENT"),0);// context, 확인 코드, 리시버를 깨울 인텐트, flag
        //sms 받는 쪽 상태확인을 위한 객체
        PendingIntent deliveredStat = PendingIntent.getBroadcast(this,0,
                new Intent("DELIVERED"),0);

        SmsManager sms = SmsManager.getDefault();

        //sms 전송
        sms.sendTextMessage(phone, null, msg, sendStat, deliveredStat);
    }
}
 ```
 
 > SMSSendResult.java  (broadcast receiver 1: 보내는 측의 상태)
```java
package com.example.receivertest;

import android.app.Activity;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.telephony.SmsManager;
import android.widget.Toast;

//문자메시지 보내는 쪽의 상태
public class SMSSendResult extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
        int result = getResultCode();
        switch (result){
            case Activity.RESULT_OK:
                Toast.makeText(context,"success", Toast.LENGTH_SHORT).show();
                break;
            case SmsManager.RESULT_ERROR_GENERIC_FAILURE:
                Toast.makeText(context,"generic failure", Toast.LENGTH_SHORT).show();
                break;
            case SmsManager.RESULT_ERROR_RADIO_OFF:
                Toast.makeText(context,"radio off", Toast.LENGTH_SHORT).show();
                break;
            case SmsManager.RESULT_ERROR_NULL_PDU:
                Toast.makeText(context,"null pdu", Toast.LENGTH_SHORT).show();
                break;
            case SmsManager.RESULT_ERROR_NO_SERVICE:
                Toast.makeText(context,"no service", Toast.LENGTH_SHORT).show();
                break;
        }
    }
}

```
 
 > SMSReceiveResult.java  (broadcast receiver 2: 받는 측의 상태)
```java
package com.example.receivertest;

import android.app.Activity;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

//문자메시지 받는 쪽의 상태
public class SMSReceiveResult extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
        int result = getResultCode();
        switch (result){
            case Activity.RESULT_OK:
                Toast.makeText(context,"receive : success", Toast.LENGTH_SHORT).show();
                break;
            case Activity.RESULT_CANCELED:
                Toast.makeText(context,"receive : fail", Toast.LENGTH_SHORT).show();
                break;
        }
    }
}

```

> manifest
- 메세지 발신 permisson 등록
```xml
    <uses-permission android:name="android.permission.SEND_SMS" />
```
- 자동생성된 receiver 지우기
왜? 묵시적으로 활성화하기 위해서 SMSActivity에서 receiver와 intent filter 직접 생성하여 사용할 것이기 때문.
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.receivertest">

    <uses-permission android:name="android.permission.RECEIVE_SMS" />
    <uses-permission android:name="android.permission.SEND_SMS" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        <activity android:name=".SMSActivity" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <receiver
            android:name=".SMSReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
            <action android:name="android.provider.Telephony.SMS_RECEIVED" />
            </intent-filter>
        </receiver>
        <receiver
            android:name=".MyReceiver2"
            android:enabled="true"
            android:exported="true" />
        <receiver
            android:name=".MyReceiver"
            android:enabled="true"
            android:exported="true" />

        <activity android:name=".MainActivity">

        </activity>
    </application>

</manifest>
```

> 실행 <br/>

![image](https://user-images.githubusercontent.com/62331803/90587878-62eb4d80-e215-11ea-87ff-f0fc0a66aa7b.png)

> 결과화면<br/>

![image](https://user-images.githubusercontent.com/62331803/90588223-13f1e800-e216-11ea-9288-0ece9f8563e8.png)

![image](https://user-images.githubusercontent.com/62331803/90588254-2409c780-e216-11ea-814a-d7749ca7b67f.png)

<br/>
<br/>

## AlertDialog | 알림 박스

> 설명 <br/>

![image](https://user-images.githubusercontent.com/62331803/90588307-44398680-e216-11ea-87a2-f537d7f3024e.png)
![image](https://user-images.githubusercontent.com/62331803/90588320-4ac7fe00-e216-11ea-964e-082dcce14242.png)

> SMSActivity.java
```java
package com.example.receivertest;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.app.Activity;
import android.app.AlertDialog;
import android.app.PendingIntent;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.telephony.SmsManager;
import android.telephony.SmsMessage;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

public class SMSActivity extends AppCompatActivity {
    private EditText msg_et;
    private EditText phone_et;
    //각 리시버를 깨울 상수값
    private final static String SEND_ACTION = "SENT";
    private final static String DELIVER_ACTION = "DELIVERED";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_s_m_s);
        msg_et = findViewById(R.id.msg);
        phone_et = findViewById(R.id.phone);
        
        /*
        <새로운 브로드캐스트 리시버 생성해서 사용하기>
        alert dialog는 activity의 자식이기 때문에
        activity 생성이후에 만들어져야 함.
        따라서 따로 receiver를 만들지 말고
        사용할 activity에서 직접 broadcast receiver 객체 생성해서 쓸 수잇도록 해야 한다.
        */
        
        BroadcastReceiver smsRecv = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                // an Intent broadcast.
                Bundle bundle = intent.getExtras();
                String msg;
                String sender;

                int i;
                if (bundle != null) {
                    Object[] rawData = (Object[]) bundle.get("pdus");
                    SmsMessage[] sms = new SmsMessage[rawData.length];

                    for (i = 0; i < rawData.length; i++) {
                        sms[i] = SmsMessage.createFromPdu((byte[]) rawData[i]);
                    }

                    for (i = 0; i < sms.length; i++) {
                        msg = sms[i].getMessageBody();
                        sender = sms[i].getOriginatingAddress();
                        AlertDialog.Builder builder = new AlertDialog.Builder(SMSActivity.this);
                        builder.setMessage(msg + "\nfrom: " + sender)
                                .setPositiveButton("OK", new DialogInterface.OnClickListener() {
                                    @Override
                                    public void onClick(DialogInterface dialogInterface, int i) {
                                        dialogInterface.cancel();//다이얼로그 창 닫음
                                        //finish(): 액티비티 종료
                                    }
                                })
                                .setTitle("You've got Message!");
                        AlertDialog alertDialog = builder.create();
                        alertDialog.show();
                    }
                }
            }
        };
        IntentFilter f3 = new IntentFilter();
        f3.addAction("android.provider.Telephony.SMS_RECEIVED");//인텐트 필터 생성.
        registerReceiver(smsRecv,f3);//브로드캐스트 리시버에 해당 필터 등록.

        // intent filter 설정
        SMSSendResult send = new SMSSendResult();
        IntentFilter f1 = new IntentFilter();
        f1.addAction(SEND_ACTION);

        SMSReceiveResult recv = new SMSReceiveResult();
        IntentFilter f2 = new IntentFilter();
        f2.addAction(DELIVER_ACTION);

        registerReceiver(send,f1);
        registerReceiver(recv,f2);

        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.M)
        {
            // For device above MarshMallow
            boolean permission = getSMSPermission();
            if(permission) {
                Toast.makeText(this,"sms 권한 획득", Toast.LENGTH_SHORT).show();
            }
        }
        else{
            Toast.makeText(this,"권한 획득이 필수가 아닌 버전", Toast.LENGTH_SHORT).show();
        }
    }


    public boolean getSMSPermission(){
        boolean hasPermission = ((ContextCompat.checkSelfPermission(this,
                Manifest.permission.SEND_SMS) == PackageManager.PERMISSION_GRANTED) &&
                (ContextCompat.checkSelfPermission(this,
                        Manifest.permission.RECEIVE_SMS) == PackageManager.PERMISSION_GRANTED));
        if (!hasPermission) {
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.SEND_SMS,
                    Manifest.permission.RECEIVE_SMS}, 1);
        }
        return hasPermission;
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode)
        {
            case 1: {
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED)
                {
                    Toast.makeText(this, "sms 권한 부여", Toast.LENGTH_SHORT).show();
                }
            }
        }
    }

    public void onSend(View view){
        String msg = msg_et.getText().toString();
        String phone = phone_et.getText().toString();

        // 둘 중 하나라도 내용이 없다면, 내용입력 받게끔 설정
        if(msg.length() <=0 || phone.length() <=0){
            Toast.makeText(getApplicationContext(),
                    "input the message and phone number!",
                    Toast.LENGTH_SHORT).show();
            return;
        }

        //sms 보내는 쪽 상태확인을 위한 객체
        PendingIntent sendStat = PendingIntent.getBroadcast(this,0,
                new Intent("SENT"),0);// context, 확인 코드, 리시버를 깨울 인텐트, flag
        //sms 받는 쪽 상태확인을 위한 객체
        PendingIntent deliveredStat = PendingIntent.getBroadcast(this,0,
                new Intent("DELIVERED"),0);

        SmsManager sms = SmsManager.getDefault();

        //sms 전송
        sms.sendTextMessage(phone, null, msg, sendStat, deliveredStat);
    }
}
```

> 결과화면 <br/>
![image](https://user-images.githubusercontent.com/62331803/90589105-443a8600-e218-11ea-85ef-2d6d7d6f45a6.png)

<br/><br/>

## 연습문제
- smsactiity에 arraylist 만들어서, 받은 문자들이 리스트에 저장되서 뜨게끔 (listview 형태)
- 메세지 담을 vo 만들어서 
