---
layout: single
categories: 
  - 안드로이드
title: "Process와 Application Lifecycle"
---

 > [구글 문서 번역](https://developer.android.com/guide/components/activities/process-lifecycle.html) 입니다.
 
 안드로이드 App은 리눅스 프로세스 위에서 구동된다. <br/>
 프로세스는 실행이 필요한 Application의 코드가 있을때 생성되고 더 이상 필요하지 않으면서 다른 앱의 실행을 위해 메모리가 회수되어야 하기 전까지 유지된다.

 안드로이드의 특징은 Application Process의 생명주기가 Application 자체에 의해 제어되지 않는다는 것이다. <br/>
 Application의 생명주기는 시스템이 결정하는데 이 때 현재 실행중인 다른 Application Process와의 조합, Process가 사용자에게 얼마나 중요한지에 대한 정도, 시스템에서 사용할 수 있는 전체 메모리양에 따라 결정된다.

 개발자는 Android 구성요소가 process 생명주기에 미치는 영향을 이해하는 것이 중요하다. 구성요소를 올바르게 사용하지 않으면 중요한 작업을 진행하는 도중 시스템에 의해 process가 죽어버릴 수 있다.
 
## 1. BroadcastReceiver와 Process lifecycle
 Receiver가 Broadcast를 수신받았을때 시간이 오래걸리는 동작을 하려고 별도 Thread를 만드는 경우가 있다. <br/>
 하지만 BroadcastReceiver의 onReceive() 메서드가 리턴되고 나면 시스템은 더 이상 Receiver가 더이상 Active 상태가 아니라고 판단한다. 
 
 이 때 Process에 active한 Component가 더이상 없다면 process는 더이상 필요하지 않은걸로 간주되고 시스템에 의해서 process가 kill 될 수 있다. 
 
 만약 이런 현상을 피하고 싶으면 BroadcastReceiver에서 `JobService` 를 써야 한다. 그러면 안드로이드 시스템은 process 내에서 active한 작업이 있다고 간주한다.

## 2. process 우선순위
 메모리가 부족한 상황에서 어떤 프로세스를 종료해야 하는지 결정하기 위해 안드로이드는 실행중인 구성요소 및 구성요소의 상태에 따라 프로세스 우선순위가 주어진다.

 우선순위가 높은 순서대로 아래와 같다.
 
### 2.1. foreground process
 foreground process는 사용자가 현재 수행중인 작업에 필요한 process 이다. <br/>
 아래와 같은 상황은 foreground process로 간주된다.
 
 1. Activity가 화면의 Top에 위치하여 유저와 인터렉션 하고 있음. (`onResume()`이 호출된 Activity이다.)
  
 2. BroadcastReceiver의 `onReceive()` 가 실행중인 경우.

 3. Service의 `onCreate()`,`onStart()`, `onDestroy()`가 실행중인 경우. 

foreground process는 왠만해선 죽지 않는다. 하지만 현재 시스템에 실행중인 process 수가 몇 개 없는데도 메모리가 부족한 경우 죽을 수 있다.

### 2.2. visible process
 visible process는 현재 사용자가 알고있는 작업을 수행하고 있는 process 이다. 따라서, 이러한 단계의 process가 죽으면 사용자 경험에 좋지 않은 영향을 준다. <br/>
 아래와 같은 상황은 visible process로 간주된다.

 1. 사용자가 화면에서 볼 수 있지만 foreground 상태는 아닌 경우. `onPause()`가 호출된 상태로 예를 들면 다이얼로그가 떠있는데 그 뒤에 있는 Activity의 process 이다.
 
 2. `Service.startForeground()` 로 호출된 foregroud service

 3. live wallpaper나 입력 서비스 등과 같이 유저가 인식할 수 있는 시스템의 특정 기능.
 
visible process는 foreground process 보다 제한적이지만 여전히 사용자에 의해 제어되는 process이다. <br/>
 이 process 또한 매우 중요하게 여겨지며 foreground process 때문에 죽어야 하는 경우가 아니라면 계속 유지된다.

### 2.3. service process
 service process는 `startService()` 에 의해 실행된 하나의 service가 돌아가는 process 이다. <br/>
 이 process는 사용자에게 바로 보이지 않지만 background 네트워크 업로드/다운로드와 같이 사용자가 염두에 두고 있는 작업을 수행한다. 따라서 foreground process 나 visible process 를 구동시키는데 메모리가 부족한 상황이 아니라면 service process 는 유지된다.
 
 장시간(ex- 30분 이상) service가 돌고있으면 process의 우선순위가 낮아져서 cache된 LRU 리스트에 들어갈 수 있다. <br/>
 이를 통해 memeory leak이나 다른 문제가 있는 service가 오래동안 메모리를 잡아먹으면서 이 때문에 cached process 를 효율적으로 사용하지 못하게 되는 상황을 방지할 수 있다.

### 2.4. cached process
 cached process 는 현재 중요하지 않은 프로세스로 메모리가 필요할때 언제든 시스템에 의해 죽을 수 있다. <br/>
 
 시스템상에서 application 간 효율적인 전환을 위해 여러개의 cached process가 존재하고 정기적으로 오래된 process를 종료시킨다. 심각한 상황에서는 모든 cached process를 종료시키게 되고 cached process가 모두 종료된 상태라면 service process를 죽이기 시작한다.
 
 이 프로세스는 유저가 볼 수 없는 상태의 Activity(`onStop()`이 불린 이후)를 하나 이상 보유한 process이다. <br/>
 Activity를 life-cycle에 따라 정상적으로 구현했다면 이 프로세스는 유저 경험에 영향을 미치지 않는다. 
 
 이 프로세스는 내부적으로 종료시킬 프로세스 우선순위를 위해 LRU list를 가지고 있다. <br/>
 LRU list를 관리하는 정확한 정책은 플랫폼 구현 세부사항에 따라 다르지만 일반적으로 다른 유형의 프로세스보다 유용한 프로세스를 오래 유지하려고 한다. (ex- 런처, 마지막으로 본 Activity의 process 등)


### 2.5. 정리
 시스템이 프로세스 우선순위를 결정할때 이 process내부에 있는 요소 중 가장 높은 level의 우선순위를 적용한다. 안드로이드의 component가 프로세스 우선순위에 어떻게 영향을 미치는지는 각 component 상세 내용을 참고하자.

 프로세스의 우선순위는 dependency가 있는 다른 프로세스에 의해 상승할 수 있다. <br/>
 예를 들어 process A가 Servie Binding을 통해 process B와 바인딩한 경우.. 또는 process B가 ContentProvider를 제공하고 process A가 이를 통해 접근한 경우.. B process의 우선순위는 A process 만큼 상승한다.

