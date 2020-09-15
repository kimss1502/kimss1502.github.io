---
layout: single
categories: 
  - 안드로이드
title: "안드로이드의 Thread와 Process"
toc : true
---

> Java Thread에 대한 기본은 아래 포스팅을 참고할 것 <br>
> - [Java의 Thread]({% post_url java/2016-01-01-Thread %}) <br>
> - [Thread 상태를 조절하는 메서드]({% post_url java/2016-01-01-Thread 상태를 조절하는 메서드 %})


## 1. Process와 Thread

 - Process <br/>
  프로세스는 운영체제로부터 주소공간, 파일, 메모리와 같은 자원을 할당받는 하나의 작업 단위이다. <br/>
 프로세스는 자신만의 고유 공간과 자원을 할당 받기때문에 서로 다른 프로세스간 직접적인 공유가 불가능 하다.

 - Thread <br/>
 스레드는 한 프로세스 내에서 실제 동작되는 여러 실행의 흐름으로, 프로세스 내의 주소 공간이나 자원들을 대부분 공유하면서 실행된다. 

 
### # 안드로이드는 Multi Process, Multi Thread 이다.
 안드로이드에서 앱을 하나 실행시키면 하나의 프로세스(리눅스 프로세스)가 생성된다. 또한 하나의 프로세스에서는 여러개의 쓰레드를 생성할 수 있다. <br/>
 즉, 안드로이드는 멀티프로세스, 멀티쓰레드 환경이다.
 
 다른 프로세스간에는 자원을 공유할 수 없고, 하나의 프로세스에 있는 스레드간에는 공유가 가능하다. <br/>
 따라서 하나의 앱이 잘못되어 프로세스가 죽으면 해당 프로세스에 있는 모든 스레드 역시 죽게되지만, 다른 프로세스에서 동작하는 다른 앱은 죽지않는다. <br/>


## 2. 리눅스 기반의 Process
 안드로이드는 리눅스 커널을 사용하고 있기에 프로세스 역시 리눅스 프로세스 모델을 기반으로 한다.
 
 리눅스는 모든 사용자에게 기본적으로 OS에 의해 추적되는 고유 번호인 UserID(UID)를 할당한다. <br/>
 Root가 아닌 각 사용자는 권한으로 보호되는 개인 리소스에는 접근할 수 있으나 다른 사용자의 리소스에는 접근할 수 없다.
 
 안드로이드에서 각 앱은 고유한 UserID를 가진다. 그렇기 때문에 각 앱의 고유 영역을 다른 앱이 접근할 수 없는 것이다.

> UserID라고 해서 사용자를 뜻하는것 같지만 안드로이드 레벨에서 볼때 User는 각 앱이라고 볼 수 있다.

### 2.1. 프로세스, 런타임, 앱 간의 관계
 보통 앱과 프로세스는 1:1의 관계지만 필요에 따라 하나의 앱이 여러 프로세스에서 각각 동작하거나 여러 앱을 하나의 프로세스에서 실행할 수도 있다.
 
![프로세스와 런타임 관계](https://kimss1502.github.io/assets/images/android_process_and_runtime.jpeg)

> 하나의 런타임 위에서 모든 앱이 돌아가는것이 아니다. 런타임도 각 프로세스마다 독립적이다.

### 2.2. 앱의 시작 과정
 안드로이드의 주요 컴포넌트(Activity, Service, BroadcastReceiver, ContentProvider)는 앱 시작에 대한 진입점이 될 수 있다. <br/>
 앱 시작 시 아래와 같은 과정을 거친다.
 
 1. 리눅스 프로세스 생성
 2. 런타임 생성
 3. Application 인스턴스 생성
 4. 요청된 앱의 진입점 컴포넌트를 생성

새로운 리눅스 프로세스를 생성하는 것과 런타임을 생성하는 것은 부하가 큰 작업이다. <br/>
 따라서 안드로이드 시스템은 시스템 부트에 **Zygote 라는 특별한 프로세스를 만들어 둔다.** Zygote는 미리 로드된 핵심라이브러리 전체 세트를 가지고 있다. 
 
 새로운 앱 실행시 생성되는 프로세스는 바로 미리 만들어둔 Zygote 프로세스를 복제(fork)하여 만드는 방식을 사용하여 프로세스 생성 시간을 단축한다.

> fork : 프로세스의 복제. fork로 자식 프로세스를 생성할 경우 데이터, heap, stack 영역이 모두 독립적으로 복제된다.

### 2.3. 프로세스 관련 기본 용어

1. 사용자 ID(UID) <br/>
 리눅스는 멀티유저 시스템으로 각 앱은 시스템 입장에서 별도의 사용자이다. <br/>
 따라서 앱이 설치되면 고유의 사용자 ID가 할당된다.
 
2. Process ID(PID) <br/>
 프로세스 고유의 식별자

3. 부모 Process ID(PPID) <br/>
 각 프로세스는 다른 프로세스에 의해서 생성되고, 프로세스끼리는 트리 계층 구조를 형성한다. <br/>
 따라서 각 프로세스는 부모 프로세스를 가지게된다. <br/>
 안드로이드의 경우 모든 프로세스는 Zygote를 fork하여 생성되므로 모든 프로세스 부모는 Zygote 이다.
 
### 2.4. 앱의 프로세스 정보 찾기
 실행중인 모든 앱의 프로세스 정보는 ADB쉘에서 `ps(process status)` 명령어로 알아낼 수 있다. <br/>
 참고로 안드로이드의 ps명령은 리눅스 ps와 같지만 옵션에서는 차이가 있다.

```
> adb shell ps | grep com.skt.skaf.A000Z00040

USER      PID   PPID  VSIZE   RSS    WCHAN      PC           NAME
u0_a1     11995 3187  3752432 307836 SyS_epoll_ 0000000000 S com.skt.skaf.A000Z00040
``` 
 
 모든 Thread 정보는 -t 옵션으로 확인할 수 있다. 앱의 work Thread는 모두 UI thread로부터 만들어지기 때문에 이 thread들의 PPID는 UI thread의 PID와 동일하다.
 
```
> adb shell ps -t | grep u0_a1

u0_a1     11995 3187  3971920 228360 SyS_epoll_ 0000000000 S com.skt.skaf.A000Z00040
u0_a1     12000 11995 3971920 228360 do_sigtime 0000000000 S Signal Catcher
u0_a1     12001 11995 3971920 228360 futex_wait 0000000000 S ReferenceQueueD
u0_a1     12002 11995 3971920 228360 futex_wait 0000000000 S FinalizerDaemon
u0_a1     12003 11995 3971920 228360 futex_wait 0000000000 S FinalizerWatchd
u0_a1     12004 11995 3971920 228360 futex_wait 0000000000 S HeapTaskDaemon
u0_a1     12005 11995 3971920 228360 binder_thr 0000000000 S Binder_1
u0_a1     12006 11995 3971920 228360 binder_thr 0000000000 S Binder_2
u0_a1     12019 11995 3971920 228360 futex_wait 0000000000 S Thread-41486
u0_a1     12037 11995 3971920 228360 futex_wait 0000000000 S Answers Events
u0_a1     12040 11995 3971920 228360 futex_wait 0000000000 S Crashlytics Exc
u0_a1     12044 11995 3971920 228360 futex_wait 0000000000 S measurement-1
u0_a1     12045 11995 3971920 228360 futex_wait 0000000000 S AsyncTask #1
u0_a1     12046 11995 3971920 228360 futex_wait 0000000000 S pool-5-thread-1
u0_a1     12059 11995 3971920 228360 futex_wait 0000000000 S Timer-0
u0_a1     12060 11995 3971920 228360 futex_wait 0000000000 S AsyncTask #1
u0_a1     12061 11995 3971920 228360 futex_wait 0000000000 S AsyncTask #2
u0_a1     19342 11995 3971920 228360 SyS_epoll_ 0000000000 S RenderThread
u0_a1     19343 11995 3971920 228360 futex_wait 0000000000 S AsyncTask #6
u0_a1     21386 11995 3971920 228360 futex_wait 0000000000 S AsyncTask #4
u0_a1     21425 11995 3971920 228360 SyS_epoll_ 0000000000 S JavaBridge
u0_a1     21451 11995 3971920 228360 futex_wait 0000000000 S AsyncTask #5
u0_a1     21457 11995 3971920 228360 futex_wait 0000000000 S Timer-5
....
```

 앱이 구동될때 많은 thread가 생성되는데 이 과정에서 리눅스 프로세스와 안드로이드 런타임을 관리하는 thread도 함께 생성된다. <br/>
 
 앱에서 관심있게 봐야할 thread는 **Main thread(UI thread), Binder thread, Background thread(work thread)** 이다. <br/>


## 3. 안드로이드의 Thread
 안드로이드에서 Thread는 기본적으로 자바의 Thread를 사용하며 이는 Linux native pthread(POSIX Thread)의 자바 구현체이다.<br/>
 
 하지만 안드로이드 플랫폼은 여기에 특별한 속성 3가지를 더 추가하였는데 앱 관점에서 **UI thread, Binder thread, Background thread** 가 있다.
 
### 3.1. UI Thread(메인 Thread)
 하나의 프로세스는 반드시 하나 이상의 쓰레드를 가진다.<br/>
 안드로이드에서 프로세스 생성시 함께 생성되는 쓰레드를 메인쓰레드라 부른다.<br/>
 DDMS에서 PID와 TID가 동일한게 메인쓰레드이다. 
 
 메인 쓰레드는 UI 쓰레드라고도 불리는데 안드로이드에서 유일하게 UI elements에 접근할 수 있는 쓰레드이기 때문이다. <br/>
 안드로이드는 메인쓰레드 이외의 쓰레드가 GUI를 수정하는 것을 허용하지 않고 만약 이를 위반하면 `CalledFromWrongThread Exception`이 발생한다. <br/>
 (메인 쓰레드만 UI 변경을 허용하게 강제로 제약함으로써 UI변경에 대한 동기화 이슈를 차단함.)
 
 UI elements의 이벤트 처리는 순차적으로 처리되는데 만약 처리 시간이 긴 작업을 메인 thread에서 하게 될 경우 UI 전체가 block 되어 반응성이 떨어진다. <br/>
 또한 안드로이드 플랫폼 자체에서 사용자 경험(사용성)을 떨어트리는 작업을 막기위해 5초이상 응답을 하지 않으면 ANR을 발생시킨다.
 
> 예제로 메인쓰레드에서 5초이상 하는 작업을 돌리면 죽지않는다. 하지만 화면을 터치하거나 하면 5초 뒤 죽는다.
> 메인쓰레드가 죽게되는 ANR은 메인쓰레드가 응답을 받지 않을 경우인데 화면을 터치하면 시스템이 전달한 터치이벤트를 메인쓰레드가 받지 못하게 되어 5초 뒤 죽는것이다.


### 3.2. Binder Thread
 바인더는 안드로이드에서 서로 다른 process간 통신을 위해 사용된다. <br/>
 
 각 프로세스는 바인더 통신을 위한 **Thread pool**을 가지고 있고, 프로세스간 통신시 이 thread pool을 이용해 별도의 thread에서 바인더 통신을 한다.
 
### 3.3. Background Thread(Work thread)
 앱이 명시적으로 생성하는 모든 Thread는 Background Thread이다. <br/>
 Background thread는 앱의 main thread(UI thread)에서 파생되기 때문에 UI thread의 속성(우선순위)들을 상속받는다.
 
 앱에서 UI thread와 Background thread는 다르게 사용되지만 리눅스 입장에서 두 thread는 모두 평범한 네이티브 thread이며 동일하게 취급된다. <br/>
 
 **모든 UI 변경이 UI thread에서만 발생하도록 하는 제약 사항은 리눅스의 제약사항이 아니라 안드로이드 프레임워크의 WindowManager에 의해서 강제되는 것이다.**
 

## 4. 리눅스의 Thread 스케줄링

 리눅스에서 실행을 위한 기본 단위는 프로세스가 아니라 쓰레드로 스케줄링은 쓰레드의 스케줄링에 대한 것이다.
 
 프로세서는 멀티 쓰레드를 위해 각 쓰레드가 실행될 실행시간을 할당받는데 그 시간을 할당해주는 역할은 스케줄러가 한다.
 
 안드로이드의 경우 쓰레드는 리눅스 커널의 표준 스케줄러에 의해 스케줄링된다.<br/>
 **즉, 앱의 각 쓰레드는 프로세서로부터 실행시간을 할당받기 위해 단말 모든 앱의 모든 쓰레드와 경쟁함을 의미한다.**
 
 리눅스 커널 스케줄러는 **완전히 공정한 스케줄러(completely fair scheduler-CFS)**이다. '완전히 공정하다'는 뜻은 쓰레드의 우선순위 뿐 아니라 각 쓰레드에 부여된 실행시간을 추적하여 실행이 균형을 유지하려 한다는 것을 말한다.
 
 참고로 쓰레드 스케줄링에 영향을 미치는 방법은 **쓰레드 우선순위** 와 **쓰레드 컨트롤 그룹** 이 있다. 

### 4.1. 우선순위
 스케줄러는 각 쓰레드의 우선순위 값을 보고 실행 시간 할당에 참고한다.
 
 리눅스에서 쓰레드의 우선순위는 **niceness value 또는 nice value** 이라고 불린다. <br/> 이 값이 낮을수록 높은 우선순위에 해당한다.
 
 안드로이드에서 리눅스 쓰레드는 -19 ~ 20 까지의 niceness 값을 가지며 default는 0이다. <br/>
 쓰레드의 우선순위는 그 쓰레드를 생성한 부모 쓰레드의 값을 그대로 상속받는다.
 
 앱은 아래 두가지 방법으로 우선순위를 변경할 수 있다.
 
#### # java.lang.Thread
 `Thread.setPriority(int priority)` 를 사용한다. <br/>
 자바의 우선순위값을 기반으로 0~10까지 할당할 수 있다. <br/>
 
 자바는 플랫폼 독립적이기 때문에 리눅스가 할당하는 값과 다르다. 이 값은 높을수록 우선순위가 높게되는데 실제 리눅스에 맵핑되는 값(-19~20)은 필요시 찾아서 쓰자. (거의 변경되지 않지만 안드로이드 버전마다 다를 수도 있음)
  
#### # android.os.Process
 `Process.setThreadPriority(int priority)`<br/>
 `Process.setThreadPriority(int threadId, int priority)`<br/>
 
 리눅스의 niceness 값을 사용하여 우선순위를 변경한다.
 
 
### 4.2. 컨트롤 그룹
 안드로이드는 쓰레드 스케줄링을 위해 일반적인 리눅스 CFS뿐 아니라 별도의 그룹을 만들어 나누어 관리한다. (리눅스에서는 cgroups에 해당함)
 
 컨트롤 그룹은 여러개가 있지만 앱에서 중요한 것은 **foreground group**과 **background group** 이다. <br/>

 기본적으로 포그라운드 그룹이 백그라운드 그룹보다 더 많은 실행시간을 할당받는다.<br/>
 앱이 사용자 눈에 보이는 경우 포그라운드 그룹에 들어있게되고, 사용자에게 보이지 않으면 백그라운드 그룹에 들어간다. <br/>
 따라서 사용자 눈에 보이는 앱이 더 빠르게 동작하게 하여 사용자 경험을 향상시킨다.
 
 컨트롤 그룹을 통해 백그라운드에 있는 앱이 포그라운드 앱의 동작에 영향을 미치는것을 최소화 한다.<br/>
 하지만 포그라운드 앱은 여전히 자기 앱 구동에 필요한 여러 쓰레드를 가지고 있고 이들은 UI 쓰레드와 같은 우선순위를 가지고 있기 때문에 프로세서 할당을 경쟁한다.
 
> UI 쓰레드에서 Background 쓰레드를 생성했기 때문에 기본적으로 우선순위가 동일하게 됨

 이 문제를 해결하기 위해 쓰레드의 우선순위를 백그라운드 그룹으로 만들어버릴 수도 있다.

```
Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND)
```

<br>

--- 

> **[참고 문서]**
> 
> 1. [안드로이드 가이드](https://developer.android.com/guide/components/processes-and-threads.html?hl=ko#Threads) <br/>
> 2. 도서 "이것이 안드로이드다" <br/>