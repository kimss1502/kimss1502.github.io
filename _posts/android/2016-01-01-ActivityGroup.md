---
layout: single
categories: 
  - 안드로이드
title: "ActivityGroup에 대해서"
---

> ActivityGroup은 이미 오래전 deprecated 되었다. <br>
> ActivityGroup과 관련한 내용은 [Fragment에 대해서](https://kimss1502.github.io/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C/Fragment%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C/) 포스팅을 함께 참고할 것.

 ActivityGroup은 하나의 Activity에 여러개의 Activity를 포함할 수 있게 해준다. ViewGroup이 View를 가질수 있듯 ActivityGroup은 Activity를 가질 수 있다. </br>
 
 참고로 ActivityGroup 역시  Activity이다. 여러 Activity를 포함할 상위 Activity는 ActivityGroup을 상속받는다. </br>
 
 ActivityGroup을 사용하면 하나의 화면에 전환해야할 뷰가 여러개 존재할 때 각각의 뷰를 별도의 액티비티 단위로 나눌 수 있기 때문에 코드 관리하기가 좋다.
 

## 1. ActivityGroup내에 Activity를 추가하는 방법
```java
// 1. 액티비티그룹을 상속받음으로써 자식 액티비티를 관리할 수 있게 된다.
public class MainActivityGroup extends ActivityGroup {
 
	FrameLayout mListMenuLayout     = null;
	FrameLayout mViewerLayout   = null;
   
	// 2. 액티비티그룹의 자식 액티비티를 관리하는 로컬 액티비티매니저를 참조한다.
	LocalActivityManager mLocalActivityMgr = getLocalActivityManager();
   
	// 3. 액티비티그룹도 액티비티를 상속받았기 때문에 액티비티 생명주기 함수가 모두 존재한다. 
	//   따라서 액티비티그룹도 onCreate 생명주기 함수가 최초 호출된다.
	protected void onCreate(Bundle savedInstanceState)
	{
	    super.onCreate(savedInstanceState);
	    setContentView(R.layout.activity_group_layout);
	   
	    // 4. 액티비티가 들어갈 뷰 참조
	    // 리스트 메뉴 영역의 레이아웃 참조
	    mListMenuLayout = (FrameLayout)findViewById( R.id.list_menu_layout );
	    // 뷰어 영역의 레이아웃 참조
	    mViewerLayout = (FrameLayout)findViewById( R.id.viewer_layout );
	   
	    // 5. 화면 좌측에 리스트 메뉴 액티비티 실행
	    // 메뉴 리스트 액티비티를 실행한다.
	    Intent menuActivityIntent = new Intent( this, ListMenuActivity.class ); 
	    Window menuWindow = mLocalActivityMgr.startActivity( "ListMenuActivity", menuActivityIntent );
	   
	    // 실행된 메뉴 액티비티의 레이아웃을 참조하여 액티비티그룹 좌측영역에 추가한다.
	    View menuView = menuWindow.getDecorView();
	    mListMenuLayout.addView( menuView );
	
	    // 6. 리스트 메뉴 아이템이 클릭되었을 때 호출되는 리스너를 등록한다.
	    ListMenuActivity listActivity = (ListMenuActivity)mLocalActivityMgr.getActivity( "ListMenuActivity" );
	    listActivity.setOnListItemClickListener( new ListMenuActivity.OnListItemClickListener()
	    {
	        @Override
	        public void onItemClick( Intent viewerIntent, String viewerId )
	        {
	            // 클릭된 아이템의 해당하는 액티비티를 넘겨받은 인텐트로 실행한다.
	            Window viewerWindow = mLocalActivityMgr.startActivity(
	                                                viewerId, viewerIntent );
	           
	            // 실행된 액티비티 레이아웃을 참조하여 액티비티그룹 우측영역에 추가한다.
	            View    viewerView  = viewerWindow.getDecorView();
	            mViewerLayout.removeAllViews();
	            mViewerLayout.addView( viewerView );
	        }
	    });
	}
}

```
    
  1. 컨테이너격인 ActivityGroup을 상속받는 Activity에 추가할 액티비티의 View들이 보일 영역을 만든다. 
  2. 추가할 Activity를 LocalActivityManager를 통해 실행한다. 
  3. 생성된 Activity의 View를 얻어낸다.
  4. 해당 View를 원하는 위치에 Add한다.
  5. 추가한 액티비티에서 같은 화면상의 다른 액티비티를 실행 또는 조작하려면 컨테이너인 ActivityGroup을 상속받은 Activity에서 해야 하므로 Listener를 달아준다.

> 참고로 ActivityGroup에 Activity를 포함하여 보여주는것은 Activity라는 컴포넌트가 추가된 형태가 아니다. 그냥 액티비티의 뷰 영역을 가져와 현재 뷰컨테이너에 Add하는 것이다. 


## 2. ActivityGroup의 장점
 결국 ActivityGroup을 사용하고자 하는 것은 코드를 잘 분리하여 각각의 컴포넌트로 관리하기 위함일 것이다. ActivityGroup안에 Activity를 포함한다는 것은 이러한 점에서 매우 편리하다. </br>
 심지어 개별 Activity로 동작하면서 ActivityGroup안에도 넣을 수 있다는 점은 큰 장점이다.
 
 
## 3. ActivityManager 란.
 원래 ActivityManager는 안드로이드 시스템 서비스이다. 안드로이드에서 모든 액티비티는 ActivityManager가 관리를 한다. 여기서 관리란 액티비티의 생명주기 및 테스크 관리 등을 말한다.
 그리고 각 액티비티들은 서로 다른 프로세스이기 때문에 ActivityManager에서 Activity간의 데이터 통신에는 바인더 통신을 한다. </br>
 
## 4. LocalActivityManager 란.
 LocalActivityManager는 ActivityGroup이 가지고 있다. ActivityGroup내에 속한 액티비티들을 관리하는 내부 관리자인것이다. ActivityManager는 다른 프로세스에 있는 액티비티들을 관리하기 때문에 바인더 통신을 하는데, LocalActivityManager는 같은 프로세스 내부이기 때문에 바인더 통신을 하지 않는다.
 
## 5. ActivityGroup과 Activity의 문제
 ActivityGroup은 API 13부터 deprecated 되었다. 그 이유는 여러가지가 있는데 간단하게 말하면 다음과 같다.
 
 1. ActivityGroup역시 Activity인데 기존 Activity와 다르게 동작하는 부분이 생김.
 2. ActivityGroup 내부에 포함되는 Activity의 개념이 모호해짐.

 
### 5.1. ActiityGroup의 문제
 안드로이드의 모든 Activity는 시스템 서비스인 ActivityManager에 의해 관리된다. Activity의 일종인 ActivityGroup 역시 마찬가지다. </br>
 
 ActivityGroup은 내부의 액티비티를 관리하기 위한 LocalActivityManager를 가지고 있다. 따라서 ActivityGroup내에서 액티비티 관리는 안드로이드 서비스인 ActivityManager가 아닌 LocalActivityManager의 영향을 받는다. </br>
 
 원래 안드로이드에서는 타 앱(타 앱의 액티비티)를 실행시킬 수 있고, 이건 ActivityManager가 바인더 통신을 하기 때문에 가능한 것이다. 그런데 LocalActivityManager는 바인더 통신을 하지 않는다. 따라서 ActivityGroup은 다른 액티비티를 실행시키거나 포함할 수 없다.
 
> 만약 타 액티비티를 실행하거나 포함하려고 하면 java.lang.SecurityException 이 발생한다.

### 5.2. ActivityGroup안에 포함된 Activity 개념 문제
 A라는 Activity 가 있는데 이 Activity가 ActivityGroup안에 포함되었을 때 과연 이것도 Activity라고 말을 할 수 가 있는가? </br>
 ActivityGroup내에 포함된 A는 그 안에서 레이아웃과 생명주기만을 지원하기에 완벽한 Activity라고 하기 모호하다. (기존 Activity의 속성은 모두 ActivityGroup에 귀속되어 버리기 때문에 ..)
 
 예를 들어 여러 액티비티를 실행하면 하나의 테스크가 생성되고 내부에 순서대로 액티비티 스택이 형성된다.
 그런데 ActivityGroup안에서 순서대로 Activity를 생성하여 추가한다고 해서 액티비티 스택이 형성되지 않는다. (테스크 자체를 관리하지 않는다.)
 
 또한 액티비티 스택이 존재하지 않기 때문에 액티비티간 데이터 전달도 불가능 하다. (startActivitForResult 로 데이터 전달 못함)
 

## 6. Fragment 의 탄생 이유 

**사실 ActivityGroup안에 있는 Activity에게 바라는 것은 많지 않다!!** </br>
 필요한 것은 별도의 소스로 분리된 액티비티의 레이아웃과 생명주기가 필요할 뿐!!
 
단지 이것을 위해 무거운 Activity를 ActivityGroup안에 넣는것은 효율적이지도 않고, 개념도 모호하다.  

Fragment는 바로 이러한 요건을 충족시키기 위해서 탄생한 것이다. 

 
 

 