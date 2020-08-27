---
layout: single
categories: 
  - Java
title: "Singleton 패턴에 대해서"
---

# 3. Singleton 패턴에 대해서
 Singleton은 하나의 객체만 가질 수 있는 Class이다. <br/>
 Singleton을 생성하는 방법은 여러가지가 있다.
 
 
## public final 필드를 이용 
 아래 코드는 Elvis class가 초기화될때 하나만 생성된다.
 
```
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	
	private Elvis() { ... }
}
``` 

## static factory method 사용
 Singleton객체를 만드는 일반적인 방법이다. 객체를 생성하는 방법은 여러가지가 있을 것이다.
 
 이 방법은 첫번째보다 더 유연한 방법인데. 추후 싱글턴 패턴을 포기하고자 할 때 수정이 쉽다.
 
```
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	
	private Elvis() { ... }
	public static Elvis getInstance() {
		return INSTANCE;
	}
}
```

```
public class Elvis {
	private static final Elvis INSTANCE;
	
	private Elvis() { ... }
	public static Elvis getInstance() {
		if(INATANCE == null) {
			INSTANCE = new Elvis();
		}
		return INSTANCE;
	}
}
```

## 싱글턴 객체의 Serializable
 Singleton 객체를 Serializable 객체로 만들고자 할때 단순히 Serializable을 implements 하는것으로 부족하다.
 
 Singleton의 특성을 유지하기 위해서는 모든 필드를 transient로 선언하고 readResolve() 메서드를 추가해야 한다. 그렇지 않으면 직렬화된 객체를 역직렬화 할때마다 새로운 객체가 생성되게 되어 싱글턴의 특징이 깨진다.
 
 참고로 이런 문제를 막으려면 아래와 같은 메서드 구현이 필요하다. <br/>
 그러면 동일한 Instance가 반환되는 동시에 새롭게 생성되는 가짜 Elvis 객체는 GC에 의해 처리된다고 한다.
 
```
private Object readResolve() {
	return INSTANCE;
}
```

## enum 을 활용한 Singleton 객체 생성
 원소가 하나뿐인 enum을 활용하면 가장 완벽환 Singleton 객체를 만들 수 있다고 한다.
 
```
public enum Elvis {
	INSTANCE
}
```
 
 이러면 직렬화가 알아서 처리되어 여러객체가 생길일이 없고, reflection을 통한 공격에도 안전하다고 한다.


### (참고) reflection을 통한 private 생성자 호출 
 Singleton 객체 생성을 위해 private 생성자를 만들었지만, 자바 reflection을 이용하면 private 생성자를 호출할 수 있는 방법이 존재한다.
 
 `AccessibleObject.getAccessible()` 메서드를 통해 권한을 획득한 후 reflection을 이용하면  private 생성자를 호출할 수 있다.
 
