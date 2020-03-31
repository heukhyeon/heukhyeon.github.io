---
layout: post
published: false
title: Flutter Light Dark Brightness
---
안드로이드건, IOS건 상단 StatusBar는 존재한다. 그리고 이 둘의 컨텐츠 (안드로이드는 아이콘, IOS는 일반적으로 현재 시간, 네트워크 상태 등)를 표시하는 색도 Black (안드로이드는 좀 많이 미묘하지만) 이나 White로 동일하다.
![white_status.PNG]({{site.baseurl}}/img/white_status.PNG)
![dark_status.PNG]({{site.baseurl}}/img/dark_status.PNG)

Flutter에서 이 표시색을 바꾸는 방법은 세가지 정도인데.

### 1. PageRouteBuilder return Widget에 AnnotatedRegion Wrapping.
```
MaterialApp(
        routes: {
          '/' : (ctx) => AnnotatedRegion<SystemUiOverlayStyle>(
              value: SystemUiOverlayStyle.dark,
              child:SplashPage()),
        },
      )
```
- route 정의를 App 레벨에 할경우 변동이 있는 부분마다 일일히 AnnotatedRegion을 달아줘야해서 귀찮다.

### 2.  SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle.(dark or light));
- 보통 일반적으로 나오는 대답.
- transparent status bar를 만드려면 어쩔수없이 써야한다.
- 이게 시스템 레벨에 작업 거는거라 그런지, 개발중에 이거 설정하고 지운다음에 다시 로딩하면 적용된채로 남아있다.


### 3. AppBar 생성자에 Brightness 주입
- 3과 더불어 가장 많이 쓰이는 방법.
- 그리고 언제나 따라오는 '나 앱바 안쓸건데 그럼 어떻게 함?' 이라는 질문.

방법들에 대해서는 딱히 길게 적을게 없고, 내가 쓰려는건 좀 해맨게 있어서다.



![white_status.PNG]({{site.baseurl}}/img/white_status.PNG)

이 이미지를 보자, 이 이미지의 status bar icon의 Brightness는 뭘까?

IOS를 생각하면 자연스럽게 dark라 생각하게 될텐데....대체 왜 인지 알수없지만, light다.

![dark_status.PNG]({{site.baseurl}}/img/dark_status.PNG)

그러니 이것도 색만 봐선 light라 생각하겠지만, dark다.

IOS는 '해당 상태일때 표시되기 적절한' 것에 대한 정의고,
안드로이드는 '해당 상태로 표시된 ' 것에 대한 정의인지 뭔지...

하나 짐작이 드는건 안드로이드의 경우 light status bar ( 아이콘이 검게 나오는) 자체가 dark status bar ( 아이콘이 희게 나오는) 것보다 나중에 나오다보니 IOS랑 반대로 간건가 싶기도.


...그런데 웃기는게, 이게 또 3에는 적용되지 않는다. 1도 사실상 2와 같은 맥락이니 SystemUIOverlayStyle의 문제라는것.

```
AppBarTheme(
            brightness: Brightness.dark
          )
```

AppBar의 brightness의 경우 light 일경우 아이콘이 검게 나오고, dark 일경우 하얗게 나온다.

뭐야 이게?


찾다보니 [이미 있는 이슈다.](https://github.com/flutter/flutter/issues/17523) 근데 닫혔다 나오는데?


```
[√] Flutter (Channel stable, v1.12.13+hotfix.8, on Microsoft Windows [Version 10.0.18362.720], locale ko-KR)
    • Flutter version 1.12.13+hotfix.8 at C:\Users\heukh\Documents\flutter\flutter
    • Framework revision 0b8abb4724 (7 weeks ago), 2020-02-11 11:44:36 -0800
    • Engine revision e1e6ced81d
    • Dart version 2.7.0

```

나중에 '내가 할땐 제대로 되는데 왜 딴소리임?'이라 할것 같으니 버전을 적어둔다.

[이 이슈](https://github.com/flutter/flutter/issues/41256) 에서는 또 반대로 작동하는 모양이니 내가 이상한것 같기도 하고....


뭐여 이게...

이게 됫든 저게 됫던 AppBar를 먹이면 의도한대로 돌아간다. 앱바가 없을때 문제지...

어떤 글에선 height 0짜리 앱바를 만드는 글도 봣는데 참...






    

  
  
