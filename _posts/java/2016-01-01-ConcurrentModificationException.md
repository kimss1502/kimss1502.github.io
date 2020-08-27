---
layout: single
categories: 
  - Java
title: "ConcurrentModificationException"
---

 `Concurrent`는 `동시의` 라는 뜻이고, `Modification`은 `수정` 이란 뜻이다. <br/>
 즉, 객체의 상태를 동시에 변경하는 것을 허용되지 않는데 변경이 발생하여 상태가 깨지는 경우에 발생하는 예외이다.
 
 
## 1. Multi thread 문제만은 아니다.
 동시에 수정이 발생한다라고 하면 Multi thread를 생각할 수 있는데 꼭 Multi Thread가 아니더라도 ConcurrentModificationException 이 발생할 수 있다.
 
 Javadoc API 문서에서도 아래와 같이 설명한다.
 
```
 Single thread 가 객체 접근을 잘못하면 exception을 throw 할 수 있다. 예를 들어 fail-fast Iterator를 가지는 Collection을 반복 처리하는 동안 Collection을 직접 수정하면 이 exception을 throw 한다.
```

 즉, Collection이 Iterator를 통해 순회(Iterate)중인데 Collection의 수정이 발생하면 exception이 발생한다.

 
## 2. fail-fast 란?
### 2.1. Enumeration과 Iterator
 fail-fast를 알기전에 Enumeration과 Iterator에 대해 간단히 알아야 한다.
 
 Enumeration, Iterator 모두 자바에서 제공하는 Collection에 대해 각 항목들을 순차적으로 접근하는데 사용된다.
 
 * Enumeration - 자바 초기버전 개발된 것으로 HashTable과 Vector에서 사용 가능
 * Iterator - JDK 1.2(자바 2)에서 생긴 중요한 변화중 하나가 Collection 클래스들이 Collection framework 으로 통합 관리되는 것이다. Iterator는 기존 Enumeration 기능을 확장한 것으로 Collection interface를 구현한 모든 클래스에서 사용 가능하다.
 
 Enumeration과 Iterator의 가장 큰 차이는 바로 `Snap Shot` 인데 여기서 fail-fast 가 나오게 
된다.

### 2.2. Snap shot
 Enumeration은 snap shot을 사용하고 Iterator는 snap shot을 사용하지 않는다. 이 차이로 인해 fail-fast가 발생한다. <br/>
 스냅샷이란 순간의 상태를 말하는데 Enumeration은 컬렉션 순간의 상태를 따로 저장하고 있기때문에 컬렉션 객체의 변경에 영향을 받지 않은 것이다.
  
### 2.3. fail-fast 
 자바 2 이전의 Enumeration은 fail-fast 방식이 아니지만 Iterator는 fail-fast 방식이다.
 
 컬렉션에 저장된 객체들에 대한 순차적 접근을 제공하는 컬렉션 뷰(Enumeration이나 Iterator)객체는 순차적 접근이 끝나기 전에 컬렉션에 변경이 일어나면 순차적 접근에 실패하게 된다.
 
 이렇게 순차적 접근에 실패할 경우 Enumeration 객체는 실패를 무시하고 끝까지 순차적 접근을 제공한다.(스냅샷이 있기 때문에) 이와 달리  Iterator 객체는 ConcurrentModificationException 예외를 발생한다. (스냅샷이 없기 때문에)
 
 이렇게 순차적 접근에 실패했을때 예외를 발생시키도록 되어 있는 방식을 fail-fast라고 한다.
 
 좀 더 자세히 말하면 컬렉션에 변경이 발생했을때 빠르게 예외를 발생시켜 안전하지 않을지도 모르는 행위 수행을 막는 것이다. 
 

## 3. ConcurrentModificationException 을 신뢰하지 말라
 Javadoc API 문서에 보면 ConcurrentModificationException의 경우 최선의 노력일 뿐이니 버그를 검출하는데만 사용하고 이 exception에 의존해서 개발하지 말라고 되어있다. 
 
 실제로 경우에 따라서 예외가 발생하지 않는 경우가 있는데 이는 Iterator 순환 구조때문에 그렇다.
 내부 구조에 대한 내용은 [내부 구조에 대한 상세내용](http://okky.kr/article/279692) 를 참고한다.
 
--
[참고 문서]
> [ConcurrentModificationException 설명 잘되있음](http://suein1209.tistory.com/323) <br/>
> [Javadoc](https://docs.oracle.com/javase/7/docs/api/)<br/>
> [fail-fast](http://happystory.tistory.com/33)<br/>
> [내부 구조에 대한 상세내용](http://okky.kr/article/279692)<br/>