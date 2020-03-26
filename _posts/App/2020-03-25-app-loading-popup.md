---
layout: post
published: false
title: App/Loading Popup
---
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
    debugPrint("Create POPUP");

    stream.takeWhile((ev) {
      return !(ev is ApiErrorState || showUntil(ev));
    }).listen((event) { }, onDone: (){
      debugPrint("DISMISS POPUP");
      Navigator.pop(context);
    });
  }


  @override
  Widget build(BuildContext context) {
    return Dialog(child: ProgressWidget(), backgroundColor: Colors.transparent,);
  }


}

```
