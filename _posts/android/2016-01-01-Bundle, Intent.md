---
layout: single
categories:
  - 안드로이드
title: "Bundle과 Intent"
---

> 프로세스간 통신에 대해서는 [IPC, RPC, Binder에 대해서]({% post_url android/2016-01-01-IPC, RPC, Binder %}) 포스팅을 참고할 것.

## 1. Bundle
 Bundle은 IPC(Inter Process Communication)을 지원하기 위한 Android의 클래스이다. <br/>
 Bundle은 Parcelable을 구현하고 있어 직렬화 할 수 있다. <br/>

### 1.1. 왜 Bundle이 필요한가.
 서로 다른 Process간 객체를 전달하기 위해서는 두 프로세스 모두 해당 객체를 알고 있어야 한다. <br/>
 또한, 객체가 수정이 되면 해당 클래스 파일을 다시 배포해 줘야 하는 문제도 생긴다 <br/>
 
 이에 반해 Bundle은 안드로이드 SDK에 포함되어 있는 직렬화 객체이다. <br/>
 즉, 안드로이드에서 돌아가는 모든 서비스는 이미 Bundle을 알고있고, 수정에 따른 재배포 문제도 생기지 않는다. <br/>
 
### 1.2. Bundle의 특징
 Bundle의 내부는 Map으로 구성되어 있다. 즉, 키와 값으로 되어 있어 다른 프로세스에 전달하려는 값을 map에 저장하는 방식으로 전달할 수 있다.
 
 물론 키는 전달받는 프로세스에서 미리 알고 있어야 한다.

## 2. Intent
 Bundle과 마찬가지로 Intent역시 Pacelable을 구현하고 있어 직렬화 할 수 있다. <br/>
 
 Bundle과 Intent의 차이는 Bundle은 단순히 데이터 전달을 위한 직렬화 객체라면 Intent는 시스템 서비스간 약속된 데이터를 저장하여, 특정 컴포넌트를 실행하고 원하는 데이터를 전달하기 위한 "의도" 라는 점이다. <br/>
 (기본적으로 Activity 실행, Service 실행, Broadcast 전달에 사용함)
 
 Intent-filter란 앱의 Manifest파일에 들어 있는 표현으로, 해당 구성 요소가 수신하고자 하는 인텐트의 유형을 나타낸다. 인텐트 필터를 전혀 선언하지 않으면 명시적 인텐트로만 시작할 수 있다.
 
### 2.1. 명시적, 암시적 Intent

#### 2.1.1. 명시적 Intent (implicit intent) 
 요청을 위한 안드로이드 component를 이름을 통해 명시적으로 지정함. 일반적으로 동일한 앱 내에서 다른 component를 실행할때 사용한다.
  
### 2.1.2. 암시적 Intent (explicit intent)
 요청을 위한 component 이름을 명시하지 않고, 수행할 작업만을 명시하여 이 작업을 수행 할 component를 찾아서 수행하게 한다. <br/>
 암시적 Intent를 수행하면 안드로이드 시스템은 모든 앱의 manifest파일을 뒤지고 각 component의 intent-filter를 찾는다. <br/>
 만약 매칭되는 intent-filter를 찾으면 해당 component를 수행한다. 매칭되는 component가 여러개라면 어떤 component를 실행할지 사용자에게 묻는 dialog를 표시하고 사용자가 선택한 component를 수행한다.

 * 주의 <br/>
  Service는 암시적 인텐트로 사용하지 않는것이 좋다. Service는 눈에 보이지 않기 때문에 암시적으로 수행하면 보안 위험이 있다. Android 5.0(api 21)부터는 암시적 인텐트로 서비스를 실행하면 exception이 발생한다.
   

### 2.2. Intent의 의미를 부여하는 정보
 
| 분류 | 멤버변수 | 설명 | 설정을 위한 메서드 |
|----|-------|-----| ---- | 
| 컴포넌트 정보 | String mPackage, ComponentName mComponent | 요청을 위한 component의 이름으로 명시적 인텐트를 위한것. 즉 명식적으로 컴포넌트를 지정하고 이 정보가 없으면 암시적 인텐트이다. | Intent 생성자, `setComponent()`, `setClass()`, `setClassName()`|
| 액션 | String mAction| 수행할 작업으로 동작을 설명하기 위해 미리 정의한 문자열. 예를 들어 "전화를 건다", "메일을 발송한다" 등으로 암시적 인텐트를 보낼때 해당 Action을 받을 수 있는 컴포넌트들이 응답한다. | Intent 생성자, `setAction()` |
| 카테고리 | HashSet<String> mCategories| component의 종류에 대한 추가정보를 담은 문자열. 예를 들어 런처에서 앱 아이콘을 클릭했을 때 실행되는 액티비티는 `android.intent.category.LAUNCHER`이라고 지정되어 있는 액티비티이다. <br/> 안드로이드의 기본 내장 앱(브라우저, 주소록, 달력, 이메일, 갤러리, 지도, 메시지, 음악)의 경우 별도로 지정된 카테고리가 있다. | `addCategory()` | 
| 데이터 위치, 타입 | Uri mData, String mType | URI는 실행될 컴포넌트가 특정 경로 데이터를 필요로 할 경우 사용된다. (예를 들어 음악실행시 음악파일의 경로)<br/> 때론 URI외에 데이터 타입을 지정하는게 유용할 수있다. 예를 들어 이미지표시와 오디오파일 재생은 URI 형식이 비슷하지만 서로 다른 작업을 하는 동작이다. 이 때 타입을 지정해두면 안드로이드 시스템이 최적의 component를 찾는데 도움이 된다. | URI만 설정 시 `setData()`<br/>, 타입만 설정 시 `setType()`<br/>, 둘다 설정시 `setDataAndType()`|
| 엑스트라 | Bundle mExtras | 각종 컴포넌트 실행시 데이터를 전달하기 위한 용도| `putExtra()`, `putExtras()`|
| 플래그 | int mFlags | 각종 컴포넌트를 제어하기 위한 플래그| `setFlags()`, `addFlag()` |

> setData(), setType()은 각각쓰면 안된다. 서로 덮어쓰기 때문에 함께 쓸때는 꼭 setDataAndType()을 쓰자. 
 
### 2.3. 예제
 아래 예제는 안드로이드 기본 계산기 앱을 실행한다. <br/>
 (Action, Category를 이용한 암시적 인텐트)

````
 // 아래는 앱을 실행했을때 제일 먼저 실행되는 액티비티를 보여달라는 것.
 // 해당 앱의 카테고리가 LAUNCHER인 액티비티를 실행한다.
 intent.setAction(Intent.ACTION_MAIN);
 
 // 카테고리에 계산기를 지정했다.
 intent.addCategory(Intent.CATEGORY_APP_CALCULATOR);
 
 startActivity(intent);
````

 아래 예제는 웹브라우저를 통해 사이트를 연다.  <br/>
 (Action, Uri를 이용한 암시적 인텐트)
 
````
 // 어떤 데이터를 보여달라고 하는 Action으로 Data에 따라 다른 액티비티가 실행.
 intent.setAction(Intent.ACTION_VIEW);
 intent.setData(Uri.parse("http://naver.com"));
 startActivity(intent);
````

 아래 예제는 MP3를 재생한다. <br/>
 (Action, Uri, DataType을 이용한 암시적 인텐트)
 
````
 intent.setAction(Intent.ACTION_VIEW);
 // "file:///" 은 단말기 내부의 파일이라는 뜻.
 String mp3Path = "file:///" + (mp3 파일 위치);
 // "audio/*" 는 모든 포맷의 오디오 파일이라는 뜻.
 intent.setDataAndType(Uri.parse(mp3Path), "audio/*");
 startActivity(intent);
````

### 2.4. 암시 Intent로 실행되기 위한 컴포넌트 등록
 Manifest에 `<intent-filter>`가 암시적 컴포넌트의 등록을 위한 부분이다.
 
#### 2.4.1. action, category

````
 <intent-filter>
     <!-- 이미지를 보여주는 기능을 함. -->
     <!-- "action.ACTION_IMAGE_VIEW"는 사실 임의로 지정한 값이다. -->   
     <!-- 실제로는 android.intent.action.VIEW 를 사용하자. -->
     <!-- 임의로 지정한 값은 외부에서도 알고있어야 하기 때문에 특정 용도에 쓴다.-->   
     <action android:name="action.ACTION_IMAGE_VIEW" />
     
     <!-- DEFAULT 지정이 되어 있어야 암시적 Intent로 실행이 가능함. -->   
     <category android:name="action.intent.category.DEFAULT" />
 </intent-filter>
```` 
 
 DEFAULT 카테고리를 추가하는 이유는 Intent 객체에 default로 category가 DEFAULT로 지정이 되어있기 때문이다.
 따라서 만약 DEFAULT 카테고리를 intent-filter에 추가하지 않으면 해당 Intent를 컴포넌트가 받을 수 없다.
 
 안드로이드에서 DEFAULT 카테고리의 유무는 암시적 인텐트를 받을 수 있는 컴포넌트인지를 구별할 수 있기도 하다.
 
 
#### 2.4.2. data
 URI를 통해 어떤 암시적 Intent를 받았는데 URI에서도 특정 값에 해당하는 URI만 받고자 할 때 쓸 수 있다. (잘쓰지는 않는것 같다.)
 
 예를 들어 A 라는 홈페이지를 열때만 내 앱의 컴포넌트를 활성화 시키고 다른 홈페이지는 무시하고 싶은 경우 활용 가능하다.
 
````
 <intent-filter>
     <action android:name="action.ACTION_IMAGE_VIEW" />
     <category android:name="action.intent.category.DEFAULT" />
     
     <data
          android:scheme="http"
          android:host="www.superdroid.com"
          android:port="80"
          android:path="/files/images/aaa.png"
          android:mimeType="image/png" />
 </intent-filter>    
````

 위와같이 intent-filter가 등록되어 있을 경우 Uri가 위와 정확히 일치해야 암시적 인텐트에 대해 실행된다.
 위와같이 정적인 path말고 pathPrefix나 pathPattern 속성을 이용하면 유연하게 대처할 수 있다. 
 
### 2.5. 암시적 인텐트를 명시적으로 실행하기
 암시적 인텐트로 던지면 안드로이드에서 해당 Action과 Category를 받을 수 있는 모든 컴포넌트가 반응하며, 이 때 다이얼로그를 통해 유저가 선택할 수 있다.
 
 하지만 암시적 인텐트임에도 불구하고 내가 원하는 컴포넌트만 반응하길 원할 수 있는데 이를 위해 package 지정 방법이 있다.
 
````
 intent.setPackage("com.superdroid.test");
```` 
 위와 같이 intent에 package를 지정하면 해당 패키지의 컴포넌트만 한정할 수 있다.
 

### 2.6. Intent의 Extra 
 Intent에서 Extra는 순수 데이터로 Bundle로 되어 있다. 
 따라서 primitive 데이터 및 직렬화된 객체를 저장할 수 있다. 
 
### 2.7. Intent의 Flag
 컴포넌트 실행 시 제어하거나 상태를 변경하는 등의 목적으로 사용된다.
 예를 들어.. 
 `Intent.FLAG_ACTIVITY_NO_ANIMATION` 는 액티비티 실행시 애니메이션을 사용하지 않는다.
 
 
--- 

> 참고자료 <br/>
> [안드로이드 가이드](https://developer.android.com/guide/components/intents-filters.html?hl=ko#Building) <br/>
> 도서 "이것이 안드로이드다"