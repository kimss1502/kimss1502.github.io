---
layout: single
categories: 
  - 디자인패턴
title: "Singleton 패턴"
toc: true
---

## 1. Singleton Pattern 이란
 클래스에 대해서 오직 하나의 instance만을 가지는 클래스이다.
 설정과 같이 전역변수를 관리해야 하는 class가 있다면 쓸때마다 instance를 생성하는 일반적인 클래스가 아니라. 매번 같은 instance가 필요하므로 Singleton 으로 만들 수 있다.
 
## 2. v 만드는 4가지 방법

### 2.1. 방법 1
 가장 간단한 방법으로 class 로드시에 instance가 생성이 된다. <br/>
 코드가 짧고 성능도 좋다. 하지만 class를 사용할때가 아니라 로드시 instance가 생성되므로 실제 사용하지 않을때도 instance가 생성되어 불필요하게 메모리를 낭비할 수도 있다. 

````
public class Singleton {
    private static final Singleton instance = new Singleton();
    
    private Singleton(){
    }
    
    public static SingletonCounter getInstance(){
        return instance;
    }
}
````
    
 이 방법의 경우 instance를 public으로 해도 된다. 뒤에서 나오는 방법과 달리 초기화가 보장이 되기 때문이다. <br/>
 단, 이때는 반드시 final로 선언하는것이 좋다. 만약 상수로 만들지 않으면 외부에서 null가 같이 instance를 변경해버릴수 있기 때문이다. 참고로 권장사항은 아니다.. 


### 2.2. 방법 2
 1번 방법과 달리 class 로드시에 instance가 생성되지 않고, getInstance()가 호출시에 생성된다. <br/>
 synchronized가 있는 이유는 만약 없을 경우 multi-thread 환경에서 타이밍에 따라 여러개의 instance가 생성될 수도 있기 때문이다. <br/>
 synchronized 때문에 성능이 안좋은 단점이 있다.

````
public class Singleton {
    private static Singleton instance;
	
    private Singleton2(){
    }
	
    public static synchronized Singleton getInstance(){
        if (instance == null) {
            instance = new Singleton();
        }
    return instance;
    } 	
}
````
	
### 2.3. 방법 3
 1번의 장점인 성능이 좋다는것과 2번의 장점인 사용할때만 instance를 생성한다는 장점을 합친 형태이다. <br>
 주의해야 할 부분은 (instance == null) 체크를 2군대서 한다는 것이다. <br>
 
 만약 A,B Thread가 동시에 접근을 하여 첫번째 null 체크를 둘다 통과했다고 가정하자.  <br>
 synchronized로 인해 A가 먼저 통과를 했다면 젤 처음엔 instance가 null 이므로 instance를 생성할 것이고, A가 synchronized 를 빠져나오면 다음에 B가 접근하는데 이때는 instance가 null이 아니라 새로운 instnace를 생성하지 않는다. <br>
 
 여기서의 핵심은 2번처럼 synchronized를 처음부터 수행하지 않기때문에 성능이 좋다는 것이고, instance를 getInstance() 내부에서 생성하기 때문에 불필요하게 instance를 생성하지 않는다는 점이다.

````
public class Singleton {
    private volatile static Singleton instance;
	
    private Singleton3(){
    }
	
    public static Singleton getInstance(){
        if (instance == null) {
            synchronized(Singleton3.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
````

 instance를 보면 volatile로 선언이 되어 있는데 volatile 키워드는 변수의 원자성을 보장한다. 자바가 new를 통해 instance를 생성하는 과정은 원자적이지 않다. 그래서 singleton이 보장되려면 volatile로 선언이 되어 있어야 한다고 한다.

### 2.4. 방법4
 SingletonHolder라는 inner class를 사용하는 방법이다. <br/>
 inner class인 Holder클래스가 호출될 때 instance를 생성하므로 3번 방법과 같이 불필요하게 instance가 생성되지도 않고 2번처럼 성능이 느리지도 않다.

````
public class Singleton {
    private Singleton(){
    }
	
    private static class SingletonHolder{
        static final Singleton instance = new Singleton4();
    }

    public static Singleton getInstatnce(){
        return SingletonHolder.single;
    }
}
````

## 3. Singleton의 특징
 - instance가 오직 하나만 생성된다. (제대로 구현했다면..)
 - private 생성자때문에 상속이 불가능하다. (상속받은 하위 클래스 생성시 상위 클래스 생성자가 호출되기 때문에)
 - 자바 api에서 singleton은 대부분 1번으로 구현이 되어 있다고 한다.
	