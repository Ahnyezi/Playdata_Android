# 안드로이드 | Activity (view)

# 1. Spinner:: combo box와 동일한 역할

### 예제코드
```java
//drawable 디렉토리에 img1~img6 삽입.
package com.example.myapplication;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.ImageView;
import android.widget.Spinner;
import android.widget.Toast;

import java.util.ArrayList;

public class MainActivity4 extends AppCompatActivity {
    private Spinner spinner;
    private ArrayList<String> list;
    private ArrayList<Integer> list2;
    private ArrayAdapter<String> aa;
    private ImageView imageview;

    public MainActivity4() {
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main4);
        spinner = findViewById(R.id.spinner);
        imageview = findViewById(R.id.imageView);

        list2 = new ArrayList<>();
        list2.add(R.drawable.img1);
        list2.add(R.drawable.img2);
        list2.add(R.drawable.img3);
        list2.add(R.drawable.img4);
        list2.add(R.drawable.img5);
        list2.add(R.drawable.img6);

        list = new ArrayList<>();
        list.add("img1");
        list.add("img2");
        list.add("img3");
        list.add("img4");
        list.add("img5");
        list.add("img6");

        //현재 view가 있는 context, spinner layout, 각 view 삽입할 데이터
        aa = new ArrayAdapter<>(this, android.R.layout.simple_spinner_item, list);
        spinner.setAdapter(aa);
        spinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> adapterView, View view, int i, long l) {
                imageview.setImageResource(list2.get(i));
            }

            @Override
            public void onNothingSelected(AdapterView<?> adapterView) {
                Toast.makeText(MainActivity4.this, "선택된 항목 없음", Toast.LENGTH_SHORT).show();
                //이벤트 호출 시에는 activity명.this 사용하는 것이 안전
            }
        });

    }
}
```
- 결과 화면
![image](https://user-images.githubusercontent.com/62331803/90200872-36e65b80-de14-11ea-843e-e1046c2cffae.png)

# 2. activity 이동
### intent 호출을 통해 다른 activity를 활성화 시킨다.
- 명시적 방법(같은 program에 있는 activity를 실행): 활성화할 클래스 이름을 직접 담는다.
```java
Intent intent = new Intent(this, MainActivity5.class);//명시적으로 activity 활성화
```
- 묵시적 방법(다른 program에 있는 activity를 실행): action, data, category 등의 정보로 지목을 한다.<br/>
```java
Intent intent = new Intent(Intent.ACTION_DIAL, Uri.parse("tel:"+m.getTel()));//묵시적으로 activity 활성화
```

### Intents란?
 ![image](https://user-images.githubusercontent.com/62331803/90203697-5d0ff980-de1c-11ea-9492-9c29d863c5e2.png)
<br/>
 
- 역할: 메인 컴포넌트를 활성화시키는 역할
- 정보: 이동할 컴포넌트 정보, 컴포넌트에 전달하는 정보: 데이터와 함께 이동함. (기본 타입, 클래스 타입 *직렬화등 필요)
- intent객체를 생성해서 사용할 경우, 동작을 나타내는 문자열을 정의해서 사용 
**tip: 묵시적으로 활성화 할경우, 패키지 이름이 이름이 참조된다. 이름 설정 주의.

### intent 이동방법
![image](https://user-images.githubusercontent.com/62331803/90206399-79149a80-de1e-11ea-92a8-692d310ff4f0.png)<br/>
 - startActivity(intent): 이동한 곳에서 멈춰있음
 - startActivityForResult(intent): 이동한 뒤에 자동으로 되돌아 옴
 
 ### 기본 intent 객체들. 
 ![image](https://user-images.githubusercontent.com/62331803/90201757-c4c34600-de16-11ea-9ccd-686166e00c91.png)
 
 ### 동작을 하는데 필요한 데이터
 ![image](https://user-images.githubusercontent.com/62331803/90206107-d4925880-de1d-11ea-9b8d-ccd49f40e323.png)<br/>
 URI(자원의 위치)
 ex. action edit에서 필요한 데이터: 수정할 데이터
 action call에서 필요한 데이터: 데이터 tel 위치
 
  ### 카테고리 지정
  ![image](https://user-images.githubusercontent.com/62331803/90206124-dd832a00-de1d-11ea-9681-1c4289d53397.png)<br/>
 카테고리? 컴포넌트를 더 쪼갠다.
 
 ###  extra 정보
 ![image](https://user-images.githubusercontent.com/62331803/90206140-e7a52880-de1d-11ea-8233-080693d18ceb.png)<br/>
 key이름, value값으로 담음
 intent.putextra(key,value)
 
 ### 받아온 인텐트를 확인
  ![image](https://user-images.githubusercontent.com/62331803/90206155-f7247180-de1d-11ea-85e7-5372fdfd9b8b.png)<br/>
- 명시적: 이름을 확인
- 묵시적: intent filter(manifest 파일에 기재된 조건)와 해당 intent가 가져온 정보를 비교
 
 ##### 샘플코드 
```java
//startActivity()로 활성화
Intent intent = new Intent(getApplicationContext(), MyOtherActivity.class);//인텐트 객체 생성
startActivity(intent);//activity 활성화(인텐트 전송)
```

```java
//startActivityForResult()로 활성화
Intent intent = new Intent(getApplicationContext(), Other2.class);

startActivityForResult(intent, SHOW_SUBACTIVITY);//activity 활성화.
//*SHOW_SUBACITIVITY(요청코드) :어떤 activity를 방문했느냐에 따라  반환되는 내용이 달라짐.

public void onActivityResult(int requestCode, int resultCode, Intent data) { 
//intent 되돌아오면 자동으로 호출됨. 추후 처리할 내용을 담음. 
//*requestcode(요청코드에 대한 응답), resultcode(정상처리 여부), data(처리한 결과 데이터)
}
```
```java
//상대 activity 소스코드
Intent result = getIntent(); // getIntent(): activity 클래스의 메소드. 
                             //activity를 활성화 하기 위해 상대방이 전송한 intent를 획득.
result.getExtra(~)
result.putExtra(IS_INPUT_CORRECT, inputCorrect);//돌아가기 전에 현재 처리 결과를 담음
result.putExtra(SELECTED_PISTOL, selectedPistol);
setResult(RESULT_OK, result);    
finish();//현재 activity 종료. 이전 activity로 돌아가라는 명령
```

##### 실습 예제 : 버튼 클릭으로 정보확인 페이지(다른 activity)로 이동
- MainActivity4.java 에 추가
```java
 public void onGoActivity(View view){
        Intent intent = new Intent(this, MainActivity5.class);//명시적으로 activity 활성화
        intent.putExtra("msg","hello android");
        intent.putExtra("number",95);
        Member m = new Member("yeji","0100000000",R.drawable.img1,1);
        intent.putExtra("m",m);
        startActivity(intent);
    }
```
- activity_main5.xml
![image](https://user-images.githubusercontent.com/62331803/90208836-c3007f00-de24-11ea-8057-b01df28cf08a.png)
- 정보확인 view MainActivity5.java 생성
```java
package com.example.myapplication;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Intent;
import android.os.Bundle;
import android.widget.TextView;

import com.example.myapplication.model.Member;

public class MainActivity5 extends AppCompatActivity {
    private TextView msg;
    private TextView num;
    private TextView member;
    private Intent intent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main5);
        msg = findViewById(R.id.textView8);
        num = findViewById(R.id.textView7);
        member = findViewById(R.id.textView6);

        intent = getIntent();
        String m = intent.getStringExtra("msg");
        int n = intent.getIntExtra("number",0);
        Member mm = (Member) intent.getSerializableExtra("m");

        msg.setText(m);
        num.setText(n+"");
        member.setText(mm.toString());
    }
}
```
- 결과 화면
![image](https://user-images.githubusercontent.com/62331803/90206937-f391ea00-de1f-11ea-99c4-b93292ac099c.png)<br/>

### 실습예제 | 수정 페이지
- activity_main6.xml
![image](https://user-images.githubusercontent.com/62331803/90208348-9bf57d80-de23-11ea-8e2a-58fdeb397fd7.png)<br/>
- MainActivity6.java
```java
package com.example.myapplication;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;

import com.example.myapplication.model.Member;

public class MainActivity6 extends AppCompatActivity {
    private EditText name_et;
    private EditText tel_et;
    private Intent intent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main6);
        name_et = findViewById(R.id.et1);
        tel_et = findViewById(R.id.et2);

        intent = getIntent();
        Member m = (Member) intent.getSerializableExtra("m");
        name_et.setText(m.getName());
        tel_et.setText(m.getTel());
    }

    public void onGoBack(View view){
        Member m = new Member();
        m.setName(name_et.getText().toString());
        m.setTel(tel_et.getText().toString());
        intent.putExtra("m",m);
        setResult(RESULT_OK,intent);//resultCode, data
        finish();
    }
}
```
- MainActivity4.java 에 추가
```java
public void onGoActivity6(View view){
        Intent intent = new Intent(this,MainActivity6.class);//명시적 activity 활성화 방법.
        Member m = new Member("MING","0104888888",R.drawable.img1,1);
        intent.putExtra("m",m);
        startActivityForResult(intent,1);//request code로 방문하는 activity 종류를 구분
    }

    // startActivityForResult()를 사용하면 자동으로 호출되는 함수.
    // : 처리결과를 현재 activity에 적용
    public void onActivityResult(int requestCode, int resultCode, @Nullable  Intent data){
        super.onActivityResult(requestCode,resultCode,data);
        if(resultCode==RESULT_OK){
            switch(requestCode){
                case 1:
                    Member m = (Member) data.getSerializableExtra("m");
                    Toast.makeText(this,m.toString(),Toast.LENGTH_SHORT).show();//수정된 정보를 toast로 띄운다.
                    break;
            }
        }
    }
```
- 결과 화면<br/>
![image](https://user-images.githubusercontent.com/62331803/90209536-a06f6580-de26-11ea-84c1-d62fa01a77e9.png)<br/>
![image](https://user-images.githubusercontent.com/62331803/90209544-a5341980-de26-11ea-840c-4b25514cc06a.png)

### 연습문제
1. 첫 페이지: 리스트 뷰. OPTIONS 에 ADD메뉴 하나<br/>
1-A. ADD 누르면 ADDACTIVITY로 이동<br/>
- ADDACTIVITY: 이름, 전화번호, 번호 종류(라디오 버튼*), 이미지(스피너), 추가 버튼<br/>
- 추가하면 첫 화면으로 돌아와서 띄워주기<br/>
1-B. CONTEXT MENU (EDIT, DEL)<br/>
- 수정하면 첫 화면으로 돌아와서 띄워주기<br/>
