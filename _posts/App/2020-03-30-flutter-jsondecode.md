---
layout: post
published: false
title: Flutter jsonDecode
---
안드로이드에서 흔히 쓰던 fromJson, toJson이 Flutter에는 jsonDecode, jsonEncode로 내장되어있다.

jsonEncode는 뭔 짓을 하던 그럭저럭 잘 돌아가는데, jsonDecode가 말썽.

보통 이런경우다.

## GalleryPage.dart
```
class GalleryPage {
  final int id;
  final String name;
  final int latestGalleryId;
  final String latestGalleryName;
  bool selected;
  final Map<String, List<String>> params;

  GalleryPage({this.id,this.name, this.latestGalleryId, this.latestGalleryName, this.selected, this.params});

  factory GalleryPage.fromObject(Map<String,dynamic> obj){
    var params2 = jsonDecode(obj['params']);
    return GalleryPage(
      id: obj['id'],
      name: obj['name'],
      latestGalleryId: obj['latestGalleryId'],
      latestGalleryName: obj['latestGalleryName'],
      selected: obj['selected'] == 1,
      params: params2
    );
  }

  Map<String,dynamic> toObject(){
    return {
      'id' : id,
      'name' : name,
      'latestGalleryId': latestGalleryId,
      'latestGalleryName': latestGalleryName,
      'selected' : selected,
      'params' : jsonEncode(params)
    };
  }

}
```

다른건 다 상관없고 params를 보자. toObject 할때 string으로 만들었으니 fromObject 때는 역으로 decode 했다.

얼핏 봐선 멀쩡히 돌아갈것같은데 100% 오류가 난다.

```
[ERROR:flutter/lib/ui/ui_dart_state.cc(157)] Unhandled Exception: type '_InternalLinkedHashMap<String, dynamic>' is not a subtype of type 'Map<String, List<String>>'
```

디버그를 찍어보면 확실히 jsonDecode 했을때의 타입은 ~Map<String,dynamic>이 맞지만, 웃긴건 그래서 내부 타입을 뜯어봤을때의 타입이 다르냐 하면, 그것도 모두 List다.

인터넷에 [관련 이슈] (https://medium.com/codespace69/flutter-json-decode-type-internallinkedhashmap-dynamic-dynamic-is-not-a-subtype-of-type-9d6b3e982b59) 가 있어서 Map.from으로 감싸봤다.

```
  factory GalleryPage.fromObject(Map<String,dynamic> obj){
    var params2 = Map<String, List<String>>.from(jsonDecode(obj['params']));
    return GalleryPage(
      id: obj['id'],
      name: obj['name'],
      latestGalleryId: obj['latestGalleryId'],
      latestGalleryName: obj['latestGalleryName'],
      selected: obj['selected'] == 1,
      params: params2
    );
  }
```

이러면 될까? 다른 오류가 나온다.

```
 [ERROR:flutter/lib/ui/ui_dart_state.cc(157)] Unhandled Exception: type 'List<dynamic>' is not a subtype of type 'List<String>'
```

List까진 맞는데 이게 String이라 확언을 못하겠다는거.

이놈의 제네릭은 여길가나 저길가나 골치구만..

보통 이렇게까지 중첩되면 클래스를 개별로 분할해서 fromJson을 따로 부르게끔 하지만, Map Key 자체가 가변인 놈이라 그 방법도 애매하다.

결국 찾은 해답은...

```
  factory GalleryPage.fromObject(Map<String, dynamic> obj){
    return GalleryPage(
        id: obj['id'],
        name: obj['name'],
        latestGalleryId: obj['latestGalleryId'],
        latestGalleryName: obj['latestGalleryName'],
        selected: obj['selected'] == 1,
        params: Map<String, List<dynamic>>.from(jsonDecode(obj['params'])).map((
            key, value) => MapEntry(key, List<String>.from(value)))
    );
  }
```

한번 더 transform을 명시적으로 해주는식으로 땜빵.




