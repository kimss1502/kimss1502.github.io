---
layout: single
categories:
  - 안드로이드
title: "Looper, MessageQueue, Handler"
---


# Looper, MessageQueue, Handler
> `(안드로이드) 안드로이드의 Thread와 Process.md` 참고 <br/>
> `(Java) Thread.md` 참고 

## 1. 안드로이드의 Message Handling
 자바에서 Thread간 통신하는 방법에는 pipe, shared memory, blocking queue 등 여러가지가 있다. <br/>
 
 이 방법들은 안드로이드에서 역시 그대로 사용할 수 있지만 모두 쓰레드가 block 될 수 있다는 문제가 있다. Work Thread는 상관 없지만 UI Thread가 block될 경우 사용자 반응성이 저하된다. <br/>
 이러한 점 때문에 안드로이드에서 UI Thread에 대해서는 nonblocking 소비자-생산자 패턴 모델을 사용한다. <br/>
 
 Message Handling은 thread간 메세지(데이터) 통신방법에 대한 것이고, 이는 안드로이드 플랫폼의 핵심 내용으로 `android.os` 패키지에서 제공하고 있다.
 
### 1.1 Message handling 매커니즘
 안드로이드 message handling은 아래 4가지를 통해 이뤄진다.
 
 1. `android.os.Looper` <br/>
  Looper는 message dispatcher로서, 하나의 소비자 thread에서 동작한다.
  
 2. `android.os.Handler` <br/>
  Handler의 경우 소비자 thread의 message 처리역할 및 생산자 thread가 MessageQueue에 Message를 넣을 수 있도록 하는 interface를 제공한다. 

 3. `android.os.MessageQueue` <br/>
  MessageQueue는 크기제한이 없는 Linked List이다. 모든 Looper는 하나의 MessageQueue만 가질 수 있다.

 4. `android.os.Message` <br/> 
  Message는 임의의 데이터나 객체를 담는 객체로 소비자 쓰레드에서 실행될 내용을 담고있다.
  
### 1.2. 전체 과정 간략 정리 (중요)
우선 안드로이드의 제약 조건을 보자면 다음과 같다.
  
  1. UI thread는 block 되면 안된다.
  2. UI 변경은 UI thread에서만 할 수 있다.

안드로이드 메세지 핸들링 과정은 이러한 제약조건을 지키기 위한 nonblocking 소비자-생산자 패턴을 구현한 것이다. <br/>

> 원래 소비자-생산자 패턴에 따르면 생산자 thread는 데이터를 생산하기만 하고 소비자 thread는 만들어진 데이터를 소비하기만 한다. 그리고 생산자와 소비자 사이에 만들어진 데이터를 관리하기 위한 공유메모리 영역이 있는데 이 공유 메모리에 대한 생산자 thread와 소비자 thread의 점유 문제 때문에 thread가 block이 된다.

**아래는 Work thread에서 메인 thread로 UI 변경을 요청하는 과정이다.**

![메세지 매커니즘](./_attach/android_msg_machanism.jpeg)

1. 메인 thread 입장에서..
 - 메인 thread는 소비자 thread이다.
 - 소비자 thread는 내부에 Looper를 가지고 있는데 Looper는 MessageQueue를 가지고 있다.
 - MessageQueue는 말 그대로 Message를 가지고 있는 Queue이고 Message는 실행과 관련된 데이터 객체이다.
 - 소비자 thread가 가지고 있는 Looper의 역할은 무한 루프를 돌면서 MessageQueue에 있는  Message를 하나씩 꺼내 dispatch한다.

2. Work thread 입장에서..
 - Work thread는 생산자 thread이다. 
 - 생산자는 요청 데이터(ui를 변경하겠다는 요청)를 만든 뒤 소비자에게 전달해줘야 한다.
 - 이러한 요청이 Message 이다.
 - 그리고 요청을 전달하는 역할은 Handler가 한다.
 - 즉, 생산자 thread인 Work thread는 소비자 thread인 메인 thread에게 Message를 전달해야 하는데 이 때  Handler를 통해서 전달하는 것이다.
 - Handler의 경우 소비자 thread인 메인 thread에게서 얻어와야 한다.

3. Handler 입장에서..
 - 핸들러는 내부에 멤버로 Looper를 참조하고 있다.
 - 즉, 메인 thread로 부터 얻어온 Handler는 내부에 메인 Looper를 알고있다.
 - Work thread에서 Message를 전달받으면 Handler는 자기가 참조하고 있는 메인 Looper의 MessageQueue를 통해 Message를 추가한다.

4. 메인 thread가 Message를 꺼내고 난 뒤..
 - 메인 thread의 Looper에 의해서 Message가 꺼내지고 나면 아래의 dispatch 과정을 거친다.
 - 참고로 Handler를 통해 MessageQueue에 Message를 넣는 과정은 생산자 thread인 Work thread에서 동작하지만, 자체적으로 무한루프를 도는 Looper에 의해 Message가 꺼내지고 dispatch되는  과정은 소비자 thread인 Main thread에서 동작한다.
 - Message는 내부에 멤버로 자기를 전달한 Handler를 참조하고 있다.
 - 이 Handler의 handleMessage() 메서드를 호출한다.
 - 이 동작은 이미 메인 thread에서 동작하고 있기 때문에 UI 변경을 정상적으로 할 수 있다.


> 소비자-생산자 패턴은 `(Java) Thread.md 참고`
  

## 2. MessageQueue
 `android.os.MessageQueue`에 정의된 메시지큐는 단방향 Linked List로 구현되어 있다. <br/>
 메세지큐는 생산자 Thread가 추가한 Message가 차례대로 dispatch되서 소비자 Thread에서 실행될수 있게 한다. 
 
 메세지큐에 추가되는 Message는 timestamp에 따라서 정렬된다. <br/>
 만약 timestamp가 현재 시간 이전이라면 바로 dispatch되고, 미래라면 dispatch하지 않고 대기를 하게 된다.

 ![메세지큐](./_attach/messagequeue.jpeg)

> t1은 dispatch 될 것이고, t2,t3는 대기할 것이다.

 만약 더이상 처리할 Message가 없다면 thread는 block 되고, 다시 Message가 추가되면 실행된다. (단 UI thread는 block 되지 않는다.)
 
### 2.1. MessageQueue.IdleHandler
 처리할 메세지가 없을때 Thread는 block되고 유휴 시간을 가지게 된다. <br/>
 이 시간동안 MessageQueue.IdleHandler를 사용하면 다른 작업을 처리할 수 있다.

```
// 현재 Thread의 MessageQueue를 얻는다.
MessageQueue msgQueue = Looper.myQueue();
// 리스너 드록
MessageQueue.IdleHandler idleHandler = new MessageQueue.IdleHandler();
msgQueue.addIdleHandler(idleHandler);
// 리스너 해제
msgQueue.removeIdleHandler(idleHandler);
``` 

IdleHandler는 하나의 메서드만 가지는 interface이다.

```
interface IdleHandler {
	boolean queueIdle();
}
```

 - return true : IdleHandler를 계속 유효한 상태로 둬서 콜백을 받을 수 있다.
 - return false : IdleHandler를 해제한다. `MessageQueue.removeIdleHandler()`
와 동일한 역할을 한다.

> IdleHandler는 딱히 어디서 쓰면 좋은지는 잘 모르겠다.


## 3. Message

`android.os.Message` 클래스는 컨테이너 객체로서, 데이터나 task를 전달하는데 사용된다. `Message`의 파라미터로는 여러 가지가 있는데, 정리하면 다음과 같다.

| Name     | Type   | Description |
|----------|--------|-------------|
| `what`      | `int`    | 메시지 식별자 |
| `arg1`, `arg2` | `int` | 정수값을 전달하는 경우에 사용되는 간단한 데이터 값. 정수 데이터를 전달하는 경우 Bundle(data)을 전달하는것보다 효율적이다. |
| `obj` | `Object` | 임의의 객체. 다른 프로세스로 전달될 때는 반드시 Parcelable로 구현해야 한다. |
| `data` | `Bundle` | 임의의 데이터 값들을 가지는 컨테이너 |
| `replyTo` | `Messenger` | 다른 프로세스의 핸들러를 참조한다. 프로세스간 통신을 가능하게 한다. |
| `callback` | `Runnable` | 스레드에서 실행할 task. `Handler.post()`에서 전달한 Runnable 객체를 담고있는 내부 인스턴스 변수이다. |

### 3.1. 메세지의 두가지 형태
 메세지는 두가지 형태가 있다.
 
 1. 어떤 동작 자체(Task)인 Runnable(파라미터로는 callback)을 가지는 테스크 메세지
 2. 데이터인 arg, obj, data를 가지는 데이터 메세지

테스크 메세지의 경우 동작 자체를 가지고 있기 때문에 다른 데이터 파라미터들은 아무 의미가 없다. <br/>
 
 소비자 스레드에서 Looper에 의해 Message Queue에 있는 메세지가 처리될때 메세지가 테스크 메세지인 경우 Runnable 객체를 실행시키고, 데이터 메세지인 경우 `Handler.handleMessage(Message msg)`에서 처리할 수 있도록 한다. <br/>
 (테스크 메시지인 경우 handleMessage가 호출되지 않는다.)
 
### 3.2. 메세지의 생명주기
 메세지의 lifecycle은 간단하다. <br/>
 생산자 스레드에서 메시지를 생성 및 초기화하고 소비자 스레드에서 처리된다.
 
 참고로 메세지는 Message pool이 따로 있고 안드로이드 런타임에 의해 재활용된다. <br/>
 따라서 매번 새로운 메세지 인스턴스를 생성하는 오버헤드를 피한다.
 
 ![메시지 라이프사이클](./_attach/message_lifecycle.png)
 
#### 3.2.1. 초기화 상태
 메세지 객체가 생성된 상태. 메세지 객체는 다양한 방법으로 생성할 수 있다. (3.3. 메세지의 생성 참고)

#### 3.2.2. 대기 상태
 메세지가 생산자 스레드에 의해 Message Queue에 삽입되었고, dispatch되기를 기다리고 있는 상태(pending 상태)
 
#### 3.2.3. 전달 상태
 메세지는 루퍼에 의해 Message Queue에서 가져와지고 Message 객체가 참조로 가지고 있는 Handler를 이용해 메세지를 전달한다.(dispatch 과정) 전달된 메세지들은 소비자 스레드에서 실행된다. <br/>
 
#### 3.2.4. 재활용 상태
 메세지 상태가 해제되고 Message pool에 인스턴스가 반환된다. 메세지 객체의 재활용은 런타임에 의해 제어되므로 앱이 명시적으로 수행할 수는 없다.
 
### 3.3. 메세지의 생성 

 1. 명시적인 생성 <br/>
   `Message m = new Message();`
 
 2. empty 메세지 <br/>
   `Message m = Message.obtain();`
   
 3. 데이터 메세지 <br/>
   `Message m = Message.obtain(Handler h);` <br/>
   `Message m = Message.obtain(Handler h, int what);` <br/>
   `Message m = Message.obtain(Handler h, int what, Object o);` <br/>
   `Message m = Message.obtain(Handler h, int what, int arg1, int arg2);` <br/>
   `Message m = Message.obtain(Handler h, int what, int arg1, int arg2, Object o);` <br/>
      
 4. 테스크 메세지 <br/>
   `Message m = Message.obtain(Handler h, Runnable task);`
   
 5. 복사 생성자 <br/>
   `Message m = Message.obtain(Message originMsg);`
   


## 4. Looper
 루퍼는 내부적으로 무한루프를 돌면서 Message Queue에 있는 메세지를 관련된 Handler로 발송하는 일을 한다.

> 일반적으로 쓰레드는 무한루프를 돌고있지 않는 한 내부 작업이 끝나면 종료된다. 안드로이드에서 앱을 실행했을때 아무 동작을 하지 않아도 종료되지 않는 이유는 메인 쓰레드의 메인루퍼에 의해 무한루프를 돌고 있기 때문이다. 메인 쓰레드 외에 다른 쓰레드 역시 루퍼를 사용할 경우 루퍼를 종료하지 않는다면 쓰레드가 종료되지 않는다. 

![루퍼](./_attach/looper.png)

아래는 루퍼의 구현 코드이다.

```
public class Looper{

    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mRun = true;
        mThread = Thread.currentThread();
    }
	
	private static void prepare(boolean quitAllowed) {
		// 쓰레드는 오직 하나의 루퍼만 가질 수 있다. 
		if (sThreadLocal.get() != null) {
			throw new RuntimeException("Only one Looper may be created per thread");
		}
		sThreadLocal.set(new Looper(quitAllowed));
	}

    public static void loop() {
        final MesssageQueue queue = me.mQueue;
        for(;;) {
            Message msg = queue.next();
            ...
            // target은 메세지 객체가 참조하고 있는 Handler 이다.
            msg.target.dispatchMessage(msg);
            ...
        }
    }
}
```
 
 참고로 쓰레드는 오직 하나의 루퍼만 가질 수 있다. 만약 쓰레드에 이미 루퍼가 설정되어 있는데 다시 설정하려고 하면 RuntimeException이 발생한다. <br/>
 위 코드를 보면 메세지큐를 루퍼가 생성하기 때문에 결국 한 쓰레드에는 메세지 큐 역시 하나만 가진다는걸 알 수 있다.

 `msg.target.dispatchMessage(msg)`가 호출될때 메세지가 dispatch되는데 만약 전달 경계(dispatch barrier)를 넘은 메세지가 없다면 blocking이 되고 기다리게 된다.
 
> 이게 이해가 안된다.. 안드로이드는 nonblocking 소비자-생산자 패턴이라고 했다. 그런데 여기서 block 된다는게 무슨 의미인가.. 그럼 UI 쓰레드도 block 되는건가? 아니면 UI 쓰레드만 별게인가? 
 
### 4.1. Looper의 설정
 
```
class ConsumerThread extends Thread{
	public void run() {
		Looper.prepare();
		...
		Looper.loop();
	}
}
``` 
 
 `Looper.prepare()`를 통해 현재 쓰레드의 루퍼와 메세지큐가 생성된다. <br/>
 `Looper.loop()`가 호출되면 무한루프를 돌면서 메세지를 처리한다.
 
### 4.2. Looper의 종료
 아래 두 메서드를 통해 종료할 수 있다.
  
 1. `quit()` <br/>
  전달 경계(dispatch barrier)를 통과한 Message를 포함, MessageQueue의 모든 pending 메시지를 삭제한다. 
 
 2. `quitSafely` (api 18) <br/>
  전달 경계를 넘지 않은 메세지만 삭제한다. 
  
Looper를 제거한다고 thread가 제거되는 것은 아니다. 하지만 무한 루프를 멈추기 때문에 그 스레드에서 더 이상의 작업이 없으면 자연스럽게 제거된다.

루퍼 종료 후에는 기존 루퍼나 새로운 루퍼를 다시 설정할 수 없다. 즉, 해당 쓰레드에서는 더이상 메세지를 처리할 수 없다.
  
### 4.2. UI 쓰레드의 Looper
 메인 쓰레드의 경우 처음 생성될때 기본적으로 Main Looper를 함께 생성하기 때문에 별도로 설정하는 과정이 필요없다. <br/>
 UI 쓰레드의 Looper는 다른 쓰레드의 Looper와 차이가 있다.
 
 1. Looper.getMainLooper()를 통해 어디서든 접근할 수 있다.
 2. 종료시킬 수 없으며 `Looper.quit()`가 호출되면 RuntimeException이 발생한다.
 3. 런타임은 `Looper.prepareMainLooper()`를 통해 메인쓰레드에 루퍼를 연결하는데 이 메서드는 단 한번만 호출될 수 있다. 따라서 메인 루퍼를 다른 쓰레드에 부착하려 하면 Exception이 발생한다.


## 5. Handler
 핸들러는 안드로이드 쓰레드 통신에서 핵심 요소로 메세지의 추가(insertion)와 처리(processing)를 모두 담당한다. <br/>

### 5.1. Handler의 설정
 핸들러는 항상 특정 쓰레드와 연결되어 있어야 하고, 해당 쓰레드에는 메세지를 담을 수 있는 MessageQueue와 메세지를 전달해줄 Looper가 존재해야 한다. <br/>

 만약 루퍼가 없는 상태에서 핸들러 객체를 생성하면 RuntimeException이 발생한다. <br/>
 
 아래는 Handler의 코드 일부이다. 
 
```
// 루퍼를 받지 않는 생성자.
public Handler() { this(null, false); }
public Handler(Callback callback) { this(callback, false); }
public Handler(boolean async) { this(null, async); }

public Handler(Callback callback, boolean async) {
	if (FIND_POTENTIAL_LEAKS) {
		final Class<? extends Handler> klass = getClass();
		if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) && (klass.getModifiers() & Modifier.STATIC) == 0) {
			Log.w(TAG, "The following Handler class should be static or leaks might occur: " + klass.getCanonicalName());
		}
    }
	
	// 핸들러 생성시 루퍼를 명시적으로 받는것이 아니라면 현재 쓰레드의 루퍼와 연결된다.
	mLooper = Looper.myLooper();
	if (mLooper == null) {
		throw new RuntimeException("Can't create handler inside thread that has not called Looper.prepare()");
	}
	mQueue = mLooper.mQueue;
	mCallback = callback;
	mAsynchronous = async;
}

// 루퍼를 명시적으로 받는 생성자.
public Handler(Looper looper) { this(looper, null, false); }
public Handler(Looper looper, Callback callback) { this(looper, callback, false); }
public Handler(Looper looper, Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}

``` 
 
 위 코드를 보면 알수있듯 루퍼를 명시적으로 설정하지 않는 경우 `Looper.myLooper()` 를 통해 현재 쓰레드의 루퍼를 연결하려고 하는데 루퍼가 없으면 RuntimeException이 발생한다. <br/>  
 즉, 메세지큐와 루퍼를 생성하는 `Looper.prepare()` 호출 이전에 핸들러를 생성하려 하면안된다.
 
```
final MessageQueue mQueue;
final Looper mLooper;
```

 핸들러 class에서 Looper와 MessageQueue는 final로 선언되어 있다. <br/>
 즉, 한번 설정된 루퍼가 다른 루퍼로 변경될 수 없다.
 
### 5.2. Handler와 쓰레드의 관계
 하나의 쓰레드에 루퍼와 메세지큐는 하나만 가질 수 있지만 Handler는 여러개를 가질 수 있다. <br/>
 서로 다른 Handler를 참조하고 있는 메세지들이 같은 메세지큐안에 있을 수 있는데, dispatch될때 자신의 Handler 객체를 통해 처리된다.
 
![핸들러](./_attach/handler_message_dispatch.jpeg)


### 5.3. 메세지의 생성
 Message를 설명할때(`3.3. 메세지의 생성`) 메세지 객체를 생성하는 여러가지 팩토리 메서드가 있다고 했는데, Handler 클래스에는 이를 랩핑하는 메서드가 있다. <br/>
 이를 통해 좀 더 간결하게 메세지 객체를 생성할 수 있다. (메세지의 Handler가 자동으로 연결됨)
 
```
Message obtainMessage()
Message obtainMessage(int what)
Message obtainMessage(int what, Object obj)
Message obtainMessage(int what, int arg1, int arg2)
Message obtainMessage(int what, int arg1, int arg2, Object obj)
```

### 5.4. 메세지의 삽입
 핸들러는 메세지 유형(Data 메세지, Task 메세지)에 따라 여러 방식으로 MessageQueue에 메세지를 추가한다.
 
 Task 메세지는 접두사로 post가 붙은 메서드를 통해 삽입되고, Data 메세지는 접두사로 send가 붙은 메서드를 통해 삽입된다.

 1. MessageQueue에 Task 메세지 추가 <br/>
  `boolean post(Runnable r)` <br/>
  `boolean postAtFrontOfQueue(Runnable r)` <br/>
  `boolean postAtTime(Runnable r, Object token ,long uptimeMillis)` <br/>
  `boolean postAtTime(Runnable r, long uptimeMillis)` <br/>
  `boolean postDelayed(Runnable r, long delayMillis)` <br/>

 2. MessageQueue에 Data 메세지 추가
  `boolean sendMessage(Message msg)` <br/>
  `boolean sendMessageAtFrontOfQueue(Message msg)` <br/>
  `boolean sendMessageAtTime(Message msg, long uptimeMillis)` <br/>
  `boolean sendMessageDelayed(Message msg, long delayMillis)` <br/>
 
 3. MessageQueue에 간단한 Data 메세지 추가 
  `boolean sendEmptyMessage(int what)` <br/> 
  `boolean sendEmptyMessageAtTime(int what, long uptimeMillis)` <br/> 
  `boolean sendEmptyMessageDelayed(int what, long delayMillis)` <br/>  
 
 
참고로 명시적인 uptime이나 delaytime이 설정되어 있어도 각 메세지 처리시간은 여전히 불명확하다. <br/>
처리 시간은 먼저 처리해야 하는 기존 메세지들과 운영체제의 스케줄링에 좌우된다.
 
### 5.5. 메세지 dispatch
 핸들러가 메세지를 처리할때는 메세지의 유형이나 Callback여부에 따라서 다르게 처리된다. <br/>
 아래는 Handler에 있는 dispatchMessage메서드이다. 

> Looper.loop() 코드 중 msg.target.dispatchMessage(msg)에 의해 호출됨.

```
// 이 메서드는 루퍼에 의해 호출된다.
public void dispatchMessage(Message msg) {
	
	// Task 메세지인 경우
	if (msg.callback != null) {
		handleCallback(msg);
	} else {
		// Handler 생성시 Callback 인터페이스를 설정한 경우
		if (mCallback != null) {
			if (mCallback.handleMessage(msg)) {
				return;
			}
		}
		// 일반적인 Handler의 handleMessage() 메서드 호출됨.
		handleMessage(msg);
	}
}

private static void handleCallback(Message message) {
	message.callback.run();
}

public interface Callback {
	public boolean handleMessage(Message msg);
}
    
public void handleMessage(Message msg) { }
``` 

 위 코드를 보면 메세지 dispatch 과정을 직관적으로 알 수 있다. <br/>
 참고로 mCallback은 메세지가 아니라 Handler에 설정된 Callback interface인데 만약 callback이 설정된 경우 이 interface를 통해서도 메세지를 받을 수 있다. <br/>
 이를 이용하면 아래와 같이 Handler 객체의 구현 없이 메세지를 받을 수 있다.
 
```
public class HandlerCallbackActivity extends Activity implements Handler.Callback {
	Handler mUiHandler;
	
	@Override
	public void onCreate(Bundle savedInstance) {
		super.onCreate(savedInstance);
		mUiHandler = new Handler(this);
	}
	
	@Override
	public boolean handleMessage(Message msg) {
		//메세지 처리
		
		// return 값에 따라서 Handler.handleMessage(msg) 메서드의 호출 여부가 결정된다. 
		return true;
	}
}

```
  
 이 방법말고 일반적으로 Handler의 handleMessage(msg)를 이용한 예는 아래와 같다.
 
```
public class MyActivity extends Activity{
	private TextView mTv;
	
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.my_activity);
		mTv = (TextView)findViewById(R.id.mTv);
		
		Log.i("Test", "Thread ID > " + Thread.currentThread().getId());
		WorkThread workThread = new WorkThread(mHandler);
		workThread.start();
	}
	
	// 실제로는 이렇게 하면 memory leak이 발생할 수 있으니 다른 방식으로 구현.
	Handler mHandler = new Handler() {
	    public void handleMessage(Message msg) {
	        // UI 작업 가능 
	        mTv.setText("가능");
	        Log.i("Test", "Thread ID > " + Thread.currentThread().getId());
	    }
	}
	
	class WorkThread extends Thread() {
	    Handler handler;
		 public WorkThred(Handler handler) {
		     this.handler = handler;
		 }
		 
	    public void run() {
	    	 Log.i("Test", "Thread ID > " + Thread.currentThread().getId());
	        Message msg = Message.obtain(handler);
	        handler.sendMessage(msg);
	    }
	}
}

```
  
### 5.6. MessageQueue에서 메세지 제거
 메세지큐에 삽입된 메세지는 루퍼에 의해 처리되기 전에 Handler를 통해서 제거할 수 있다. <br/>

 1. Task 메세지 제거 <br/>
  `removeCallback(Runnable r)` <br/>
  `removeCallback(Runnable r, Object token)` <br/>
  
 2. Data 메세지 제거 <br/> 
  `removeMessages(int what)` <br/>
  `removeMessages(int what, Object object)` <br/>
  
 3. Task 메세지, Data 메세지 모두 제거 <br/> 
  `removeCallbacksAndMessages(Object token)`


Object를 이용하면 일종의 태그 형식으로 메세지큐에 삽입된 여러 메세지를 동시에 삭제할 수 있다.


### 5.7. View와 Activity가 가지는 Handler 
 View와 Activity는 자체적으로 하나의 Handler를 가지고 있다. 따라서 별도로 Handler 생성 없이 MessageQueue 에 메세지를 추가할 수 있다. <br/>
 
 단, Handler의 handleMessage() 처리 방식은 Handler 생성 시에 override하여 정의하기 때문에 이 방법은 사용할 수 없고, Runnalbe 객체를 사용하는 Task 메세지만 사용할 수 있다.

#### 5.7.1. View의 Handler
 View에 사용하는 아래 두가지 메서드가 View의 Handler를 이용해 MessageQueue에 메세지를 추가하는 메서드이다.
 
  1. View.post(Runnable action)
  2. View.postDelayed(Urnnable action, long delayMillis)

이를 이용해서 다음과 같은 코드를 만들 수 있다.

```
TextView mTextView;
Thread thread = new Thread() {
    public void run() {
    
        mTextView.post(new Runnable() {
            public void run() {
                mTextView.setText("텍스트");
            }
        });
        
    }
}

```

 위와 같이 MainThread가 아닌 WorkThread에서 마치 View를 바로 수정하는 것처럼 코딩할 수 있다. <br/>
이런 경우 좀 더 직관적이라는 점에서 장점이 있는데 만약 위와 같이 하나의 View에 대한 수정이 아니라 여러 View를 수정하는 경우라면 별도의 Handler를 만들어 처리하거나 Activity의 Handler를 이용하는 것이 좋다.

> View.getHandler()를 통해 Handler를 직접 참조할 수도 있다.
 
##### (주의) View의 Handler 사용 시 주의사항
 View의 Handler는 Activity lifeCycle에서 onCreate(), onStart(), onResume() 이 모두 호출 된 이후에 참조가 가능하다. 그 이전에 사용하게 되면 에러가 발생하거나 동작하지 않을 수 있다.
 
 
#### 5.7.2. Activity의 Handler 
 Activity에서 사용하는 아래 메서드는 Activity의 Handler를 이용해 MessageQueue에 메세지를 추가하는 메서드이다.
 
 Activity.runOnUiThread(Runnable action)
 
이를 이용해 아래와 같은 코드를 만들 수 있다.

``` 
TextView mTextView, mTextView2;
Thread thread = new Thread() {
    public void run() {
    
        MyActivity.this.runOnUiThread(new Runnable() {
            public void run() {
                mTextView.setText("텍스트");
                mTextView2.setText("텍스트2");
            }
        });
    }
}
```
 
View의 Handler와 달리 액티비티에서 사용하는 여러 View를 갱신하고자 할때 직관적이다.


### 5.8. Handler의 Memory leak 이슈
 아래와 같은 코드는 Memory leak 을 발생시킬 수 있다. 실제 개발 툴에서도 Android lint는 "In Android, Handler classes should be static or leaks might occur." 와 같은 warning을 보여준다.
 
```
public class SampleActivity extends Activity {
 
  private final Handler mLeakyHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
      // ...
    }
  }
 
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
 
    // Post a message and delay its execution for 10 minutes.
    mLeakyHandler.postDelayed(new Runnable() {
      @Override
      public void run() { }
    }, 600000);
 
    // Go back to the previous Activity.
    finish();
  }
}
```
 
 이 코드는 Handler와 Looper, MessageQueue, Message 의 구조상 Memory leak 이 발생할 수 있다.
 
 Memory leak이 발생하는 재현 과정은 Looper의 MessageQueue에 처리할 Message가 들어가있는데 Activity가 종료될 때이다. <br/>
(위 코드와 같이 Handler에게 Message를 보낼때 지연(postDelayed)시켜 보내는 경우도 있고, MessageQueue에 많은 메세지가 들어가있어 아직 처리가 되지 않았을 수도 있다.) 

 MessageQueue에 들어있는 Message는 자신을 전달한 Handler에 대한 reference를 갖고 있고, 현재 위 코드에서 Handler는 Activity 아래 non-static Inner class로 선언되어 있어 Activity에 대한 reference를 가지고 있다. <br/>
 따라서 Activity를 종료했음에도 불구하고, Looper의 MessageQueue에 있는 Message가 처리되기 전까지는 Activity Context가 Gargage Collect 될 수 없다.
 
### 5.9. Handler의 Memory leak을 방지하는 방법
 위 예제에서 Handler는 Activity안에 non-static inner class로 선언되어 있는데, 이를 static inner class로 변경하면 leak을 방지할 수 있다. <br/>
 (static class는 결국 별도로 존재하는 클래스 이기 때문에 Outer class인 Activity의 reference를 가지고 있지 않음)
 
 하지만, Handler를  static class로 만들면 Handler 내부에서 접근할 수 있는 멤버가 Activity의 static 멤버 밖에 없으므로 문제가 된다.
 
 이를 위해 Activity에 대한 참조를 WeakReference를 갖도록 하는 방식으로 수정한다.
 
```
public class SampleActivity extends Activity {
 
  private static class MyHandler extends Handler {
    private final WeakReference<SampleActivity> mActivity;
 
    public MyHandler(SampleActivity activity) {
      mActivity = new WeakReference<SampleActivity>(activity);
    }
 
    @Override
    public void handleMessage(Message msg) {
      SampleActivity activity = mActivity.get();
      if (activity != null) {
        // ...
      }
    }
  }
 
  private final MyHandler mHandler = new MyHandler(this);
 
  /**
   * Instances of anonymous classes do not hold an implicit
   * reference to their outer class when they are "static".
   */
  private static final Runnable sRunnable = new Runnable() {
      @Override
      public void run() { }
  };
 
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
 
    // Post a message and delay its execution for 10 minutes.
    mHandler.postDelayed(sRunnable, 600000);
 
    // Go back to the previous Activity.
    finish();
  }
}
```

--
[참고 문서]
 
> [참고사이트](https://realm.io/kr/news/android-thread-looper-handler/) <br/>
> [참고사이트2(Good)](https://github.com/goznauk/NEXT_Mobile_Backend_201501/wiki/(%EC%9E%84%EC%8B%9C)-%EA%B8%B0%EB%A7%90%EA%B3%BC%EC%A0%9C-&-Android-Threading-Draft)
> [이것이 안드로이드다.]
> [Efficient Android Threading]

