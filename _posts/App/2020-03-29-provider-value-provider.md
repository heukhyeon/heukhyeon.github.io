---
layout: post
published: true
title: Provider.value와 일반 Provider
categories:
  - App
tags:
  - Flutter
---
플러터에서 Provider를 제공하는 방법은 두가지다.

create Function을 제공하던가, Provider.value를 사용해서 즉시 인스턴스를 만들던가.

이런 경우에는 두가지 방법중에 어떤걸 써야할까.

## MainPageSelector.dart
```
class MainPageSelector extends ChangeNotifier {

  Widget _current = GalleryPageListPage();

  Widget get current => _current;
  set current(Widget value){
    _current = value;
    notifyListeners();
  }

  static MainPageSelector of(BuildContext context) => Provider.of<MainPageSelector>(context,listen: false);
}


//USING
//Scaffold(
//       body: Consumer<MainPageSelector>(
//         builder: (ctx, selector, _) => AnimatedSwitcher(
//             duration: Duration(milliseconds: 300), child: selector.current),
//       ),
//       )
```


MainPage에서 BottomAppBar의 선택지에 따라 위에 렌더링되는 위젯을 스위칭하는 용도의 클래스다.

컨텍스트가 필요없으니 그냥 Provider.value를 썼는데..런타임에는 사실 별 의미가 없어보이는것도 맞다.

문제는 apply Changes. 자꾸 객체가 다시 생성되서 적용전의 위젯 선택 상태가 보존되지 않는다.

혹시나 하는 마음에 함수를...

```
//before
ChangeNotifierProvider.value(MainPageSelector());


//after
ChangeNotifierProvider(create:(ctx)=>MainPageSelector());
```

바꿔보니 apply Changes 를 먹여도 유지가 된다.


배포 레벨에선 별 의미가 없지만 개발단계에서 삐그적 대는 앱이 좋은 앱일거라곤...
