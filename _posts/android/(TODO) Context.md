---
layout: single
categories:
  - 안드로이드
title: "안드로이드 Context"
---


## 1. 안드로이드 Developer 사이트의 설명
 어플리케이션 환경에 관한 글로벌 정보를 접근하기 위한 인터페이스. <br/>
 **Abstract 클래스이며 실제 구현부는 안드로이드 시스템에 의해 제공**된다. Context를 통해 어플리케이션에 특화된 리소스나 클래스에 접근할 수 있을 뿐만 아니라 어플리케이션 레벨의 작업 (Activity 실행, Intent 브로드캐스팅, Intent 수신 등)을 수행하기 위한 API 를 호출 할 수도 있다.

## 2. Context에 대한 고찰
 아래 내용에 따르면 Context는 크게 두 가지 역할을 수행하는 Abstract 클래스다.

 1. 어플리케이션에 관하여 시스템이 관리하고 있는 정보에 접근하기<br/>
    (ex- getPackageName(), getResourece() )
    
 2. 안드로이드 시스템 서비스에서 제공하는 API 를 호출 할 수 있는 기능<br/>
    (ex- startActivity(), bindService() )

--

> 아래는 블로그 내용 복사한 것.

### 2.1 왜 Context 가 필요할까?
 Context와 관련하여 'Context가 왜 필요하지?' 라고 생각합니다.
 전역적인 어플리케이션 정보에 접근하거나 어플리케이션 연관된 시스템 기능을 수행하기 위해, 시스템 함수를 호출하는 일은 안드로이드가 아닌 다른 플랫폼에서도 늘상 일어나는 일입니다. 또, 그런 일들은 대게의 경우 어떠한 매개체를 거칠 필요없이, 직접적으로 시스템 API 호출하면 됩니다. <br/>
 반면 안드로이드에서는 Context 라는 인스턴스화된 매개체를 통해야만 유사한 일들을 수행할 수 있습니다.

 예를 들어, C# 에서 어플리케이션 이름을 가져오고, 다른 어플리케이션을 실행시키는 코드는 아래와 같이 작성될 수 있습니다.

```
 //Get an Application Name.
 String applicationName = System.AppDomain.CurrentDomain.FriendlyName;

 //Start a new process(application)
 System.Diagnostics.Process.Start("test.exe");
```

반면에 안드로이드 Activity 에서는 아래와 같이 작성 해야 됩니다.

```  
//Get an application name
String applicationName = this.getPackageName();

//Start a new activity(application)
this.startActivity(new Intent(this, Test.class));
```

 보시는 것 처럼, C# 의 경우 System 단에서 제공하는 정적 함수(static function)를 호출 함으로서 간단하게 할 수 있는 일들을 안드로이드에서는 Context 에 정의된 인스턴스 함수를 호출해야만 가능하게 되어있습니다. 
 
 즉, 위에서 처럼 반드시 인스턴스화된 Context 클래스(여기서는 this) 를 사용해야 되는 셈이지요. 

 왜 이런 차이가 생기게 된걸까요? 

 대답하기 애매한 질문인 경우에, 질문을 거꾸로 뒤짚어 보면 서광이 비치는 경우가 있습니다. 안드로이드가 아닌 플랫폼에서는 어떻게 정적 함수 호출을 통해서 어플리케이션에 관한 정보를 가져오고, 시스템 함수를 호출 할 수 있는 걸까요? 

 제가 OS 에 관한 지식이 짧아, 꼭 찝어 이야기하기에는 어려움이 있지만... 일반적인 경우, 어플리케이션이 프로세스가 아주 긴밀하게 연결되어 있기 때문입니다. OS 커널의 가장 중요한 일 중 하나는 프로세스를 관리하는 것 입니다. (아마도..) 특정 프로세스가 특정 어플리케이션과 맵핑 된다면, 우리는 별다른 매개체 없이 시스템에 직접 프로세스의 정보에 관한 물어 볼 수 있고, 프로세스와 연관된 시스템 함수를 호출 할 수 있습니다. (이미 관리하고 있는 정보에 대해 묻고 있음으로)

 그런데 안드로이드에서 어플리케이션과 프로세스와의 관계는 조금 요상한 구석이 있습니다. 안드로이드에서 어플리케이션과 프로세스는 서로 독립적으로 존재입니다. 이와 관련된 구체적인 내용이나 이러한 구조를 갖는 원인에 대해서는 안드로이드 멀티태스킹에 관한 [구글 개발자 블로그 포스트](http://blog.naver.com/huewu/110085391353) 에서 자세하게 다루어져 있습니다.

 예를 들자면, 안드로이드 플랫폼에서는 프로세스가 없는 상황에도 어플리케이션은 살아있는 것처럼 사용자에게 표시되기도 하고, 메모리가 부족한 상황이 될 경우, 작동중이던 프로세스가 강제로 종료되고, 대시 해당 프로세스에서 작동중이던 어플리케이션에 관한 일부 정보만 별도로 관리하고, 이 후에 메모리 공간이 확보되면 저장되어있던 어플리케이션 정보를 바탕으로 새로운 프로세스를 시작하는등의 신기한 일이 벌어집니다.

 안드로이드에서도 프로세스는 당연히 OS 커널 (리눅스)에서 관리됩니다. 어플리케이션과 프로세스가 별도로 관리되고 있다면, 어플리케이션 정보는 어디에서 관리하고 있을까요? 안드로이드의 시스템 서비스 중 하나인 ActivityManagerService 에서 그 책임을 집니다. 그렇다면 ActivityManagerService 는 어떤식으로 어플리케이션을 관리하고 있을까요? 이외로 단순 합니다. 특정 토큰을 키값으로 'Key-Value' 쌍으로 이루어진 배열을 이용해 현재 작동중인 어플리케이션 정보를 관리합니다.

 거의 결론에 다다른거 같습니다. Context 는 어플리케이션과 관련된 정보에 접근하고자 하거나 어플리케이션과 연관된 시스템 레벨의 함수를 호출하고자 할 때 사용됩니다. 그런데 안드로이드 시스템에서 어플리케이션 정보를 관리하고 있는 것은 시스템이 아닌, ActivityManagerService 라는 일종의 또 다른 어플리케이션입니다. 따라서 다른 일반적은 플랫폼과는 달리, 안드로이드에서는 어플리케이션과 관련된 정보에 접근하고자 할때는 ActivityManagerService 를 통해야만 합니다. 당연히 정보를 얻고자 하는 어플리케이션이 어떤 어플리케이션인지에 관한 키 값도 필요해집니다.

 즉, 안드로이드 플랫폼상에서의 관점으로 샆펴보면, Context 는 다음과 같은두 가지 역할을 수행하기 때문에 꼭 필요한 존재입니다.
 
  1. 자신이 어떤 어플리케이션을 나타내고 있는지 알려주는 ID 역할 
  2. ActivityManagerService 에 접근할 수 있도록 하는 통로 역할 

일반 OS 플랫폼에서 어플리케이션은 곧 Process 입니다. 특정 어플리케이션이 OS 에게 내가 어떤 Process 인지만 알려주면 어플리케이션 관련된 정보를 얼마든지 획득 할 수 있습니다. 이른바 자신의 존재 자체가 자신임을 증명해주는 '지문인식' 혹은 '홍채인식' 등의 '생체인식' 과 비슷한 개념이기 떄문에 Context 와 같은 애매한 중간 매개체가 존재할 이유가 없습니다. 

 하지만 안드로이드 플랫폼은 조금 다릅니다. 비유하자면 '생체인식' 보다는 'ID카드' 를 통한 보안 시스템과 유사한 구조입니다. 특정 어플리케이션이 자신이 본인임을 확인 받을 수 있는 방법은 자신이 작동중인 Process 를 보여주는 것이 아니라, 자신이 건내받은 ID카드를 제시하는 것 입니다. 이 때 ID카드의 역할을 수행하는 것이 바로 Context 이고, 당연히 이 카드는 위변조가 가능하기때문에, 자신의 권한을 제삼의 어플리케이션에게 넘겨주는 PendingIntent 와 같은 기능도 가능해집니다.
 
--

### 2.2. Context 는 언제 태어날까?
 Context 에 관한 마지막 질문은 약간의 부록과도 같습니다. Context 는 언제 태어날까요? 대답은 당연히 어플리케이션이 태어날 때입니다. 그렇다면 하나의 어플리케이션을 구성하는 각종 컴포넌트들(Activity,Service,BroadcastReceiver) 들은 모두 동일한 Context 를 공유해서 사용하고 있을까요? 저도 처음에는 그렇지 않을까 생각했었는데 알고보니 그렇지는 않더군요.

 Activity 와 Service 가 생성될 때 만들어지는 Context 와 BroadcastReceiver 가 호촐될 때( onReceive() ) 전해지는 Context 는 모두 서로다른 인스턴스입니다. 즉, Context 는 어플리케이션이 시작될 때는 물론이요, 어플리케이션 컴포넌트들이 생성될때마다 태어나는 셈입니다. 물론, 새롭게 생성되는 Context 들이 부모와 완전히 독립되어 있는 존재는 아니고 '거의' 비슷한 내용을 담고 있습니다.

 어째서 동일한 Context 인스턴스를 어플리케이션 컴포넌트들이 공유해서 사용하지 않고, 모두 서로 다른 (그러나 알고보면 알맹이는 거의 같은) 인스턴스를 만들어 사용하고 있을까요? 음... 어려운 문제입니다. 잘 모르겠네요. 일단 한 가지 분명한 원인이 있습니다.

 Context 의 기능 중, 시스템 API 를 호출하는 기능과 관련되어 한 가지 문제점이 있습니다. 어떤 어플리케이션 컴포넌트가 시스템 API를 호출하느냐에 따라서 서로 다른 결과가 나타나야 한다는 점입니다. 예를들어, Service 에서 Activity 실행하기포스트에서 언급한 것 처럼, 동일한 형태로 startActivity 메서드 호출하더라도, 일반적인 Activity 에서는 정상적으로 새로운 Activity 를 시작하게 되지만, Service 에서 호출할 경우에는 예외가 발생합니다. 만일 어플리케이션을 구성하는 Service 와 Activity 가 서로 동일한 Context 인스턴스를 공유하고 있다면 동일한 메서드 호출에 대하여 서로 다른 결과를 나타내도록 구현하지 못했을겁니다.

 따라서, 현재 안드로이드 시스템은 어플리케이션 Context 를 기반으로 컴포넌트를 위한 Context 를 생성할 때 해당 Context 가 어떤 종류의 컴포넌트인지 알 수 있도록 약간의 표시를 해두곤 합니다. 

 또 한가지 안드로이드 개발팀의 코딩 스타일과 관련된 문제가 아닌가 하는 심증을 갖고 있습니다. 안드로이드 어플리케이션 컴포넌트들은 Context 를 포함하는 대신 상속받은 방식으로 구현되어 있습니다. Context 를 포함하는 방식이였다면 아마 작성해야할 코드량이 제법 늘어났을 것으로 짐작됩니다 (개별 컴포넌트 클래스마다 추상 API 들을 반복해서 구현해주어야 하니까...). 하지만 포함대신 상속받는 방식이 적용된만큼 당연하게 어플리케이션 Context 인스턴스를 어플리케이션을 구성하는 개별 컴포넌트들이 동일하게 사용하는 것은 어플리케이션 클래스 자체를 엄청나게 복잡하게 구현하지 않는 이상 불가능한 일이 되어 버렸습니다.

### 2.3. 마무리
 결론! 결국, 안드로이드 Context 는 여러가지 이유로 기존 플랫폼과는 다른 방식으로 어플리케이션을 관리하고 있고, 때문에 기존 플랫폼들에서는 단순하게 시스템 API 를 통해 할 수 있는 일들을, Context 인스턴스라는 조금은 귀찮지만 강력한 녀석을 통해 대행 처리하고 있다고 할 수 있겠습니다.

## 3. Context 구현체에서 볼수있는 여러 클래스들

### 3.1. Context
 Context 자체는 abstract 클래스이다. 

### 3.2. ContextWrapper
 ContextWrapper는 Context를 사용하는 호출을 다른 Context(내부에 baseContext를 멤버로 가짐) 에게 위임하는 구현체이다. <br/>
 Adapter 패턴으로 구현된 형태로 원본 Context를 변경하지 않으면서도 동작을 변경하기 위해 서브클래스화 되어있다. 
 
 Activity와 Service는 ContextWrapper를 상속받고 있다.

```
public class ContextWrapper extends Context {
    Context mBase;

    public ContextWrapper(Context base) {
        mBase = base;
    }
    ...
}
```

### 3.3. ContextImpl
 ContextImpl은 baseContext를 제공하는 Activity나 Service와 같은 컴포넌트들의 Context API에 대한 일반적인 구현체이다. <br/>
 
 BaseContext를 제공한다는 것은 그 자체가 ContextWrapper를 상속받고 있다는 뜻이다. <br/>
 Activity나 Service는 ContextWrapper를 상속받고 있고 ContextWrapper는 baseContext를 멤버로 가지고 있는데 이 baseContext의 구현체가 ContextImpl이다.

> ContextImpl을 구현하는 구현체는 안드로이드 플랫폼이 Activity나 Service 같은 컴포넌트를 생성할때 만들어준다.

## 4. Context 타입별 차이 

### 4.1. Application
 앱 프로세스 레벨에서 동일한 싱글턴 인스턴스이다. <br/>
 호출방법은 아래와 같다.
 
 1. `Activity.getApplication()`
 2. `Service.getApplication()`
 3. `Context.getApplicationContext()`

### 4.2. Activity, Service
 Activity, Service의 인스턴스별로 고유한 Context가 생성된다.
 
### 4.3. BroadcastReceiver
 BroadcastReceiver는 Activity나 Service와 달리 그 자체는 Context가 아니다. <br/>
 대신 Broadcast를 수신받을때 `onReceive()` 메서드에서 Context를 파라미터로 전달받는데 Broadcast를 전달받을때 마다 Context는 별개의 instance로 생성된다. <br/>
 
참고로 `onReceive()` 에서 전달받는 Context는 아래 두개의 기능이 disabled 된 Context이다.

 1. `registerReceiver()`
 2. `bindService()`
 
### 4.4. ContentProvider
 BroadcastReceiver처럼 그 자체로는 Context가 아니다. <br/>
 하지만 `getContext()` 메서드를 통해 Context에 접근할 수 있다.
 
 만약 ContentProvider가 내 앱의 프로세스에서 호출되었다면 리턴되는 Context는 Application Context와 동일하다. 
 
 하지만 만약 다른 프로세스에서 호출되었다면 그 프로세스의 Context가 리턴된다.

### 4.5. Context별 기능

![Context 사용](./_attach/context.png)

1) Application과 Service는 일반적으로 `startActivity`를 할 수 없다. 하지만 NEW_TASK flag를 이용하면 가능하다.

2) Activity가 아닌곳에서 LayoutInflate 를 하는것은 가능하다. 하지만 애플리케이션의 Theme을 사용하지 않고 시스템 default Theme을 사용하여 inflate 된다.

> 오래된 자료라 다 맞지는 않을수 있음.

## 5. 호출 방식에 따른 Context 차이

### 5.1. getBaseContext()
 ContextWrapper의 메서드로 baseContext를 리턴한다. <br/>
 
 주의할것은.. `Activity나 Service의 Context(자신) != getBaseContext()` 임을 알아야 한다. 
 
---
 ContextWrapper와 Context는 Adapter 패턴으로 구현되었다. 실제 Context 구현체는 CotextImpl에 의해 구현되며, 이를 구현하는 곳은 시스템이다. 즉, 시스템 서비스를 호출할 수 있는것이 이 구현체를 시스템이 만들었기 때문이다.
 Context마다 기능이 다른건.. 
 
  
### 5.2. getApplicationContext()
 ContextWrapper의 메서드이다. 어플리케이션 Context를 리턴한다.
 
### 5.3. getApplication()
 Activity와 Service에서 Application 인스턴스를 리턴한다.
 
### 5.4. View.getContext()
 view가 속한 Activity의 Context이다. 
 
 
## 6. Context를 멤버로 가지는 Singleton
 경우에 따라 Context를 멤버로 가지는 싱글턴 객체를 만들때가 있다. <br/>
 이 때 전달하는 Context는 Activity나 Service의 Context가 되서는 안된다. <br/>
 당연히.. Application Context가 되어야 한다.

 Activity나 Service의 Context가 사용되면 결국 참조가 유지되어서 memory leak이 발생한다.

 * 아래는 잘못된 예. 

```
public class CustomManager {
    private static CustomManager sInstance;
 
    public static CustomManager getInstance(Context context) {
        if (sInstance == null) {
            sInstance = new CustomManager(context);
        }
 
        return sInstance;
    }
 
    private Context mContext;
 
    private CustomManager(Context context) {
        mContext = context;
    }
}
```

 * 아래처럼 할 것.

```
public class CustomManager {
    private static CustomManager sInstance;
 
    public static CustomManager getInstance(Context context) {
        if (sInstance == null) {
            //Always pass in the Application Context
            sInstance = new CustomManager(context.getApplicationContext());
        }
 
        return sInstance;
    }
 
    private Context mContext;
 
    private CustomManager(Context context) {
        mContext = context;
    }
}
```

-- 
[참고 문서 ]
> [참고자료1](http://blog.naver.com/PostView.nhn?blogId=huewu&logNo=110085457720) <br/>
> [참고자료2](https://possiblemobile.com/2013/06/context/) <br/>
> [참고자료4](http://wengdiiiy.tistory.com/1) <br/>
> [참고자료5](http://shnoble.tistory.com/57) <br/>
> [참고자료6](https://www.quora.com/What-is-getBaseContext-method-in-Android) <br/>