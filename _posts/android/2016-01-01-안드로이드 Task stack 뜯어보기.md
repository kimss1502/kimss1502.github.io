---
layout: single
categories:
  - 안드로이드
title: "안드로이드 Task stack 뜯어보기"
---

> 안드로이드 Task에 대한 내용은 [안드로이드 Task]({% post_url android/2016-01-01-안드로이드 Task %}) 포스팅을 참고할 것.

 아래 ADB 명령을 통해 현재 기기에 생성된 Task를 확인할 수 있다.

 `adb shell dumpsys activity activities > result.txt`

> Activity의 상태를 보여달라는 명령으로 Activity에 Task 정보가 포함되어 있다.


 아래는 현재 단말에 아무것도 실행된 앱이 없는 상태에서 `kimss.app.tasktest` 앱의 A Activity가 B Activity를 실행시킨 경우에 대한 예이다. (갤럭시 노트5로 테스트)


```
ACTIVITY MANAGER ACTIVITIES (dumpsys activity activities)
Display #0 (activities from top to bottom):
  Stack #1:
    Task id #2374
    * TaskRecord{db853f8 #2374 A=kimss.app.tasktest U=0 sz=2}
      userId=0 effectiveUid=u0a755 mCallingUid=2000 mCallingPackage=null
      affinity=kimss.app.tasktest
      intent={act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] flg=0x10000000 cmp=kimss.app.tasktest/.A}
      realActivity=kimss.app.tasktest/.A
      autoRemoveRecents=false isPersistable=true numFullscreen=2 taskType=0 mTaskToReturnTo=1
      rootWasReset=false mNeverRelinquishIdentity=true mReuseTask=false mLockTaskAuth=LOCK_TASK_AUTH_PINNABLE
      Activities=[ActivityRecord{522ec94 u0 kimss.app.tasktest/.A t2374}, ActivityRecord{e1bd371 u0 kimss.app.tasktest/.B t2374}]
      askedCompatMode=false inRecents=true isAvailable=true
      lastThumbnail=android.graphics.Bitmap@ae180d1 lastThumbnailFile=/data/system/recent_images/2374_task_thumbnail.png
      stackId=1
      hasBeenVisible=true mResizeable=false firstActiveTime=1475299182857 lastActiveTime=1475299210659 lastActiveElapsedTime=1290568837 (inactive for 5s)
      multiWindowStyle=MultiWindowStyle{type=0, zone=ZONE_UNKNOWN, option=0x00000000, bounds=null, isNull=false, isolatedCenterPoint=Point(0, 0), scale=0.0, specificTaskId=-1}
      bHidden=false
      isSecretMode=false
      * Hist #1: ActivityRecord{e1bd371 u0 kimss.app.tasktest/.B t2374}
          packageName=kimss.app.tasktest processName=kimss.app.tasktest
          launchedFromUid=10755 launchedFromPackage=kimss.app.tasktest userId=0
          app=ProcessRecord{9e9b736 11970:kimss.app.tasktest/u0a755}
          Intent { cmp=kimss.app.tasktest/.B bnds=[102,473][1337,1708] }
          frontOfTask=false task=TaskRecord{db853f8 #2374 A=kimss.app.tasktest U=0 sz=2}
          taskAffinity=kimss.app.tasktest
          realActivity=kimss.app.tasktest/.B
          baseDir=/data/app/kimss.app.tasktest-1/base.apk
          dataDir=/data/user/0/kimss.app.tasktest
          stateNotNeeded=false componentSpecified=true mActivityType=0
          compat={560dpi} labelRes=0x7f060020 icon=0x7f030000 theme=0x7f08008e
          config={1 1.0 themeSeq = 0 showBtnBg = 0 450mcc5mnc ko_KR ldltr sw411dp w411dp h707dp 560dpi nrml long port finger -keyb/v/h -nav/h mkbd/h s.263}
          stackConfigOverride={0 1.0 themeSeq = 0 showBtnBg = -1 ?mcc?mnc ?locale ?layoutDir ?swdp ?wdp ?hdp ?density ?lsize ?long ?orien ?uimode ?night ?touch ?keyb/?/? ?nav/? mkbd/?}
          taskDescription: iconFilename=null label="null" color=ff3f51b5
          launchFailed=false launchCount=0 lastLaunchTime=-32s982ms
          haveState=false icicle=null
          state=RESUMED stopped=false delayedResume=false finishing=false
          keysPaused=false inHistory=true visible=true sleeping=false idle=true
          fullscreen=true noDisplay=false immersive=false launchMode=0
          frozenBeforeDestroy=false forceNewConfig=false
          mActivityType=APPLICATION_ACTIVITY_TYPE
          waitingVisible=false nowVisible=true lastVisibleTime=-5s241ms
          multiWindowStyle=MultiWindowStyle{type=0, zone=ZONE_UNKNOWN, option=0x00000000, bounds=null, isNull=false, isolatedCenterPoint=Point(0, 0), scale=0.0, specificTaskId=-1}
          bMultiInstance=false
          mIsLastShownWhenLocked=false
      * Hist #0: ActivityRecord{522ec94 u0 kimss.app.tasktest/.A t2374}
          packageName=kimss.app.tasktest processName=kimss.app.tasktest
          launchedFromUid=2000 launchedFromPackage=null userId=0
          app=ProcessRecord{9e9b736 11970:kimss.app.tasktest/u0a755}
          Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] flg=0x10000000 cmp=kimss.app.tasktest/.A }
          frontOfTask=true task=TaskRecord{db853f8 #2374 A=kimss.app.tasktest U=0 sz=2}
          taskAffinity=kimss.app.tasktest
          realActivity=kimss.app.tasktest/.A
          baseDir=/data/app/kimss.app.tasktest-1/base.apk
          dataDir=/data/user/0/kimss.app.tasktest
          stateNotNeeded=false componentSpecified=false mActivityType=0
          compat={560dpi} labelRes=0x7f060020 icon=0x7f030000 theme=0x7f08008e
          config={1 1.0 themeSeq = 0 showBtnBg = 0 450mcc5mnc ko_KR ldltr sw411dp w411dp h707dp 560dpi nrml long port finger -keyb/v/h -nav/h mkbd/h s.263}
          stackConfigOverride={0 1.0 themeSeq = 0 showBtnBg = -1 ?mcc?mnc ?locale ?layoutDir ?swdp ?wdp ?hdp ?density ?lsize ?long ?orien ?uimode ?night ?touch ?keyb/?/? ?nav/? mkbd/?}
          taskDescription: iconFilename=null label="null" color=ff3f51b5
          launchFailed=false launchCount=0 lastLaunchTime=-33s556ms
          haveState=true icicle=Bundle[mParcelledData.dataSize=680]
          state=STOPPED stopped=true delayedResume=false finishing=false
          keysPaused=false inHistory=true visible=false sleeping=false idle=true
          fullscreen=true noDisplay=false immersive=false launchMode=0
          frozenBeforeDestroy=false forceNewConfig=false
          mActivityType=APPLICATION_ACTIVITY_TYPE
          displayStartTime=-33s231ms startTime=0
          multiWindowStyle=MultiWindowStyle{type=0, zone=ZONE_UNKNOWN, option=0x00000000, bounds=null, isNull=false, isolatedCenterPoint=Point(0, 0), scale=0.0, specificTaskId=-1}
          bMultiInstance=false
          mIsLastShownWhenLocked=false

    Running activities (most recent first):
      TaskRecord{db853f8 #2374 A=kimss.app.tasktest U=0 sz=2}
        Run #1: ActivityRecord{e1bd371 u0 kimss.app.tasktest/.B t2374}
        Run #0: ActivityRecord{522ec94 u0 kimss.app.tasktest/.A t2374}

    mResumedActivity: ActivityRecord{e1bd371 u0 kimss.app.tasktest/.B t2374}
    mLastPausedActivity: ActivityRecord{e1bd371 u0 kimss.app.tasktest/.B t2374}

  Stack #0:
    Task id #176
    * TaskRecord{deafbac #176 A=com.android.systemui U=0 sz=1}
      userId=0 effectiveUid=u0a528 mCallingUid=u0a528 mCallingPackage=com.android.systemui
      affinity=com.android.systemui
      intent={flg=0x10800000 cmp=com.android.systemui/.recents.SeparatedRecentsActivity bnds=[83,1670][1358,2945]}
      realActivity=com.android.systemui/.recents.SeparatedRecentsActivity
      autoRemoveRecents=false isPersistable=false numFullscreen=1 taskType=2 mTaskToReturnTo=0
      rootWasReset=false mNeverRelinquishIdentity=true mReuseTask=false mLockTaskAuth=LOCK_TASK_AUTH_PINNABLE
      Activities=[ActivityRecord{487ad68 u0 com.android.systemui/.recents.SeparatedRecentsActivity t176}]
      askedCompatMode=false inRecents=true isAvailable=true
      lastThumbnail=null lastThumbnailFile=/data/system/recent_images/176_task_thumbnail.png
      stackId=0
      hasBeenVisible=true mResizeable=false firstActiveTime=1474085921036 lastActiveTime=1475299210647 lastActiveElapsedTime=1290568825 (inactive for 5s)
      multiWindowStyle=MultiWindowStyle{type=0, zone=ZONE_UNKNOWN, option=0x01000002, bounds=null, isNull=false, isolatedCenterPoint=Point(0, 0), scale=0.0, specificTaskId=-1, or=3}
      bHidden=false
      isSecretMode=false
      * Hist #0: ActivityRecord{487ad68 u0 com.android.systemui/.recents.SeparatedRecentsActivity t176}
          packageName=com.android.systemui processName=com.android.systemui.recents
          launchedFromUid=10528 launchedFromPackage=com.android.systemui userId=0
          app=ProcessRecord{8c3c075 4366:com.android.systemui.recents/u0a528}
          Intent { flg=0x10800000 cmp=com.android.systemui/.recents.SeparatedRecentsActivity bnds=[83,1670][1358,2945] }
          frontOfTask=true task=TaskRecord{deafbac #176 A=com.android.systemui U=0 sz=1}
          taskAffinity=com.android.systemui
          realActivity=com.android.systemui/.recents.SeparatedRecentsActivity
          baseDir=/system/priv-app/SystemUI/SystemUI.apk
          dataDir=/data/user/0/com.android.systemui
          stateNotNeeded=true componentSpecified=false mActivityType=2
          compat={560dpi} labelRes=0x7f0d024e icon=0x7f020247 theme=0x7f100015
          config={1 1.0 themeSeq = 0 showBtnBg = 0 450mcc5mnc ko_KR ldltr sw411dp w411dp h707dp 560dpi nrml long port finger -keyb/v/h -nav/h mkbd/h s.263}
          stackConfigOverride={0 1.0 themeSeq = 0 showBtnBg = -1 ?mcc?mnc ?locale ?layoutDir ?swdp ?wdp ?hdp ?density ?lsize ?long ?orien ?uimode ?night ?touch ?keyb/?/? ?nav/? mkbd/?}
          taskDescription: iconFilename=null label="null" color=fff5f5f5
          launchFailed=false launchCount=0 lastLaunchTime=-13d19h3m31s799ms
          haveState=true icicle=Bundle[mParcelledData.dataSize=2324]
          state=STOPPED stopped=true delayedResume=false finishing=false
          keysPaused=false inHistory=true visible=false sleeping=false idle=true
          fullscreen=true noDisplay=false immersive=false launchMode=3
          frozenBeforeDestroy=false forceNewConfig=false
          mActivityType=RECENTS_ACTIVITY_TYPE
          waitingVisible=false nowVisible=false lastVisibleTime=-6s268ms
          multiWindowStyle=MultiWindowStyle{type=0, zone=ZONE_UNKNOWN, option=0x01000002, bounds=null, isNull=false, isolatedCenterPoint=Point(0, 0), scale=0.0, specificTaskId=-1, or=3}
          bMultiInstance=false
          mIsLastShownWhenLocked=false

    Task id #102
    * TaskRecord{67fd55f #102 A=com.sec.android.app.launcher U=0 sz=1}
      userId=0 effectiveUid=u0a63 mCallingUid=u0a528 mCallingPackage=com.android.systemui
      affinity=com.sec.android.app.launcher
      intent={act=android.intent.action.MAIN cat=[android.intent.category.HOME] flg=0x10800000 cmp=com.sec.android.app.launcher/com.android.launcher2.Launcher}
      origActivity=com.sec.android.app.launcher/.activities.LauncherActivity
      realActivity=com.sec.android.app.launcher/com.android.launcher2.Launcher
      autoRemoveRecents=false isPersistable=false numFullscreen=1 taskType=1 mTaskToReturnTo=1
      rootWasReset=false mNeverRelinquishIdentity=true mReuseTask=false mLockTaskAuth=LOCK_TASK_AUTH_PINNABLE
      Activities=[ActivityRecord{f248222 u0 com.sec.android.app.launcher/.activities.LauncherActivity t102}]
      askedCompatMode=false inRecents=true isAvailable=true
      lastThumbnail=null lastThumbnailFile=/data/system/recent_images/102_task_thumbnail.png
      stackId=0
      hasBeenVisible=true mResizeable=false firstActiveTime=1474008635870 lastActiveTime=1475298860659 lastActiveElapsedTime=1290218836 (inactive for 355s)
      multiWindowStyle=MultiWindowStyle{type=0, zone=ZONE_UNKNOWN, option=0x00000000, bounds=null, isNull=false, isolatedCenterPoint=Point(0, 0), scale=0.0, specificTaskId=-1, or=1}
      bHidden=false
      isSecretMode=false
      * Hist #0: ActivityRecord{f248222 u0 com.sec.android.app.launcher/.activities.LauncherActivity t102}
          packageName=com.sec.android.app.launcher processName=com.sec.android.app.launcher
          launchedFromUid=0 launchedFromPackage=null userId=0
          app=ProcessRecord{590c70f 4655:com.sec.android.app.launcher/u0a63}
          Intent { act=android.intent.action.MAIN cat=[android.intent.category.HOME] flg=0x10800000 cmp=com.sec.android.app.launcher/.activities.LauncherActivity }
          frontOfTask=true task=TaskRecord{67fd55f #102 A=com.sec.android.app.launcher U=0 sz=1}
          taskAffinity=com.sec.android.app.launcher
          realActivity=com.sec.android.app.launcher/com.android.launcher2.Launcher
          baseDir=/system/priv-app/TouchWizHome_2016/TouchWizHome_2016.apk
          dataDir=/data/user/0/com.sec.android.app.launcher
          stateNotNeeded=true componentSpecified=false mActivityType=1
          compat={560dpi} labelRes=0x7f070002 icon=0x7f02006f theme=0x7f0d0010
          config={1 1.0 themeSeq = 0 showBtnBg = 0 450mcc5mnc ko_KR ldltr sw411dp w411dp h707dp 560dpi nrml long port finger -keyb/v/h -nav/h mkbd/h s.263}
          stackConfigOverride={0 1.0 themeSeq = 0 showBtnBg = -1 ?mcc?mnc ?locale ?layoutDir ?swdp ?wdp ?hdp ?density ?lsize ?long ?orien ?uimode ?night ?touch ?keyb/?/? ?nav/? mkbd/?}
          taskDescription: iconFilename=null label="null" color=ff51b0d3
          launchFailed=false launchCount=0 lastLaunchTime=-14d2h8m28s114ms
          haveState=true icicle=Bundle[mParcelledData.dataSize=24324]
          state=STOPPED stopped=true delayedResume=false finishing=false
          keysPaused=false inHistory=true visible=false sleeping=false idle=true
          fullscreen=true noDisplay=false immersive=false launchMode=2
          frozenBeforeDestroy=false forceNewConfig=false
          mActivityType=HOME_ACTIVITY_TYPE
          waitingVisible=false nowVisible=false lastVisibleTime=-7m54s674ms
          multiWindowStyle=MultiWindowStyle{type=0, zone=ZONE_UNKNOWN, option=0x00000000, bounds=null, isNull=false, isolatedCenterPoint=Point(0, 0), scale=0.0, specificTaskId=-1, or=1}
          bMultiInstance=false
          mIsLastShownWhenLocked=false

    Running activities (most recent first):
      TaskRecord{deafbac #176 A=com.android.systemui U=0 sz=1}
        Run #1: ActivityRecord{487ad68 u0 com.android.systemui/.recents.SeparatedRecentsActivity t176}
      TaskRecord{67fd55f #102 A=com.sec.android.app.launcher U=0 sz=1}
        Run #0: ActivityRecord{f248222 u0 com.sec.android.app.launcher/.activities.LauncherActivity t102}

    mLastPausedActivity: ActivityRecord{487ad68 u0 com.android.systemui/.recents.SeparatedRecentsActivity t176}

    mLastPausedActivity: ActivityRecord{d1b2128 u0 com.android.systemui/.multiwindow.RecentsMultiWindowActivity t2059 f}

  mFocusedActivity: ActivityRecord{e1bd371 u0 kimss.app.tasktest/.B t2374}
  mPersistDownloadablePkgs:
    com.android.systemui
  mFocusedStack=ActivityStack{fb0ae96 stackId=1, 1 tasks} mLastFocusedStack=ActivityStack{fb0ae96 stackId=1, 1 tasks}
  mSleepTimeout=false
  mCurTaskId=2374
  mUserStackInFront={}
  mActivityContainers={0=ActivtyContainer{0}A zone=0, 1=ActivtyContainer{1}A zone=0, 2=ActivtyContainer{2}A zone=12, 3=ActivtyContainer{3}A zone=3}
  mLockTaskModeState=NONE mLockTaskPackages (userId:packages)=
    0:[]
 mLockTaskModeTasks[]
  mCurrentUser=0

GlobalTaskHistory
  ActivityDisplay #0 (1440x2560)
    TASK id #2374	u0	(Stack #1)	kimss.app.tasktest
    TASK id #176	u0	(Stack #0)	com.android.systemui
    TASK id #102	u0	(Stack #0)	com.sec.android.app.launcher

MultiWindow setting
  current

  history
    u0 history[0] - mobile_keyboard : true reason : prev

```

### 1. 전체적인 구조
 현재 전체 구조가 아래와 같다.

```
 Stack #1:
   Task id #2374
     TaskRecord(#2374)
       Hist #1: ActivityRecord(...)
       Hist #0: ActivityRecord(...)
       
   Running activities (most recent first):
     TaskRecord(#2374)
       Run #1: ActivityRecord(...)
       Run #0: ActivityRecord(...)
         
 Stack #0:
   Task id #176
     TaskRecord(#176)
       Hist #0: ActivityRecord(...)
       
   Task id #102
     TaskRecord(#102)
       Hist #0: ActivityRecord(...)
       
   Running activities (most recent first):
     TaskRecord(#176)
       Run #1: ActivityRecord(...)
     TaskRecord(#102)
       Run #0: ActivityRecord(...)

```

#### 1.1 Stack #.. 
 여기서 나타나는 Stack은 어떤 기준으로 나오는 것인지 잘 모르겠다. <br/>
 구조상 Stack 아래에 여러개의 Task가 올 수 있다. <br/>
 
 현재 "Stack #1" 에는 실행시킨 테스트앱의 Task가 들어가있고, "Stack #0" 은 런처앱의 Task가 있다. <br/>
 런처앱에는 2개의 Task가 있는데 각각 "최근앱 목록" 에 해당하는 Task와 "런처 홈"에 해당하는 Task 이다.
  
 현재는 테스트 앱 하나만 띄운 경우인데 여러앱을 띄우고 dump 데이터를 다시 뽑으니 "Stack #1" 에 여러개의 Task가 들어가게 된다. <br/>
 정확히 어떤 기준으로 Stack이 나뉘어지고 Task들이 자리잡는지 모르겠으나 개발자가 임의로 설정할 수 있고 어떠한 의미를 가진다 라는 내용은 보지 못하였다.
 
#### 1.2. Task id #...
 Stack 아래에 있는 Task가 Activity Task의 정보이다. <br/>
 하나의 Task는 별도의 Task ID를 가지고 분류된다.
 
 Task의 순서도 안드로이드에서 의미가 있는 정보이다. <br/>
 상위에 있는 Task가 최근에 실행한 Task이고, 이건 최근 앱 목록에 나오는 순서이기도 하다.
 
#### 1.3. TaskRecord의 Hist #...
 해당 Task의 Activity Stack 정보이다. <br/>
 "Hist #0 > Hist #1 > Hist #2" 와 같이 순서대로 Stack 구조로 쌓인다. <br/>
 최상위에 있는 Hist 정보가 Top Activity이고, 최하단에 있는 Hist #0 이 Root Activity 이다.
 
#### 1.4. Running activities
 해당 Stack 아래에 있는 각 Task에서 실행되고 있는 Activity 정보를 보여준다. <br/>
 그냥 요약해서 보여주는 정보로 큰 의미는 없는것 같다.


### 2. Task 정보 
 Task 아래에 있는 정보는 아래와 같다.
 
```
Task id #2374
* TaskRecord{db853f8 #2374 A=kimss.app.tasktest U=0 sz=2}
  userId=0 effectiveUid=u0a755 mCallingUid=2000 mCallingPackage=null
  affinity=kimss.app.tasktest
  intent={act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] flg=0x10000000 cmp=kimss.app.tasktest/.A}
  realActivity=kimss.app.tasktest/.A
  autoRemoveRecents=false isPersistable=true numFullscreen=2 taskType=0 mTaskToReturnTo=1
  rootWasReset=false mNeverRelinquishIdentity=true mReuseTask=false mLockTaskAuth=LOCK_TASK_AUTH_PINNABLE
  Activities=[ActivityRecord{522ec94 u0 kimss.app.tasktest/.A t2374}, ActivityRecord{e1bd371 u0 kimss.app.tasktest/.B t2374}]
  askedCompatMode=false inRecents=true isAvailable=true
  lastThumbnail=android.graphics.Bitmap@ae180d1 lastThumbnailFile=/data/system/recent_images/2374_task_thumbnail.png
  stackId=1
  hasBeenVisible=true mResizeable=false firstActiveTime=1475299182857 lastActiveTime=1475299210659 lastActiveElapsedTime=1290568837 (inactive for 5s)
  multiWindowStyle=MultiWindowStyle{type=0, zone=ZONE_UNKNOWN, option=0x00000000, bounds=null, isNull=false, isolatedCenterPoint=Point(0, 0), scale=0.0, specificTaskId=-1}
  bHidden=false
  isSecretMode=false
``` 

> Task 내용은 [TaskRecord.java](http://androidxref.com/9.0.0_r3/xref/frameworks/base/services/core/java/com/android/server/am/TaskRecord.java) 코드를 참고하면 된다.

#### 2.1. taskId
 Task를 구분하는 유일 값. (Unique identifier for this task.)

#### 2.2. affinity 
 Task 친밀도를 나타내는 정보. <br/>
 (The affinity name for this task, or null; may change identity.)

#### 2.3. rootAffinity
 Initial base affinity. or null; does not change from initial root.
 
#### 2.4. intent
 이 Task에서 가장 먼저 실행된 Root Activity를 실행시킨 Intent 정보이다.
 (The original intent that started the task.)
 
#### 2.5. affinityIntent
 Intent of affinity-moved activity that started this task.

#### 2.6. realActivity
 The actual activity component that started the daytask.
 
#### 2.7. origActivity
 The non-alias activity component of the intent.
 


### 3. Activity History 정보
 Task 아래 Activity Stack의 각 History가 가지는 정보는 아래와 같다.

```
* Hist #1: ActivityRecord{e1bd371 u0 kimss.app.tasktest/.B t2374}
  packageName=kimss.app.tasktest processName=kimss.app.tasktest
  launchedFromUid=10755 launchedFromPackage=kimss.app.tasktest userId=0
  app=ProcessRecord{9e9b736 11970:kimss.app.tasktest/u0a755}
  Intent { cmp=kimss.app.tasktest/.B bnds=[102,473][1337,1708] }
  frontOfTask=false task=TaskRecord{db853f8 #2374 A=kimss.app.tasktest U=0 sz=2}
  taskAffinity=kimss.app.tasktest
  realActivity=kimss.app.tasktest/.B
  baseDir=/data/app/kimss.app.tasktest-1/base.apk
  dataDir=/data/user/0/kimss.app.tasktest
  stateNotNeeded=false componentSpecified=true mActivityType=0
  compat={560dpi} labelRes=0x7f060020 icon=0x7f030000 theme=0x7f08008e
  config={1 1.0 themeSeq = 0 showBtnBg = 0 450mcc5mnc ko_KR ldltr sw411dp w411dp h707dp 560dpi nrml long port finger -keyb/v/h -nav/h mkbd/h s.263}
  stackConfigOverride={0 1.0 themeSeq = 0 showBtnBg = -1 ?mcc?mnc ?locale ?layoutDir ?swdp ?wdp ?hdp ?density ?lsize ?long ?orien ?uimode ?night ?touch ?keyb/?/? ?nav/? mkbd/?}
  taskDescription: iconFilename=null label="null" color=ff3f51b5
  launchFailed=false launchCount=0 lastLaunchTime=-32s982ms
  haveState=false icicle=null
  state=RESUMED stopped=false delayedResume=false finishing=false
  keysPaused=false inHistory=true visible=true sleeping=false idle=true
  fullscreen=true noDisplay=false immersive=false launchMode=0
  frozenBeforeDestroy=false forceNewConfig=false
  mActivityType=APPLICATION_ACTIVITY_TYPE
  waitingVisible=false nowVisible=true lastVisibleTime=-5s241ms
  multiWindowStyle=MultiWindowStyle{type=0, zone=ZONE_UNKNOWN, option=0x00000000, bounds=null, isNull=false, isolatedCenterPoint=Point(0, 0), scale=0.0, specificTaskId=-1}
  bMultiInstance=false
  mIsLastShownWhenLocked=false       
```

> Activity 내용은 [ActivityRecord.java](http://androidxref.com/9.0.0_r3/xref/frameworks/base/services/core/java/com/android/server/am/ActivityRecord.java) 코드를 참고하면 된다.
 
#### 3.1. packageName
 the package implementing intent's component
 
#### 3.2. processName
 process where this component wants to run
 
#### 3.3. launchedFromPackage
 always the package who started the activity.
 
#### 3.4. app
 if non-null, hosting application
 
#### 3.5. intent
 the original intent that generated us
 
#### 3.6. frontOfTask
 is this the root activity of its task?
 
#### 3.7. task
 the task this is in.

#### 3.8. taskAffinity
 as per ActivityInfo.taskAffinity
 
#### 3.9. realActivity
 the intent component, or target of an alias.


