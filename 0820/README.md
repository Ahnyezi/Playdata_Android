
20200820 (목) 
### 수업 목차
#### 1. 연습문제 | 전화번호부에 메세지 기능 추가
[1-a.  Receiver Test 프로젝트 (메세지 기능)](#메세지-기능) <br/>
[1-b. PhoneBook 프로젝트 (전화번호부 기능)](#전화번호부-기능) <br/>
#### 2. DataStorage | 안드로이드 데이터 영구저장

[2-a. Preferences](#DataStorage-매커니즘-1) <br/>
[2-b. files](#DataStorage-매커니즘-2) <br/>
[2-c. dataBases](#DataStorage-매커니즘-3) <br/>

<br/>
<br/>


## 1. 연습문제 | 전화번호부에 메세지 기능 추가
### 메세지 기능

#### <Receiver Test 프로젝트>
> manifest.xml
- 수정된 부분 | 묵시적 호출을 위한 intent filter 추가

```xml
        <activity android:name=".SMSActivity" >
            <intent-filter>
                <action android:name="com.example.receivertest.sms_send"></action>
            </intent-filter>
        </activity>
```  
- 묵시적으로 활성화할 컴포넌트 설정.<br/>
        이 이름으로 지정된 인텐트 필터가 날아온다면 해당화면으로 넘어오게 조건을 지정한다.
- 패키지 이름까지 안넣어줘도 상관 없음.<br/>
하지만 중복 방지를 위해서 안전하게 넣어줌

> SMSActivity.java 

- 수정된 부분
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_s_m_s);
        msg_et = findViewById(R.id.msg);
        phone_et = findViewById(R.id.phone);

        Intent intent = getIntent();
        String tel = intent.getStringExtra("tel");
        if(tel!=null && ! tel.equals("")){
            phone_et.setText(tel);
        }
    }
```
<details>
<summary> SMSActivity.java 전체코드 </summary>
        
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

        Intent intent = getIntent();
        String tel = intent.getStringExtra("tel");
        if(tel!=null && ! tel.equals("")){
            phone_et.setText(tel);
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
        setResult(RESULT_OK);//보낼 데이터가 없다면 intent 보내지 않음.
        finish();//현재 activity 종료.
    }
}
```

</details>


### 전화번호부 기능
#### <PhoneBook 프로젝트>
> phoneAdapter.java
- sms 버튼을 누르면 동작할 이벤트 함수 추가
- 추가된 부분
```java
sms.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    //다른 액티비티에 있는 프로그램을 호출하기 위해,
                    //해당 프로그램의 이름과 해당 액티비티 클래스이름을 알려줘야 함.
                    ComponentName cn = new ComponentName("com.example.receivertest","com.example.receivertest.SMSActivity");
                    Intent intent = new Intent("com.example.receivertest.sms_send");//action 이름 설정
                    intent.putExtra("tel",m.getTel());
                    intent.setComponent(cn);
                    context.startActivity(intent);
                }
            });
```

<details>
<summary> phoneAdapter.java 전체코드 </summary>

```java
package com.example.phonebook;

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.net.Uri;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;

import com.example.phonebook.model.Member;

import java.util.ArrayList;
import java.util.List;

public class phoneAdapter extends ArrayAdapter<Member> {
    private Context context;
    private ArrayList<Member> list;
    private int resId;

    public phoneAdapter(@NonNull Context context, int resource, @NonNull List<Member> objects) {
        super(context, resource, objects);
        this.context = context;
        this.list = (ArrayList<Member>) objects;
        resId = resource;
    }

    @NonNull
    @Override
    //getview: 어댑터에서 가장 중요한 작업! 데이터 위치(position)를 읽어와서 리소스로 지정한 뷰로 생성하여 반환
    //list의 요소개수만큼 호출되는 함수
    public View getView(int position, @Nullable View convertView, @NonNull ViewGroup parent) {
        View itemView = convertView;
        if(itemView==null){            //뷰가 없다면 생성
            LayoutInflater vi = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);//LayoutInflater: xml등의 리소스를 실제 뷰로 부풀리는 역할을 하는 객체
            itemView = vi.inflate(resId,null);//설계한 자료의 id를 받아서 해당 id로 실제 뷰를 생성.(item_layout.xml)
        }
        final Member m = list.get(position);//현재 위치의 멤버 객체 추출
        if(m != null) {
            TextView t1 = itemView.findViewById(R.id.textView);//이름 tv
            TextView t2 = itemView.findViewById(R.id.textView2);//전화번호 tv
            ImageView imgv = itemView.findViewById(R.id.imageView);//이미지
//            imgv.setImageResource(R.drawable.ic_launcher_foreground);//임시로 고정값 사용.

            Button sms = itemView.findViewById(R.id.button17);
            Button call = itemView.findViewById(R.id.button19);

            //뷰에 텍스트 세팅
            if(t1!=null){
                t1.setText(m.getName());
            }
            if(t2!=null){
                t2.setText(m.getTel());
            }
            if(imgv!=null){
                imgv.setImageResource(m.getImgRes());
            }

            call.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    //action_call: 시스템에서 제공하는 것이기 때문에 permission 설정 필요 => manifestAndroid
                    Intent intent = new Intent(Intent.ACTION_DIAL, Uri.parse("tel:"+m.getTel()));//묵시적으로 activity 활성화
                    context.startActivity(intent);
                }
            });

            sms.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    //다른 액티비티에 있는 프로그램을 호출하기 위해,
                    //해당 프로그램의 이름과 해당 액티비티 클래스이름을 알려줘야 함.
                    ComponentName cn = new ComponentName("com.example.receivertest","com.example.receivertest.SMSActivity");
                    Intent intent = new Intent("com.example.receivertest.sms_send");//action 이름 설정
                    intent.putExtra("tel",m.getTel());
                    context.startActivity(intent);
                }
            });
        }
        return itemView;
    }
}

```

</details>

<br/>

## 2. DataStorage | 데이터 영구저장 p.168  (Storage test 프로젝트)

### DataStorage :: 안드로이드 플랫폼에서 데이터를 저장하기
- 안드로이드 플랫폼은 Data를 저장하는 방법으로 환경설정(이하 Preferences), 파일, Local DB, 네트워크를 제공한다. <br/>
- 안드로이드에서 어플리케이션 안의 모든 데이터와 파일은 해당 어플리케이션만 사용가능하다
- Content Provide(메인 컴포넌트)를 통하여 해당 어플리케이션의 데이터를 다른 어플리케이션에서 읽거나 쓰게 할 수 있다.
- 데이터를 저장하고 회수하기 위해서, 다음의 메커니즘을 사용한다.
(1)Preferences (2)Files (3)Databases

## DataStorage 매커니즘 1 
### 2-a.  Preferences 
> preferences 파일
- Preferences는 DATA 저장 매커니즘 중 가장 간단하게 정보를 저장하는 방법(mechanism)을 제공
- App이나 그 컴포넌트 (Activity, Service 등)의 환경 설정 정보를 저장/복원하는 용도로 사용

> preferences의 형태
- 안드로이드에서 Preferences는 ListView의 형태로 표현되며 쉬운 Preferences의 구현을 위해 PreferenceActivity 클래스를 제공한다.
- PreferenceActivity 클래스는 XML 기반의 Preference 정의 문서를 통해 App 파일 저장소에 Preferences 파일을 생성하고 사용하는 방식으로 작동한다.
- 데이터 저장 시 사용되는 자료구조는 Map 방식이다. 즉 키(key)-값(value) 한 쌍으로 이루어진 1~n개의 항목으로 구성된다. 

> preferences API
-  주요메서드

```java
public static final String PREFERENCE_FILENAME = "AppPrefs";
//한 응용 프로그램에서 공유
SharedPreferences settings = getSharedPreferences(PREFERENCE_FILENAME, 0); 
//전용
SharedPreferences settingsActivity = getPreferences(MODE_PRIVATE); 

SharedPreferences.contains()		//주어진 이름의 항목이 존재하는가
SharedPreferences.edit()		//편집기 객체 얻음
SharedPreferences.getAll()		//모든 항목 담은 Map 객체 얻음
SharedPreferences.getBoolean()	//주어진 이름의 부울 값 얻어 옴
SharedPreferences.getFloat()
SharedPreferences.getInt()
SharedPreferences.getLong()
SharedPreferences.getString() // application settings 얻어와 editor open
SharedPreferences settings = getPreferences(0);
 SharedPreferences.Editor prefEditor = settings.edit();

prefEditor.clear();		// 모든 항목 제거
prefEditor.remove(key); 	//주어진 이름 항목 제거
prefEditor.putBoolean(key, bool);	//부울 값 설정
prefEditor.putFloat(key, float);
prefEditor.putInt();
prefEditor.putLong();
prefEditor.putString();
prefEditor.commit();	//모든 변경 적용
```

<br/>

> preferences 파일 위치

/data/data/<응용패키지이름>/shared_prefs/<선호설정파일명>.xml

## 실습1 | Preferences 파일 다루기

> activity_main.xml <br/>

![image](https://user-images.githubusercontent.com/62331803/90710616-b7f09780-e2d9-11ea-8aca-02f2d5d239fa.png)

> MainActivity.java
<details>
<summary> 코드보기 </summary>        
        
```java
package com.example.storagetest;

import androidx.appcompat.app.AppCompatActivity;

import android.content.SharedPreferences;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

import java.util.prefs.Preferences;

public class MainActivity extends AppCompatActivity {
    private String id;
    private EditText id_et;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        id_et = findViewById(R.id.id_et);

        if(id==null){
            SharedPreferences pref = getSharedPreferences("idcheck",MODE_PRIVATE);
            id = pref.getString("id",null);//key, default value
            if(id == null){
                Toast.makeText(this,"id를 등록하시오.",Toast.LENGTH_SHORT).show();
            }else{
                Toast.makeText(this,id+"로 로그인된 상태입니다.",Toast.LENGTH_SHORT).show();
            }
        }
    }

    public void onBtn1(View view){
        String i = id_et.getText().toString();
        SharedPreferences pref = getSharedPreferences("idcheck",MODE_PRIVATE);
        SharedPreferences.Editor editor = pref.edit();
        editor.putString("id",i);
        editor.commit();//id라는 key로 preferences 파일에 저장.
        id_et.setText("");
    }

    public void onBtn2(View view){
        SharedPreferences pref = getSharedPreferences("idcheck",MODE_PRIVATE);
        SharedPreferences.Editor editor = pref.edit();
        editor.remove("id");
        editor.commit();
    }
}
```

</details>

> 결과화면

<details>
<summary> 보기 </summary>

- 첫 화면 <br/>

![image](https://user-images.githubusercontent.com/62331803/90710128-aa86dd80-e2d8-11ea-9318-be7eee8d42e2.png)

- 프로그램 재실행  <br/>

![image](https://user-images.githubusercontent.com/62331803/90710027-7d3a2f80-e2d8-11ea-8e82-ff3e110788a5.png)

- 생성한 preferences 파일 위치  <br/>

![image](https://user-images.githubusercontent.com/62331803/90710295-06516680-e2d9-11ea-965c-b02575b00cbe.png)

- idcheck.xml 파일  <br/>

![image](https://user-images.githubusercontent.com/62331803/90710335-2123db00-e2d9-11ea-9398-93fb1cbf2f08.png)

</details>

<br/>

## DataStorage 매커니즘 2
###  2-b. files

> files 저장위치

/data/data/<응용패키지명>/files/파일명

> files API

- 주요 메서드
```java
Context.openFileInput()		//하위 디렉터리 /files의 파일 읽기 모드로 염
Context.openFileOutput()		//쓰기 모드로 오픈
Context.deleteFile()		//파일 삭제
Context.fileList()		//파일목록
Context.getFilesDir()		// /files에 대한 객체
Context.getCacheDir()		// /cache에 대한 객체
Context.getDir()		// 주어진 이름의 응용 프로그램 하위 디렉터리를 얻거나 생성

```

## 실습2 | 파일 입출력

> activity_main2.xml <br/>

![image](https://user-images.githubusercontent.com/62331803/90712765-b6759e00-e2de-11ea-9016-644f5a782783.png)

> MainActivity2.java <br/>

: 사용자에게 문자열을 입력받아 파일에 쓰고, 해당 파일을 읽어와 UI에 출력
<details>
<summary> 코드보기 </summary>

```java
package com.example.storagetest;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.view.View;
import android.widget.EditText;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Arrays;

public class MainActivity2 extends AppCompatActivity {
    private EditText content;
    private EditText content2;
    private FileOutputStream fo;
    private FileInputStream fi;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main2);
        content = findViewById(R.id.et1);
        content2 = findViewById(R.id.et2);
    }

    // 1. 파일 쓰기
    public void onSave(View view){
        try {
            /*
            <파일 저장 경로>
            android는 보안을 위해서
            데이터/파일을 공유하지 않음.
            현재 프로젝트에서만 사용할 수있게
            해당 프로젝트 내부에 모든 데이터/파일 저장
            */
            fo = openFileOutput("file.txt",MODE_APPEND);//파일명, 오픈 모드(이어쓰기)
            fo.write((content.getText().toString()+"\n").getBytes());//현재패키지에 files라는 디렉토리를 만들어서 저장.
            System.out.println("저장 완료");
            fo.close();
            content.setText("");
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 2. 파일 읽기
    public void onRead(View view){
        StringBuilder sb = new StringBuilder();
        byte[] buf = new byte[40];
        try {
            fi = openFileInput("file.txt");
            while((fi.read(buf,0,40))!=-1){
                String str = new String(buf);
                sb.append(str);
                if(fi.available()<40){ //이전에 쓴 쓰레기값 제거 //available(): 잔량 체크
                    Arrays.fill(buf,0,40,(byte)' ');
                }
                fi.close();
                content2.setText(sb.toString());
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```

</details>

> 결과화면

파일내용 출력 <br/>

![image](https://user-images.githubusercontent.com/62331803/90712344-c640b280-e2dd-11ea-8976-e7dcf38da35c.png)

file.txt 확인 <br/>

![image](https://user-images.githubusercontent.com/62331803/90712374-d5276500-e2dd-11ea-81e5-4ddf9d5ee831.png)

<br/>


## DataStorage 매커니즘 3
### 2-c. DataBase

> DB 처리방법
1. db 파일 생성. 
있으면 open, 없으면 새로 생성 후 open
return 값: db파일에 작업할 수 있는 SQLiteDatabase 객체
```java
SQLiteDatabase mDatabase=openOrCreateDatabase("my_sqlite_database.db”, SQLiteDatabase.CREATE_IF_NECESSARY,null);
```
2. 생성된 위치 

/data/data/<응용패키지명>/databases/<데이터베이스명>

3. 테이블 생성
```java
private static final String DATABASE_CREATE = "create table " + 
    DATABASE_TABLE + " (" + KEY_ID + 
    " integer primary key autoincrement, " +
    KEY_NAME + " text not null);";
mDatabase.execSQL(DATABASE_CREATE);
```

> 레코드 처리방법
- 레코드 삽입
```java
public long insertEntry(MyObject _myObject) {
    ContentValues contentValues = new ContentValues();//key(컬럼명), value를 쌍으로 저장하는 클래스
   contentValues.put(“name”,”aaa”);//데이터 셋팅
   return db.insert(DATABASE_TABLE, null, contentValues);// insert 작업
     //return value: 방금 추가한 line의 _id 값
  } 
  
```
- 레코드 삭제
```java
public boolean removeEntry(long _rowIndex) {
    return db.delete(DATABASE_TABLE, KEY_ID + "=" + _rowIndex, null) > 0; //테이블 이름, where절, '?' 쓸 경우 matching할 값 
    // return value: 처리 성공 유무 bool
  }
```

> 레코드 검색
```java
public Cursor getAllEntries () {
    return db.query(DATABASE_TABLE, new String[] {KEY_ID, KEY_NAME}, null, null, null, null, null);
  }
```
param1 : 테이블명 <br/>
param2 : 결과에 포함할 열 목록 <br/>
param3 : where절 <br/>
param4 : where절의 ‘?’ 치환 <br/>
parma5 : group by절 <br/>
param6 : having 절 <br/>
param7 : order by 절 <br/>

> 레코드 업데이트
```java
public int updateEntry(long _rowIndex, MyObject _myObject) {
    String where = KEY_ID + " = " + _rowIndex;
    ContentValues contentValues = new ContentValues();
	return db.update(DATABASE_TABLE, contentValues, where, null); //테이블명, update할 값, where절, ? mapping
  }
```
> DB 관련 상세 API
1. SQLiteDatabase
- 기본형태
```java
openOrCreateDatabase (String path, SQLiteDatabase.CursorFactory factory)
// return value : SQLiteDatabase
```

-  sql문 실행
```java
execSQL(String sql)
//Execute a single SQL statement that is not a query.
```

-  DB 닫기
```java
close() 
```

-  query
```java
query(String table, String[] columns, String selection, String[] selectionArgs, String groupBy, String having, String orderBy)
// Query the given table, returning a Cursor over the result set
// return value: cursor (==resultset)
```
-  insert, delete, update
```java
insert(String table, String nullColumnHack, ContentValues values)//Convenience method for inserting a row into the database.

delete(String table, String whereClause, String[] whereArgs) //Convenience method for deleting rows in the database.

update(String table, ContentValues values, String whereClause, String[] whereArgs) // Convenience method for updating rows in the database.
```

2.  SQLiteOpenHelper: DB오픈을 도와주는 클래스(초기화 작업) <br/>
- public void onCreate(SQLiteDatabase _db) {...	  } <br/>
a. Db 파일 생성시 단 한번 실행. <br/>
b. table 생성 코드 <br/>

- public void onUpgrade(SQLiteDatabase _db, int _oldVersion, int _newVersion) {} <br/>
a. Called when the database needs to be upgraded <br/>
b. Db 파일과 data open할 때 설정한 버전이 다를 경우 실행 (버전이 일치하지 않을 경우) <br/>
c. 버전 downgrade 불가 <br/>

- public synchronized SQLiteDatabase getReadableDatabase () <br/>
a. Create and/or open a database <br/>
b. be opened read-only <br/>
c. 읽기 모드 <br/>

- public synchronized SQLiteDatabase getWritableDatabase ()  <br/>
a. Create and/or open a database <br/>
b. can write to the database <br/>
c. 읽고 쓰기 둘다 가능한 모드 <br/>


## 실습3 | databases
> MYDBAdapter.java

<details>
<summary> 코드보기 </summary>

```java
package com.example.usr.app11_4;  
  
import android.content.ContentValues;  
import android.content.Context;  
import android.database.Cursor;  
import android.database.sqlite.SQLiteDatabase;  
import android.database.sqlite.SQLiteException;  
import android.database.sqlite.SQLiteOpenHelper;  
  
public class MyDBAdapter {  
    private static final String DB = "MyDB.db"; // DB 파일명  
  private static final String DB_TABLE = "MyTable"; // 테이블명  
  private static final String ID = "_id"; // 컬럼명  
  private static final String NAME = "name"; // 컬럼명  
  private static final int DB_VERS = 1; // DB 파일 버전  
  
  private SQLiteDatabase mdb;//db 파일 처리 객체(가장 중요)  
  private fina,l Context context;  
 private MyHelper mHelper;  
  
 public MyDBAdapter(Context context) {  
        this.context = context;  
  //helper 객체 생성  
  mHelper = new MyHelper(context, DB, null, DB_VERS);  
  //설정한 버전이 맞지 않으면 onUpgrade 호출된다.  
  }  
  
    //helper 객체로 db파일 오픈  
  public void open() throws SQLiteException {  
        try {  
            mdb = mHelper.getWritableDatabase();//읽고 쓰기 모드로 오픈  
  } catch (SQLiteException ex) {  
            mdb = mHelper.getReadableDatabase();//문제 있으면 읽기 전용으로  
  }  
    }  
  
    //db 파일 닫기  
  public void close() {  
  
        mdb.close();  
  }  
  
    // insert  
  public long insertData(String name) {  
  
        ContentValues cv = new ContentValues();  
  
  cv.put(NAME, name);//insert할 컬럼명, 값  
  
  return mdb.insert(DB_TABLE, null, cv);  
  }  
  
    // delete  
  public int removeData(long index) {  
  
        return mdb.delete(DB_TABLE, ID + "=" + index, null);  
  }  
  
    // update  
  public int updateData(long index, String name) {  
  
        String where = ID + " = " + index;  
  
  ContentValues cv = new ContentValues();  
  
  cv.put("name", name);  
  
 return mdb.update(DB_TABLE, cv, where, null);  
  }  
  
    // 전체 검색  
  public Cursor getAll() {  
        return mdb.query(DB_TABLE, new String[] { ID, NAME}, null, null, null, null, null);  
  }  
  
    private static class MyHelper extends SQLiteOpenHelper {  
  
        private static final String DB_CREATE = 
	"create table " + DB_TABLE + " (" + ID +  " integer primary key autoincrement, " + NAME + " text not null );";
	// <oracle 버전>
	// create table MyTable (
	// _id integer primary key autoincrement,
	// name text not null);
	
  
  public MyHelper(Context context, String name,  
  SQLiteDatabase.CursorFactory factory, int version) {  
  
            super(context, name, factory, version);  
  }  
  
        // 테이블 생성  
  @Override  
        public void onCreate(SQLiteDatabase db) {  
            db.execSQL(DB_CREATE);  
  }  
  
        // 버전이 맞지 않을 때 호출됨  
  @Override  
        public void onUpgrade(SQLiteDatabase db, int oVers, int nVers) {  
            db.execSQL("DROP TABLE IF EXISTS " + DB_TABLE);//현재 테이블을 삭제  
  onCreate(db);//새로 생성  
  }  
    }  
  
}
```
</details>
