---
title: 백그라운드 프로세스
description: Flutter에서 백그라운드 프로세스를 구현하는데 필요한 추가 정보
---

백그라운드에서 Dart 코드를 실행하고 싶었던 적이 있습니까? 앱이 활성화되어 있지 않은 상태일지라도 말이죠. 어쩌면 여려분은 시간을 주시하거나 또는 카메라의 움직임을 캐치하는 프로세스를 만들고 싶었을 수도 있습니다. Flutter에서는 백그라운드에서 Dart 코드를 실행할 수 있습니다.

이 기능의 메커니즘은 isolate를 설정하는 것입니다. _Isolates_ 는 Dart의 멀티스레딩을 위한 모델인데, 메인 프로그램과 메모리를 공유하지 않는다는 점에서 통속적인 스레드의 isolate과는 다릅니다. 콜백과 콜백 디스패처를 이용해서 백그라운드 실행을 위한 isolate를 설정합니다. 

더 자세한 정보 및 Dart의 백그라운드 프로세스가 적용된 Geofencing 예제는 Medium의 Flutter Publication에 있는 [Dart에서 Flutter 플러그인과 Geofencing을 이용한 백그라운드 실행]({{site.flutter-medium}}/executing-dart-in-the-background-with-flutter-plugins-and-geofencing-2b3e40a1a124) 을 참고하시기 바랍니다. 글의 끝부분에는 예제 코드에 대한 링크와 Dart, iOS, Android를 위한 관련 문서가 나와있습니다. 