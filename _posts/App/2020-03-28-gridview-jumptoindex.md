---
layout: post
published: false
title: GridView JumpToIndex
categories:
  - App
tags:
  - Flutter
---
2020-03-28 기준으로 플러터에서 기본 제공되는 ListView는 [jumpToIndex 기능이 없다.](https://stackoverflow.com/questions/54039684/flutter-listview-scroll-to-index-not-available)
링크에선 다른 커스텀 플러그인을 이야기하지만, 내 경우는 애초에 Vertcial (Horizontal) ListView가 아니라 GridView.


요컨대, 이런 케이스다.

```
  @override
  Widget blocBuilder(BuildContext context, BaseState state) {
    return WillPopScope(
      // 한장씩 보는 상태에서 뒤로갈경우 목록으로 돌아간다.
      onWillPop: () async {
        if(bloc.detailMode){
          bloc.add(DownloadGalleryDetailToggleEvent(index: bloc.selectedIndex, toggle: false));
          return false;
        }
        return true;
      },
      child:  Scaffold(
        // 한장씩 보는 상태인 경우 필요에 따라 앱바를 숨기거나 펼친다.
        appBar: (bloc.detailMode && bloc.detailAppBarHide) ? null : AppBar(
          centerTitle: true,
          title: Text(widget.record.visibleName),
        ),
        resizeToAvoidBottomPadding: !bloc.detailMode,
        
        body: bloc.files.isEmpty
            ? ProgressWidget()
            // 한장씩 보기 모드인경우 PageView를, 목록 보기 모드인경우 GridView를 사용한다.
            : bloc.detailMode ? buildDetailMode() : buildListMode(),
      ),
    );
  }
```
사용자는 자유롭게 목록 모드와 한장 모드를 교체할수있지만, 교체할때 이전 모드에서 선택한 index가 유지되어야 한다.

목록 -> 한장 모드는 상관없다. PageController(initalPage: currentPage)로 컨트롤러 갱신하고 rebuild 시켜버리면 되니까.

문제는 한장 -> 목록인 경우, jumpToIndex가 없으니 딱딱 떨어지는 계산을 할수가 없다.

    
    
    
***

## 뻘짓의 과정

그나마 GridView같은 경우는 각 cell의 크기가 고정이니 계산할수 있어보인다.

```
  Widget buildListMode() {
    return GridView.builder(
      controller: _listScroll,
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 4,
          crossAxisSpacing: 5,
          mainAxisSpacing: 5,
          childAspectRatio: 1 / 1.15),
      itemBuilder: // blabla...,
      itemCount: bloc.files.length,
    );
  }
```

GridView의 width는 실제 앱의 가로 표시 영역과 동일하므로, 대강 이런 류의 계산.

_(((화면 길이 / 4) * 1.15) + 5) * (인덱스 / 4)_

코드 상으로는 이렇게 구현된다.

```
  @override
  void blocListener(BuildContext context, BaseState state) {
    super.blocListener(context, state);
    if(state is DownloadGalleryDetailToggleState){
      final width = MediaQuery.of(context).size.width;
      final oneHeight = ((width / 4) * 1.15) + 5;
      final line = bloc.selectedIndex / 4;
      _listScroll = ScrollController(initialScrollOffset: oneHeight * line);
    }
  }
```

실행해보니 한 라인 더 들어가는것 같다. line에 1을 빼보는걸로.

```
final line = max((bloc.selectedIndex / 4) - 1, 0);
```

이러면 얼추 맞기는한데, 아래로 내려가면 내려갈수록 뭔가 의도한 줄로 내려오지않는다. oneHeight을 구할때 5를 제거해보기로.

```
final oneHeight = ((width / 4) * 1.15);
final line = max((bloc.selectedIndex / 4), 0);
```

역시 뭔가 내려가면 내려갈수록 offset이 맞지 않는다.


mainAxisSpacing이 있으니 이거까지 감안해보기로.


```
final size = MediaQuery.of(context).size;
final columnCount = 4.0;
final mainAxisSpacing = 5.0;
final aspectRatio = 1.15;
final width = size.width - (mainAxisSpacing * (columnCount - 1));
final oneHeight = ((width / columnCount) * aspectRatio);
final line = max((bloc.selectedIndex / columnCount), 0.0);
var offset = oneHeight * line;
_listScroll = ScrollController(initialScrollOffset: offset);
```

뭔가 엄청 안내려간다.


crossAxisSpacing이 적용 안된것 같아서 넣어봤다.

```
final size = MediaQuery.of(context).size;
final columnCount = 4.0;
final mainAxisSpacing = 5.0;
final crossAxisSpacing = 5.0;
final aspectRatio = 1.15;
final width = size.width - (mainAxisSpacing * (columnCount - 1));
final oneHeight = ((width / columnCount) * aspectRatio) + crossAxisSpacing;
final line = max((bloc.selectedIndex / columnCount), 0.0);
var offset = oneHeight * line;
_listScroll = ScrollController(initialScrollOffset: offset);
```

완벽하게 작동.

***


Gridview같은 경우는 여기저기 재활용할것같으니 이거 자체를 별도 클래스로 분리하기로.


## GridController.dart
```
import 'package:flutter/widgets.dart';

// https://heukhyeon.github.io/
class GridController {
  final int column;
  final double mainAxisSpacing;
  final double crossAxisSpacing;
  final double aspectRatio;
  SliverGridDelegate get delegate => SliverGridDelegateWithFixedCrossAxisCount(
      crossAxisCount: 4,
      crossAxisSpacing: 5,
      mainAxisSpacing: 5,
      childAspectRatio: 1 / 1.15);

  ScrollController _scroll = ScrollController();
  ScrollController get scroll => _scroll;

  GridController(
      {@required this.column,
      @required this.mainAxisSpacing,
      @required this.crossAxisSpacing,
      @required this.aspectRatio}):
      assert(column != null),
      assert(mainAxisSpacing != null),
      assert(crossAxisSpacing != null),
      assert(aspectRatio != null);

  void jumpToIndex({int index, double gridViewWidth}){
    final width = gridViewWidth - (this.mainAxisSpacing * (column - 1));
    final oneHeight = ((width / column) * this.aspectRatio) + this.crossAxisSpacing;
    final line = (index / column);
    var offset = oneHeight * line;
    if(_scroll.hasClients){
      _scroll.jumpTo(offset);
    }
    else {
      _scroll = ScrollController(initialScrollOffset: offset);
    }
  }

  void dispose() {
    _scroll = null;
  }

}
```

사용 방법은 이런식.


```
  final gridController = GridController(column: 4, mainAxisSpacing: 5.0, crossAxisSpacing: 5.0, aspectRatio: 1 / 1.5);

  Widget buildListMode() {
    return GridView.builder(
      controller: gridController.scroll,
      gridDelegate: gridController.delegate,
      itemBuilder: (_, i) => Container(),
      itemCount: bloc.files.length,
    );
  }
  
  ....
  
    @override
  void blocListener(BuildContext context, BaseState state) {
    super.blocListener(context, state);
    if(state is DownloadGalleryDetailToggleState){
      gridController.jumpToIndex(index: bloc.selectedIndex, gridViewWidth: MediaQuery.of(context).size.width);
      SystemChrome.setEnabledSystemUIOverlays (state.toggle ? [] : [
        SystemUiOverlay.top,
        SystemUiOverlay.bottom
      ]);
    }
  }
```
