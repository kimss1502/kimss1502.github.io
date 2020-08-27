---
layout: single
categories: 
  - 안드로이드
title: "TouchEvent에 대해서"
---

## TouchEvent
 터치 이벤트는 3가지(Down, Move, Up)의 이벤트를 감지한다. <br/>
 이 3가지는 정확히 순서대로 일어나며 하나의 프로세스로 간주한다. 사용자가 한번 터치했다고 하는 것은 이 프로세스를 한번 거쳤다는 것이기 때문이다.

### 1. TouchEvent 전달 과정
 유저의 터치로부터 실제 터치이벤트를 처리할 Activity나 View에 전달되는 과정은 아래와 같다.
 
 1. 사용자가 화면을 터치
 2. 터치 디바이스 드라이버가 이벤트를 감지하고 시스템 서비스인 WindowManager에게 전달
 3. WindowManager는 화면에 떠 있는 현재 앱의 Activity에게 이벤트 전달
 4. Activity의 터치 영역에 View가 있다면 해당 View에 이벤트 전달
 5. 이벤트 소모

특히 해당 앱의 Activity에서 View로 이벤트가 전달되는 과정은 아래와 같다.

 1. 터치가 발생된 View를 찾기 위해 View의 root부터 하위로 탐색한다.
 2. 터치 영역에 해당하는 View를 찾으면 전달된 이벤트를 소모한다.
 3. 만약 터치 영역의 View가 해당 이벤트를 소모하지 않으면 그 View를 parent view에게 넘긴다.

### 2. 주요 메서드
 이벤트 전달 과정에서 Activity나 View가 이벤트를 전달받는 메서드는 2개가 있다. <br/>
 아래 두개의 메서드는 Activity와 View(View와 ViewGroup)에 모두 있다.
 
#### 2.1. dispatchTouchEvent

```
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
	return super.dispatchTouchEvent(ev);
}
```

 Activity나 View가 이벤트를 제일 처음 전달받는 곳이자 이벤트를 하위 View에 전달하는 역할을 한다. <br/>
 
#### 2.2. onTouchEvent

```
@Override
public boolean onTouchEvent(MotionEvent ev) {
	return super.onTouchEvent(ev);
}
```

 실제 전달된 이벤트를 처리하여 소비하는 곳이다. <br/>
 true를 리턴하면 해당 View가 이벤트를 소모한 것이고, false를 리턴하면 해당 View의 parent View나 Activity가 이벤트를 소모할 수 있도록 권한을 넘겨준다. <br/>
 만약 현재 View에서 이벤트를 소모하면 parent view나 Activity는 onTouchEvent가 호출되지 않는다.

![이벤트전달](./_attach/touch_event_flow.png) 

dispatchTouchEvent 는 부모 메서드 `super.dispatchTouchEvent(ev)` 를 호출해주어야 한다. 그렇지 않으면 자식 View는 `super.dispatchTouchEvent` 와 `onTouchEvent` 를 받을 수 없다.

### 3. MotionEvent

| 메서드 | 설명 | 
| --- | --- |
| getAction() | 터치 이벤트의 액션값. <br/>`ACTION_DOWN = 0`, `ACTION_UP = 1`, `ACTION_MOVE = 2`, `ACTION_CANCEL = 3` |
| getX() | 이벤트 발생 x 좌표 | 
| getX() | 이벤트 발생 y 좌표 | 
| getEventTime() | 이벤트 발생 시간 ms | 
| getDownTime() | Down 이벤트가 발생한 시간 ms | 


### 4. 터치 다운 이벤트
 터치 이벤트는 3가지(Down, Move, Up)가 있고 하나의 프로세스라고 했다. <br/>
 이 중 첫번째 동작인 Down 이벤트는 이벤트 전달 목적지를 결정하는 역할을 한다.
 
 
![터치다운](./_attach/touch_down.png) 
 
 위와 같이 onTouchEvent를 소비하는 곳이 ViewGroup이라고 가정하자. <br/>
 그러면 Down 다음 동작인 Move와 Up의 경우 View까지 가지도 않는다. <br/>
 즉, Down이후 Move와 Up의 동작에서는 View의 `dispatchTouchEvent`와 `onTouchEvent`가 아예 호출되지 않는다. 
 
 
### 5. 터치 이벤트 인터셉트
 `onInterceptTouchEvent` 메서드는 자식 View로 전달되는 이벤트를 부모 ViewGroup이 가로챌 수 있도록 한다. <br/>
 그리고 만약 가로채진 경우 이를 자식 View가 알 수 있도록 `ACTION_CANCEL` 이라는 이벤트를 `onTouchEvent` 메서드를 통해 전달해준다.
 
 사용하고자 하는 경우 이 메서드를 Override하면 되는데 Activity는 사용할 수 없다. <br/>
 당연한게.. 액티비티가 이벤트를 가로채버리면 밑에 View가 할 수 있는게 아무것도 없기 때문이다.

 이벤트를 부모가 인터셉트 하는 경우는 아래와 같은 경우 필요하다.
 
 ![인터셉트](./_attach/touch_event_intercept.png)
 
 그림과 같이 ScrollView를 꽉채우는 Button이 있을때 Button이 터치를 잡게 되는데 이 때문에 스크롤을 할 수 없는 상황이 발생할 수 있다. 
 

### 6. 터치 이벤트 인터셉트 방지
 부모 View는 `onInterceptTouchEvent`를 통해 자식 View에게 전달될 이벤트를 가로챌 수 있다. <br/>
 반대로 자식 View는 `requestDisallowInterceptTouchEvent` 메서드를 통해 부모 View가 이벤트를 가로채지 못하도록 요청 할 수 있다. <br/>
 단, 주의할 점은 한번의 터치 프로세스에서만 유효하다는 것이다. 계속 필요하다면 매 터치가 발생할때마다 메서드를 호출해 줘야 한다.
 
 
### 7. 이벤트 리스너
 View의 이벤트를 받기 위해 모든 View를 CustomView로 만들어 `onTouchEvent`를 상속받을 수는 없다. <br/>

 View가 전달받는 이벤트를 위해 `OnTouchListener` 가 존재하는데 View에 리스너가 설정되 있을 경우 `onTouchEvent`를 호출하지 않고, `OnTouchListener.onTouch(View v, MotionEvent ev)` 를 호출해준다.
 
> 참고로 `dispatchTouchEvent`, `onIntercepptTouchEvent` 를 위한 리스너는 없음. 이를 사용하기 위해서는 무조건 View/ViewGroup을 상속받아 커스텀으로 만들어야 한다.
  
  






