---
layout: post
published: false
title: Flutter Hero와 FutureBuilder
---
페이지간 이동할때, Hero 위젯을 쓰면 해당 위젯끼리 전환되는 꽤나 멋들어진 효과가 난다.

내 경우는 이미지 목록 페이지 -> 이미지 정보 페이지로 갈때 그 효과가 꽤 큰데, 문제는 이게 언젠가부터 전환될때 ( 목록 -> 정보건 정보 -> 목록이건간에) 이미지가 자꾸 깜빡인다는것.

key 문젠가 하고 key도 넣어보고 별 쌩쇼를 다 해봤지만 효과는 없고, 그러다 문득 생각난것.

일단 코드부터.

### secret_image.dart
```
  Widget _buildS3Image(BuildContext context, SecretImageNotifier notify) {
    return FutureBuilder(
      future: notify.accessS3(param),
      builder: (ctx, snapshot) {
        if (!snapshot.hasData)
          return SpinKitFadingCube(
            size: 20,
            duration: Duration(milliseconds: 500),
            color: DefinedColor.primary,
          );

        if (snapshot.data == 404)
          return Center(
              child: Text(
                "NOT EXIST : $param",
              ));

        return Image.file(
          File(notify.thumbnailPath(param)),
          fit: fit,
        );
      },
    );
  }
```

보통 Flutter에서 외부 웹 이미지를 쓸때는 Image.network를 쓴다지만 내 경우는 헤더 만드는것부터가 좀 많이 귀찮은 작업이라 외부 Notifier에게 일임한 상태였다.

다운받고나서 notify가 오면 결과값 (status code)로 이미지가 있는지를 판단해서 뿌릴지 말지를 결정하는것.

### secret_image_notifier.dart
```
  Future<int> accessS3(int id) async {
    var provider = ThumbnailProvider.of(_context);
    // 현재 프로세스에 요청 이미지 id에 대한 결과값이 없는경우 true를 반환한다.
    if (provider.add(id)) {
      return provider.stream
          .firstWhere((element) => element['id'] == id)
          .then((value) => value['result']); // 요청 이미지 id에 대한 결과가 들어올때까지 대기한다. 
    }
    return provider.resMap[id]; //요청시 이미 프로세스에 결과값이 있는경우 즉각 반환한다.
  }
```
이 결과값을 캐싱해놓지않으면 이미 파일이 디스크에 저장되어잇어도 그걸 다시 찾는데까지 시간이 걸릴거라 생각해서 결과값까진 메모리에 캐싱해둿는데, 여기서 더 생각해야했다.

코드 상으로는 그냥 build 시점에 바로 결과값이 리턴될거라 생각했는데, 어쨌든 async 문이라 그게 생각한대로 안된다는 것.

그럼 우찌할까.

```
  int accessS3Instant(int id) {
    var provider = ThumbnailProvider.of(_context);
    return provider.resMap[id] ?? -1;
  }

  Future<int> accessS3(int id) async {
    var provider = ThumbnailProvider.of(_context);
    if (provider.add(id)) {
      return provider.stream
          .firstWhere((element) => element['id'] == id)
          .then((value) => value['result']);
    }
    return provider.resMap[id];
  }
```
async 문이 아닌애를 추가하고, 얘는 없는경우 -1을 반환하게 한다.

```
  @override
  Widget build(BuildContext context) {
    buffer ??= Consumer<SecretImageNotifier>(
      builder: (ctx, notify, __) {
        return _buildS3Image(notify, notify.accessS3Instant(param)); // sync 함수로 현재 결과값을 체크한다.
      },
    );
    return buffer;
  }
  
  Widget _buildS3Image(SecretImageNotifier notify, resCode) {
    if (resCode == -1) return _buildS3ImageFuture(notify); // 결과값이 존재하지않는경우 FutureBuilder를 사용해 대기한다.
    // 결과값이 존재하는경우 결과값에 따라 분기 처리
    if (resCode == 404)
      return Center(
          child: Text(
            "NOT EXIST : $param",
          ));

    return Image.file(
      File(notify.thumbnailPath(param)),
      fit: fit,
    );
  }
  
  Widget _buildS3ImageFuture(SecretImageNotifier notify) {
    return FutureBuilder(
      future: notify.accessS3(param),
      builder: (ctx, snapshot) {
        if (!snapshot.hasData)
          return SpinKitFadingCube(
            size: 20,
            duration: Duration(milliseconds: 500),
            color: DefinedColor.primary,
          );

        var resCode = snapshot.data;
        return _buildS3Image(notify, resCode); // 결과값이 들어온경우 구현부를 다시 호출한다.
      },
    );
  }
```
build시 futurebuilder부터 만들고보는게 아니라, 동기 함수로 결과값을 바로 체크할수 없는경우에만 FutureBuilder를 사용하게한다.


Builder 계통은 역시 최대한 덜 쓰는게 좋을지도.









