---
layout: single
categories: 
  - Java
title: "equals를 재정의할 때는 반드시 hashCode도 재정의하라."
---


# 9. equals를 재정의할 때는 반드시 hashCode도 재정의하라.

 equals() 메서드를 Override 하는 클래스는 반드시 hashCode() 도 Override해야 한다. 그렇지 않으면 `Object.hashCode()` 의 일반규약을 어기게 되어 HashMap, HashSet, Hashtable 과 같은 해시 기반 컬렉션을 사용할 때 오동작하게 된다.
 
## 9.1. hashCode() 일반 규약

 1. 응용프로그램 실행 중 같은 객체의 hashCode를 여러번 호출하는 경우, equals가 사용하는 정보들이 변경되지 않았다면 언제나 동일한 정수(integer)가 반환되어야 한다. 다만 응용프로그램이 종료되었다가 다시 실행되어도 같은 값이 나올 필요는 없다.

 2. equals() 메서드가 같다고 판정한 두 객체의 hashCode값은 같아야 한다.

 3. equals() 메서드가 다르다고 판정한 두 객체의 hashCode값이 꼭 다른 필요는 없다. 그러나 서로 다른 hashCode 값이 나오면 Hashtable의 성능이 향상될 수 있다.

> 2번째 규약때문에 equals() 메서드 override시 hashCode()도 override 해야 한다.

## 9.2. 잘못되는 예

### 9.2.1. override하지 않아 문제가 되는 경우
 PhoneNumber 라는 클래스가 있는데 number로 equals() 여부를 리턴해주도록 Override 하였다. <br/>

```
Map<PhoneNumber, String> m = new HashMap<PhoneNumber String>();
m.put(new PhoneNumber(707, 867, 5309), "Jenny");

// name은 어떤 값이 올 것인가.
String name = m.get(new PhoneNumber(707, 867, 5309));
```

위 코드에서 2개의 PhoneNumber 객체가 사용되었다. <br/>
그런데 equals() 메서드를 override 했기때문에 논리적으로 2개는 같은 객체이다. 하지만 리턴되는 name은 Jenny가 아니라 null 이다.

**hashCode()를 override 하지 않아 두 객체는 서로 다른 해시코드를 갖기때문이다. 따라서 get메서드는 put이 저장한것과 다른 해시 버킷(hash bucket)을 뒤지게 된다.**

### 9.2.2. 잘못 구현하여 문제가 되는 경우

```
@Override
public int hashCode() {
	return 42;
}
``` 

이런식으로 구현해도 동작은 한다. 하지만 모든 객체가 동일한 hashCode()값을 가지게 되어 Hashtable을 쓸때 모두 같은 버킷에 해시되고.. 결국 Hashtable은 LinkedList가 되어버린다. 

동작은 하지만 실행시간 O(N)이 되어야 하는것이 O(N^2)이 되어버려 성능이 엄청 느려 진다.

## 9.3. hashCode()의 올바른 구현
 1. 좋은 hash함수는 다른 객체에 대해서 다른 hashcode를 리턴한다.(3번째 규약의 내용)
 
 2. 다른 필드 값에 의해 유도될 수 있는 값은 hashcode 계산에서 제외해도 된다. 

 3. equals 계산에 쓰이지 않는 필드는 반드시 제외해야 한다. (2번째 규약때문)

 4. immutable 클래스인 경우 hashcode 계산 비용이 높다면 객체 내부에 캐쉬해 돌 수 있다.

 5. 성능을 개선하려고 객체의 중요 부분을 hashcode 계산에서 빼면 안된다. 그럴 경우 hash함수 속도가 빠를순 있어도 hashtable 성능은 엄청 떨어트릴 수 있다. 


