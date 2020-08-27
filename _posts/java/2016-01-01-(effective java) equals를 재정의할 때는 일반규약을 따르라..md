---
layout: single
categories: 
  - Java
title: "equals를 재정의할 때는 일반규약을 따르라."
---


# 8. equals를 재정의할 때는 일반규약을 따르라.
 equals() 메서드는 Override시 실수하기 쉬워 가능한 그대로 사용하는게 좋다. 

## 8.1. equals()를 Override할 필요가 없는 경우
아래 중 하나라도 해당된다면 equals()를 Override 하지 말고 있는 그대로 사용하자.

 1. 객체가 가지는 값(value)에 대한 동일 비교가 아니라 객체 자체에 대한 비교인 경우

 2. 값에 대한 검증(논리적인 동일성)이 있든 없든 상관없는 경우. <br/>
  : 예를 들어 java.util.Random 클래스는 두 Random 객체가 같은 난수열인지 검사하도록 equals() 메서드를 Override 할 수 있었지만 굳이 그렇게 쓸 사용자가 없을것으로 보여 만들지 않음.
 
 3. 클래스가 private 또는 package-private 으로 만들어진 경우. <br/>
  : equals() 메서드가 호출될 일이 없기 때문에 Override 할 필요도 없다. 호출 자체를 막고자 한다면 아래처럼 처리할 수도 있다.

 ```
 @Override
 public boolean equals(Object o){
 	throw new AssertionError();	//호출하면 안되는 메서드 호출
 }
 ```

## 8.2. equals()를 Override 해야 하는 경우
 
 1. 논리적인 동일성(값의 비교) 개념을 지원해 줘야 할 때. <br/>
  : Integer, Date 처럼 value class가 대부분 이에 해당한다. **참고로 이런 value class의 equals()를 적절하게 Override하면 객체 자체를 Map의 Key나 Set의 원소로 사용할 수 있다.** <br/>
  
  반대로 말하면 잘못 Override하면 Map이나 Set에 포함되었을때 예측할 수 없는 결과를 만든다.
   
 2. 상위 클래스의 equals가 하위 클래스의 필요를 충족하지 못할 때
 
 
## 8.3. equals()를 Override할때 규약
 아래는 Object 클래스 명세의 내용이다.
 
```
 equals() 메서드는 동치관계(equivalence relation)를 구현한다. 
 다음의 관계를 동치 관계라 한다.
 
 1. 반사성(reflexive) 
  : null이 아닌 참조 x가 있을때 x.equals(x)는 true를 반환한다.
  
 2. 대칭성(symmetric)
  : null이 아닌 참조 x, y가 있을때, x.equals(y)는 y.equals(x)가 true일때만 true를 반환한다.
  
 3. 추이성(transitive)
  : null이 아닌 참조 x, y, z가 있을때, x.equals(y), y.equals(z)가 true이면 x.equals(z)도 true 이다.
  
 4. 일관성(consistent)
  : null이 아닌 참조 x, y가 있을때 x.equals(y) 호출 결과는 호출 횟수에 상관없이 항상 같아야 한다.
  
 5. null이 아닌 참조 x에 대해서, x.equals(null)은 항상 false를 반환한다.

```
 
### 8.3.1. 반사성
 객체는 자기 자신과 같아야 한다는 것이다. 일부러 깨트리기도 힘들어 별로 문제가 생기지 않는다.
 
### 8.3.2. 대칭성
 두 개의 객체에게 서로 같은지 물어봤을때 둘 다 같은 답이 나와야 한다는 것이다. 실수하면 깨지기 쉬운 규칙이다.
 
```
public final class CaseInsensitiveString {
	private final String s;
	
	public CaseInsensitiveString(String s){
		if (s == null)
			throw new NullPointerException();
		this.s = s;
	}
	
	// 대칭성 위반
	@Override
	public boolean equals(Object o){
		if(o instanceof CaseInsensitiveString) {
			return s.equalsIgnoreCase((CaseInsensitiveString)o).s);
		}
		
		if(o instanceof String) {
			return s.equalsIgnoreCase((String)o).s);
		} 
		
		return false;
	}
}
``` 
 
위 예제는 CaseInsensitiveString 클래스가 일반 문자열 String과도 호환되게 하려고 한다. 

하지만 이 경우 String 클래스가 CaseInsensitiveString 클래스를 모르기 때문에 대칭에 문제가 발생한다.

```
CaseInsensitiveString cis = new CaseInsensitiveString("Aa");
String s = "aa";

> cis.equals(s) 는 true 리턴
> s.equals(cis) 는 false 리턴
```

만약 이걸 Collection에 대입하면 문제가 된다.

```
List<CaseInsensitiveString> list = new ArrayList();
list.add(cis);

// 아래 결과는?? 예측할 수 없다.
list.contains(s);
```

contains()의 결과는 자바 버전이나 JVM에 따라서 달라져서 예측할 수 없게 된다. <br/>
올바르게 equals()를 Override한다면 아래와 같이 해야 한다.

```
@Override
public boolean equals(Object o){
	return o instanceof CaseInsensitiveString && 
		s.equalsIgnoreCase((CaseInsensitiveString)o).s;
}
```

### 8.3.3. 추이성
 역시 실수하면 깨지기 쉬운 규칙이다.
 
 아래 예는 상위클래스에 없는 값을 하위클래스에 추가한 경우이다. 즉, equals()가 비교해야 할 대상이 추가된 경우다.
 
```
public class Point {
	private final int x;
	private final int y;
	public Point(int x, int y){
		this.x = x;
		this.y = y;
	}
	
	@Override
	public boolean equals(Object o){
		if( !(o instanceof Point))
			return false;
			
		Point p = (Point)o;
		return p.x == x && p.y == y;
	}
}

public class ColorPoint extends Point{
	private final Color color;
	public ColorPoint(int x, int y, Color color){
		super(x,y);
		this.color = color;
	}
	
	// 추이성 위반되는 경우.
	@Override
	public boolean equals(Object o){

	}
}
```

#### 8.3.3.1. 대칭성이 위반되는 경우
 아래의 경우는 대칭성에 위반된다. 
  
```
@Override
public boolean equals(Object o){
	if(!(o instanceof ColorPoint))
		return false;
		
	return super.equals(o) && ((ColorPoint)o).color == color;
}
```

 위 경우의 문제는 Point 객체와 ColorPoint 객체를 비교하는 순서를 바꾸면 다른 결과가 반환될 수 있다는 것이다.
 
```
Point p = new Point(1,2);
ColorPoint cp = new ColorPoint(1,2,Color.RED);

p.equals(cp);		// 결과는 true
cp.equals(p);		// 결과는 false
``` 
 
#### 8.3.3.2. 추이성이 위반되는 경우
 ColorPoint가 비교할때는 컬러를 무시하게 해보자.
 
```
@Override
public boolean equals(Object o){
	if(!(o instanceof Point))
		return false;
	
	// o가 Point면 색상 비교하지 않음
	if(	!(o instanceof ColorPoint)){
		return o.equals(this);
	}
	
	// o가 ColorPoint이므로 모든 정보 비교
	return super.equals(o) && ((ColorPoint)o).color == color;
}
```

 위 경우는 대칭성은 보장되지만 추이성이 깨진다.

```
ColorPoint p1 = new ColorPoint(1,2,Color.RED);
Point p2 = new Point(1,2);
ColorPoint p3 = new ColorPoint(1,2,Color.BLUE);

p1.equals(p2);		// 결과는 true
p2.equals(p3);		// 결과는 true
p1.equals(p3);		// 결과는 false - 추이성 위반
```

### 8.3.4. 일관성
 같다고 판정된 객체들은 그 내용이 바뀌지 않는한 늘 같아야 한다. 
 
### 8.3.5. null에 대한 비교
 모든 객체는 null과 비교시 false를 리턴해야 한다. <br/>
 참고로 이를 위해 따로 null체크를 할 필요는 없다. equals() 메서드의 첫부분은 항상 instanceof를 통해 타입체크를 하게 되는데 이 때 체크가 같이 되기 때문이다.

```
Object o = null;
if(o instanceof AA){
	return true;
}else{
	return false;
}
```

위 내용에서 o가 null이면 항상 false가 리턴된다. 


## 8.4. 상속과 equals() 관련 중요한 내용 
 **객체 생성 가능한 클래스를 상속받아 새로운 값 컴포넌트를 추가하면서 equals 규약을 어기지 않을 방법은 없다.** <br/>
 
 만약 새로운 값 컴포넌트가 추가되면서도 equals()를 깔끔하게 Override 하려면 상속받는 대신에 구성(composition)하는 방법을 쓰면 된다. 
 
```
public class ColorPoint{
	private final Point point;
	private final Color color;
	public ColorPoint(Point point, Color color){
		this.point = point;
		this.color = color;
	}
	
	@Override
	public boolean equals(Object o){
		if(!(o instanceof ColorPoint)){
			return false;
		}
		
		ColorPoint cp = (ColorPoint)o;
		return cp.point.equals(point) && cp.color.equals(color);
	}
}
``` 

### 8.4.1. abstract 로 선언된 클래스의 경우 상관없다.
 abstract 로 선언된 클래스의 경우 새로운 값 컴포넌트를 추가해도 equals 규약을 어기지 않고 가능하다. <br/>
 어짜피 상위클래스가 abstract면 직접적인 객체 생성이 불가능하기 때문이다. 


### 8.4.2. 자바 기본 라이브러리 중 잘못 구현된 내용
 java.sql.Timestamp는 java.util.Date를 상속받은 이후에 nanoseconds 필드를 추가하였다. <br/>
 
 이 때문에 Timestamp 클래스의 equals 메서드는 대칭성을 위반하여 Timestamp 클래스와 Date 클래스를 같은 컬렉션에 넣으면 문제가 생길 수 있다. <br/>
 (Timestamp 클래스 주석에 Date와 같이 쓰지 말라고 되어있음)
 

 
 
## 8.5. 훌륭한 equals()를 만드는 지침

### 8.5.1. `==`를 통해 equals()의 인자가 자기자신인지 검사하라.
 성능 최적화를 위해 우선 자기자신인지 확인하자. 객체 비교에 대한 오버헤드가 클 수록  위력이 크다.
 
### 8.5.2. `instanceof`를 통해 인자의 자료형을 검사하라.
 보통은 자기 자신 클래스를 확인하겠지만.. equals()가 구현하는게 인터페이스를 구현하는 여러 클래스에 대해 공통적으로 사용하는 것일수도 있다. <br/>
 (ex- List, Map 과 같은 컬렉션 인터페이스)

### 8.5.3. equals()인자의 정확한 자료형을 변환하라.
 `ColorPoint cp = (ColorPoint)o;` 처럼 명확히 타입 casting하여 사용하여 혹시 모를 버그를 대비하자.

### 8.5.4. 비교 대상 필드의 일치 여부를 검사한다. 
 모두 일치할때 true, 하나라도 아니면 false를 리턴한다. <br/>
 
 1. 기본자료형 : `==` 로 비교
 2. float, double : `Float.compare()`, `Double.compare()`
 3. 객체 : equals()를 재귀적으로 호출하여 비교
 4. 배열 : 배열의 원소 각각 비교

### 8.5.5. 구현을 끝낸 이후 대칭성, 추이성, 일관성 만족 여부를 검토하라.
 단위 테스트로 검사할 것.

### 8.5.6. 추가적인 주의사항
 
 1. equals()를 구현할때는 hashCode()도 재정의하라. <br/>
  hashCode()의 일반 규약이다. (규칙9 참고)
  
 2. equals() 메서드 인자를 Object에서 다른것으로 바꾸지 마라. <br/>
  아래 클래스는 Object.equals()의 Override 한것이 아니라 Overloading 한것이다. 
  
 ```
 public boolean equals(MyClass o){
 
 }
 ``` 


 
 
 
 
 
 
 
 
 
