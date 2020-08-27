---
layout: single
categories: 
  - Java
title: "자바의 Reference 형태와 GC"
---


## 1. 자바의 GC(Garbage Collector)
 GC는 프로세스의 힙 영역에 할당된 메모리 중 더 이상 사용되지 않는 메모리를 주기적으로 회수함으로써 프로세스 메모리를 관리한다. <br/>
 자바에서 GC는 다양한 형태가 있지만 공통적으로 다음과 같은 작업을 한다. <br>
 
  1. 힙(heap) 내의 객체 중 garbage를 찾는다.
  2. 찾아낸 garbage를 처리해서 힙의 메모리를 회수한다.

GC가 garbage를 판단하기 reachability 라는 개념을 사용하는데 객체의 참조 형태에 따라 reachability 세분화하여 GC의 동작을 다르게 지정한다.

## 2. GC와 Reachability
 자바의 GC는 reachability 라는 개념을 이용해 garbage를 판단하는데, 어떤 객체에 유효한 참조가 있으면 **reachable(도달 가능한)** , 유효한 참조가 없으면 **unreachable(도달할 수 없는)**로 구별하고 unreachable인 객체를 garbage로 간주해 GC를 처리한다. <br/>
 (reachable 상태는 더 세분화 될 수 있음)

 참고로 reachability 탐색을 위한 최초 객체를 GC root 객체라고 부른다. 즉, GC는 GC root가 도달할 수 없는 객체의 메모리를 회수하는 것이다.

### 2.1. root set
 한 객체는 여러 다른 객체를 참조하고, 참조된 다른 객체들도 마찬가지로 또 다른 객체들을 참조할 수 있으므로 객체들은 참조 사슬을 이룬다. 이런 상황에서 유효한 참조 여부를 파악하려면 항상 유효한 최초의 참조가 있어야 하는데 이를 객체 참조의 root set이라고 한다. (gc root의 집합)

JVM에서 메모리 영역인 런타임 데이터 영역(runtime data area)의 구조를 그림으로 그리면 다음과 같다.

 ![RuntimeArea](./_attach/java_runtime_data_area2.png)
 
> 자바 메모리 구조는 `(Java) JVM과 자바 메모리 구조.md` 참고
 
자바 메모리 구조에서 Runtime Data Area는 5개 영역으로 나뉠 수 있는데 이 중 PC Register를 제외한 4가지 영역이 GC와 관련있다. 

 1. Stack Area
 2. Native Stack Area
 3. Heap Area
 4. Method(Static) Area

화살표는 객체의 참조를 나타낸다. GC root set에 포함될수 잇는 GC root 객체는 Heap 외부에서 접근하는 객체들이다. 

즉, 아래의 3가지 경우가 GC root 객체가 되는 것이다.

 1. Stack Area에서 Heap에 있는 객체에 참조. 즉, 어떤 메서드 실행 시에 사용하는 지역 변수와 파라미터들에 의한 참조
 2. Native Stack Area에서 Heap에 있는 객체에 참조. 즉, JNI(Java Native Interface)에 의해 생성된 객체에 대한 참조
 3. Metthod Area에서 Heap에 있는 객체에 대한 참조. 즉, 클래스의 static 변수에 의한 참조

![reachable,unreachable](./_attach/java_gc_reachable_unreachable.png)

위 그림에서 참조는 `java.lang.ref`패키지를 사용하지 않은 일반적인 참조이며 이를 **strong reference** 라고 부른다.

### (참고) 안드로이드 GC 특징
 GC와 관련된 용어 중 `stop-the-world`라는 용어가 있다. <br/>
 JVM에서 GC가 호출되면 응용프로그램의 실행이 일시적으로 멈추는 현상을 말하는 것이다.
 
 안드로이드 달빅VM 역시 진저브레드까지 메모리가 회수되는 동안 앱의 실행이 중지되었다. <br/>
 이로 인해 GC될때 UI 렌더링이 일시중지되어 사용자 경험을 크게 떨어트리는 요인이 되었다. <br/>
 
 허니콤부터는 GC가 별도의 Thread에서 비동기로 실행된다.

## 3. Soft, Weak, Phantom Reference 
 `java.lang.ref` 패키지는 3가지의 reference 타입을 클래스로 제공하는데 이를 이용하면 개발자가 GC 처리 과정에 영향을 끼칠 수 있도록 한다.

 1. java.lang.ref.WeakReference
 2. java.lang.ref.SoftReference
 3. java.lang.ref.PhantomReference

### 3.1. 용어
 간단하게 WeakReference 객체 생성과정을 보고 용어를 살펴본다.
 
```
Sample sample = new Sample();
WeakReference<Sample> wr = new WeakReference<Sample>(sample);  
Sample ex = wr.get();
...
ex = null; 
``` 

위와 같이 WeakReference 클래스는 Sample 클래스의 instance를 캡슐화한다. <br/>

![weak1](./_attach/java_gc_weakreference1.png)

마지막 줄의 `ex = null;`이후에는 아래와 같이 된다.

![weak2](./_attach/java_gc_weakreference2.png)

위 예에서 용어를 보면 아래와 같다.

1. **reference object** <br/>
 WeakReference, SoftReference, PhantomReference 객체. 위 예에서는 wr

2. **referent** <br/>
 reference object에 의해 참조된 객체로 new()로 생성된 원본 인스턴스. 위 예에서 sample

3. **weakly reachable 객체** <br/>
 `ex = null;` 이후의 상태. 
 
 
### 3.2. Reference와 Reachability
 GC는 Reachability 상태를 통해서 GC 대상을 판단한다. <br/>
 Reachability 상태는 `java.lang.ref`를 사용하면 더 다양하게 분류되는데 전체적으로 총 5가지 상태가 될 수 있다. <br/>
 하나의 객체에 대한 참조는 여러가지 형태의 조합으로 될 수 있고, 다양한 참조 관계에서 아래의 5가지 상태 중 하나가 된다.
 
#### 3.2.1. Strongly reachable 
 참조 사슬 중 ref 패키지 사용 없이 Root set으로 부터 참조된 경우가 있을때. <br/>
 만약 객체에 여러 참조사슬이 있는데 하나라도 root set에서 바로 참조되는 경우가 있으면 strongly reachable 상태이다.

#### 3.2.2. Softly reachable
 SoftReference 로 참조된 경우. <br/>
 strongly reachable 이 아닌 객체 중에서 weak reference, phantom reference 없이 soft reference로만 참조되는 사슬이 하나라도 있는 경우.
    
#### 3.2.3. Weakly reachable
 WeakReference 로 참조된 경우. <br/>
 strongly reachable, softly reachable 이 아닌 객체 중에서 phantom reference 없이 weak reference로만 참조되는 사슬이 하나라도 있는 경우.

#### 3.2.4. Phantomly reachable
 PhantomReference 로 참조된 경우. <br/>
 strongly reachable, softly reachable, weakly reachable 모두 해당되지 않는 경우. <br/>
 이 객체는 finalize 되었지만 아직 메모리가 회수되지 않은 상태이다.
 
#### 3.2.5. Unreachable
 root set으로부터 시작하는 참조가 없는 경우.

![java_reachability](./_attach/java_reachability.png)

--

참조가 여러개 섞여 있을때 상태 판단이 햇갈릴 수 있다. <br/>

![예제](./_attach/java_reachability_ex.png)


위 예에서 B의 상태는 softly reachable 이다. <br/>
만약 왼쪽 하단 root set에서 SoftReference에 대한 참조가 없다면 B는 phantomly reachable 상태가 된다. <br/>
또한 그 상태에서 root set에서 WeekReference에 대한 참조가 있다면 B는 weakly reachable 상태가 된다.

### 3.3. SoftReference와 Softly Reachable
 softly reachable 상태의 객체는 아래 두가지에 의해 GC 여부가 결정된다.
 
 1. Head에 남아있는 여유 메모리 양
 2. 해당 객체의 사용 빈도

즉, 메모리가 많고 사용이 많이 될수록 GC 대상이 되지 않아 오래 살아남는다.

### 3.4. WeakReference와 Weakly Reachable
 weakly reachable 상태의 객체는 특별한 정책이 있는 soft와 달리 GC를 수행할때마다 회수의 대상이 된다. <br/>
 물론 GC가 언제 객체를 회수할지는 GC 알고리즘에 따라 다르고, 수행할때 반드시 모든 메모리를 회수하는 것도 아니다. <br/>
 단지, 바로 회수할 수 있는 상태라고 보면 된다.
 
 참고로, LRU 캐시를 구현할때는 대체로 softly reachable 객체보다 weakly reachable 객체가 쓰인다. <br/>
 softly reference를 사용하게 되면 메모리가 남아있을때동안 GC대상이 되지 않아 메모리 사용량이 점차 늘어나게 되고 나중에 메모리가 부족해지면 GC에 의해 회수되기때문에 GC가 더 자주 발생한다.
 
### 3.5. ReferenceQueue
 PhantomReference를 보기 전에 `java.lang.ref.ReferenceQueue` 를 알아야 한다. <br/>
 
 **SoftReference 객체나 WeakReference 객체가 참조하는 referent 객체가 GC 되어야 한다고 판단되면 Reference 객체 내의 referent를 null로 설정한다. 그렇게되면 해당 객체는 더이상 root set 으로부터의 참조가 없어지므로 자연스레 GC 대상이 된다. 그리고 SoftReference, WeakReference 객체 자체는 ReferenceQueue에 enqueue 된다.** <br/>
 (이 작업은 GC에 의해 자동으로 수행됨)
 
 즉, 이 ReferenceQueue에 reference object가 들어있는지 확인하면 softly reachable 객체나 weakly reachable 객체가 GC되었는지 판단할 수 있고, 이에 대한 후처리 작업을 할 수 있다.
> ReferenceQueue 확인이 아니더라도 referent 객체의 null 여부로 확인해도 되지 않을까?
 
 SoftReference, WeakReference는 이런 ReferenceQueue를 사용할 수도 있고 사용하지 않을수도 있다. 이는 생성자로 구분된다. <br/>
 하지만 PhatomReference 클래스의 경우 반드시 ReferenceQueue를 사용하도록 한다.

```
ReferenceQueue<Object> rq = new ReferenceQueue<Object>(); 
PhantomReference<Object> pr = new PhantomReference<Object>(referent, rq);
``` 
 
 SoftReference, WeakReference는 객체 내부의 참조가 null로 설정된 이후에 ReferenceQueue에 enqueue되지만, PhantomReference는 객체 내부의 참조를 null로 설정하지 않고 참조된 객체를 phantomly reachable 객체로 만든 이후에 ReferenceQueue에 enqueue된다. <br/>

 이를 통해 애플리케이션은 객체의 finalize 이후에 필요한 작업들을 처리할 수 있게 된다. 
  
### 3.6. PhantomReference와 Phantomly Reachable
 phantomly reachable 상태는 기존 두개와 많이 다르다. <br/>
 
 GC는 대상 객체를 찾는 과정과 해당 객체를 처리하는 과정이 연속적이지 않다. <br/>
 마찬가지로 해당 객체를 처리하는 과정과 실제 할당된 메모리를 회수하는 과정 역시 연속적이지 않다. <br/>
 GC는 대상 객체를 처리하는 작업(finalize)이후 GC알고리즘에 따라서 할당된 메모리를 회수한다.
 
 softly reachable, weakly reachable 상태는 GC 대상객체를 판단하는데 관여한다. <br/>
 이와 달리 **phantomly reachable은 finalize와 메모리 회수 사이에 관여**한다.
 
 strongly reachable, softly reachable, weakly reachable에 해당하지 않고 PhantomReference로만 참조되는 객체는 우선 finalize 된 이후에 phantomly reachable로 간주된다. 
 
 GC가 객체를 처리하는 순서는 아래와 같다..
  
  1. strong reference
  2. soft reference
  3. weak reference
  4. finalize
  5. phantom reference
  6. 메모리 회수
 
즉, 어떤 객체에 대해서 GC여부를 판별하는 작업은 이 객체가 strongly reachable 인지, softly reachable 인지, weakly reachable 인지 여부를 순서대로 먼저 판별하고 모두 아니면 finalize를 수행한다. 

이후에 해당 객체를 참조하는 PhantomReference 가 있으면 phantomly reachable로 간주하여 PhantomReference 객체를 ReferenceQueue에 enqueue 한다. 이 때 후처리 작업을 애플리케이션이 수행하도록 하고 메모리 회수는 지연된다.
 
#### 3.6.1. 주의할 점
 PhantomReference의 get() 은 SoftReference, WeakReference와 달리 항상 null을 리턴한다. <br/>
 (내부적으로 유지하고 있는데 null을 리턴한다.)
 
 phantomly reachable 상태로 판단된 객체는 더 이상 사용할 수 없게 된다. <br/>
 (finalize() 이후에 이 상태로 판단되므로 이미 객체에 대해서 정리하는 단계인 것이다.) <br/>
 
 phantomly reachable 상태로 판단된 객체는 객체에 대한 참조를 GC가 자동으로 null 처리를 하지 않는다. <br/> 따라서 후처리 작업 후 반드시 clear() 메서드를 실행해 null로 만들어 줘야 메모리가 회수된다.
 
 
### 3.7. 메모리 leak 회피를 위한 WeakReference 사용 예 
 Memory leak을 피하기 위해서는 그냥 Innner class 대신 Static Inner class를 사용하는 것이 좋다. 그런데 이 경우 Outer class의 멤버에 접근할 수 없는 문제가 발생한다. 이럴 경우 Outer 클래스를 weakReference로 접근하면 된다.

```
public class Outer{
    private int mField;
    
    private static class SampleThread extends Thread{
        private final WeakReference<Outer> mOuter;
        
        SampleThread(Outer outer){
            mOuter = new WeakReference<Outer>(outer);
        }
        
        @Override
        public void run(){
            if(mOuter.get() != null){
                mOuter.get().mField = 1;
            }
        }
        
    }
}
```

-- 
[참고 문서]
> [네이버 블로그](http://helloworld.naver.com/helloworld/textyle/329631) <br/>
> [Efficient Android Threading] 책
