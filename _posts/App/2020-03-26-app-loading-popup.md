---
layout: post
published: true
categories: [App]
tags: [Flutter]
title: Loading Popup
---

뭔가 응답 시간이 긴 API를 호출할때는 화면 인터렉션 전체를 차단해야하고, 그냥 놔두면 ANR이나 진배없으니 뭔가 로딩중이라는 팝업을 띄워야한다.

띄우는건 상관없는데 이놈을 죽이는게 또 일.
프레임워크에 따라 여러가지 방법이 있지만 Bloc을 쓰는 Flutter의 경우 보통 Loading State에 팝업을 띄우고, Loaded State에 pop을 호출하는 방식을 쓸것이다.

나쁘지않지만, 팝업이 겹쳐서 뜬다던가 할때는 관리가 골치아파지기에 팝업 스스로 닫히는 방법을 생각해봤다.

# 코드

```
import 'package:flutter/material.dart';
import 'package:flutter/widgets.dart';
import 'package:vidyarthi_app/base/base_bloc.dart';
import 'package:vidyarthi_app/widget/progress_widget.dart';

class LoadingPopup extends StatelessWidget {
  final BuildContext context;
  final Stream<BaseState> stream;
  final bool Function(BaseState state) showUntil;

  LoadingPopup({this.context, this.stream, this.showUntil}):assert(showUntil != null),
  assert(stream != null) {

    stream.takeWhile((ev) {
      // API Error State는 비정상 응답에 따른 상태이므로 자동 종료처리한다.
      return !(ev is ApiErrorState || showUntil(ev));
    }).listen((event) { }, onDone: (){
      Navigator.pop(context);
    });
  }


  @override
  Widget build(BuildContext context) {
    return Dialog(child: ProgressWidget(), backgroundColor: Colors.transparent,);
  }


}

```

# 사용 예
```
  @override
  void blocListener(BuildContext context, BaseState state) {
    super.blocListener(context, state);
    if (state is GalleryDownloadingState) {
      showDialog(context: context,
          child: LoadingPopup(context: context,
            stream: bloc,
            showUntil: (ev) => ev is GalleryMultiSelectEndState,));
    }
  }
```
