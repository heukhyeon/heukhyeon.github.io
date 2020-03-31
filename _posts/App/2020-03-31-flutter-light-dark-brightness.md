---
layout: post
published: true
title: Flutter Light Dark Brightness
categories:
  - App
tags:
  - Flutter
image: '{{site.baseurl}}/img/white_status.PNG'
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
- PreferredSize(height:0 child:AppBar())라는 엽기적인 놈도 봤다.

방법들에 대해서는 딱히 길게 적을게 없고, 내가 쓰려는건 좀 해맨게 있어서다.



![white_status.PNG]({{site.baseurl}}/img/white_status.PNG)

brightness가 '테마에 맞는' 값이라 생각해서 위 이미지의 경우 brightness가 dark일거라 생각했고



![dark_status.PNG]({{site.baseurl}}/img/dark_status.PNG)

마찬가지로 밝은 화면에 맞는 색이니 light일거라 생각했는데, 반대라는것.


코드를 보면 사실 이해가 되긴 한다.

## system_chrome.dart (Flutter Offical)
```
  /// System overlays should be drawn with a light color. Intended for
  /// applications with a dark background.
  static const SystemUiOverlayStyle light = SystemUiOverlayStyle(
    systemNavigationBarColor: Color(0xFF000000),
    systemNavigationBarDividerColor: null,
    statusBarColor: null,
    systemNavigationBarIconBrightness: Brightness.light,
    statusBarIconBrightness: Brightness.light,
    statusBarBrightness: Brightness.dark,
  );

  /// System overlays should be drawn with a dark color. Intended for
  /// applications with a light background.
  static const SystemUiOverlayStyle dark = SystemUiOverlayStyle(
    systemNavigationBarColor: Color(0xFF000000),
    systemNavigationBarDividerColor: null,
    statusBarColor: null,
    systemNavigationBarIconBrightness: Brightness.light,
    statusBarIconBrightness: Brightness.dark,
    statusBarBrightness: Brightness.light,
  );
```

오버레이를 '그리는' 색상이 light인지 dark인지를 정의한다. 




재밌는건 AppBar는 이게 또 반대다. 얘는 내가 생각한대로 '화면 밝기에 맞게끔' status bar를 조정한다.

```
AppBarTheme(
            brightness: Brightness.dark
          )
```

AppBar의 brightness의 경우 light 일경우 아이콘이 검게 나오고, dark 일경우 하얗게 나온다.

...테마라는건 참 알다가도 모르겠다.
