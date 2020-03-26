---
layout: post
published: false
title: Private Image
categories:
  - App
---
저녁 시간대에만 이미지를 보여주고, 아닐때는 대체할 이미지를 보여주고싶다.

대충 5분마다 한번씩 체크해서 유효 시간인경우 이미지를 보여주고, 아니면 감추는 식으로.


# secret_image.dart

```
import 'package:flutter/widgets.dart';
import 'package:provider/provider.dart';

class SecretImage extends StatelessWidget {

  final Widget child;

  const SecretImage({Key key, this.child}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Consumer<SecretImageNotifier>(
      builder: (_,notify,_) {
        if(notify.hide) return Container();
        else return child;
      },
    );
  }

}

class SecretImageNotifier extends ChangeNotifier {

  bool hide = false;

  SecretImageNotifier(){
    _refresh();
    new Timer.periodic(Duration(minutes: 5), (timer) {
       _refresh();
    });
  }

  void _refresh(){
    final hour = DateTime.now().hour;
    final hide = hour > 9 && hour < 18;
    if(this.hide != hide){
      this.hide = hide;
      notifyListeners();
    }
  }
}
```

# main.dart
```
class MyApp extends StatelessWidget {

  @override
  Widget build(BuildContext context) {

    return MultiProvider(
      providers: [
        ....
        ChangeNotifierProvider.value(value: SecretImageNotifier())
      ],
      child: MaterialApp(
    );
  }
}
```

생각나는대로 만들어서 구현하면 어쨌든 생각한대로 구현되는건 좋은데, 너무 대충 만드는 감이 없잖아 있는것같다.

섬네일 계통을 모두 이걸로 한다하면 아예 섬네일 제공자를 여기에 통합하는것도 나쁘지않을것같고...


