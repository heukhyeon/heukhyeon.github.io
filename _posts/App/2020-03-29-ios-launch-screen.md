---
layout: post
published: false
title: IOS Launch Screen 업데이트 안됨
---
Flutter로 앱을 구성해도 프로세스가 부팅되는 몇초동안은 [Flutter의 위젯이 아닌 네이티브 (안드로이드 ,IOS)의 화면이 출력된다.](https://flutter.dev/docs/development/ui/splash-screen/android-splash-screen)

안드로이드는 개별 SplashScreen 클래스로 만들어도 잠깐의 공백이 있어 결국 windowBackground를 커스텀하는 수밖에 없고, IOS는 LaucnScreen.storyboard를 커스텀한다.

근데 이게....가끔 반영이 안된다. 플러터의 문제가 아니라 [IOS 자체의 이슈.](https://stackoverflow.com/questions/33002829/ios-keeping-old-launch-screen-and-app-icon-after-update)

보통 공통적인 방법은 앱을 삭제 하고 다시 깔면 된다는거지만, 퍽이나 삭제하고 다시 깔겠다...

내 경우, 기존 로딩되는 이미지가 LauchImage 였는데, 이걸 SplashIcon으로 이름을 바꾸는 방식으로 해결.


글만 봐선 App Store에서 받을땐 발생안한다 카니 그닥 중요한 이슈는 아닌것같다만 묘하게 불편하다.




