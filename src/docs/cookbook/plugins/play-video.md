---
title: 영상 재생 및 일시 중지
prev:
  title: 디스크에 키-값 데이터 저장하기
  path: /docs/cookbook/persistence/key-value
next:
  title: 카메라 사진 찍기
  path: /docs/cookbook/plugins/picture-using-camera
---

앱 개발에 있어 영상 재생은 일반적인 작업으로 Flutter에서도 예외는 아닙니다. 영상을
재생하기 위해 Flutter는 
[`video_player`]({{site.pub-pkg}}/video_player) 플러그인을
제공하고 있습니다. `video_player` 플러그인을 통해 파일 시스템에 저장된 영상이나
인터넷의 영상을 재생할 수 있습니다.

iOS의 경우 `video_player` 플러그인은
[`AVPlayer`](https://developer.apple.com/documentation/avfoundation/avplayer)를
사용하여 영상을 재생합니다. Android의 경우는 
[`ExoPlayer`](https://google.github.io/ExoPlayer/)를 사용합니다.

본 예제에서는 `video_player` 패키지를 사용하여 인터넷 영상을 스트리밍하고
재생 및 일시 정지와 같은 기본적인 컨트롤을 해볼 것입니다.

## 진행 단계

  1. `video_player` 의존성 추가하기
  2. 앱에 권한 부여하기
  3. `VideoPlayerController` 생성 및 초기화하기
  4. video player 화면에 보여주기
  5. 영상을 재생 및 일시 중지하기

## 1. `video_player` 의존성 추가하기

본 예제는 Flutter 플러그인에 의존합니다: `video_player`. 우선 이 라이브러리를
`pubspec.yaml`에 추가합니다.

```yaml
dependencies:
  flutter:
    sdk: flutter
  video_player:
```

## 2. 앱에 권한 부여하기

다음으로, 앱이 인터넷 영상을 스트리밍하기 위해 올바른 권한을 가졌는지 확인해야 합니다.
이를 위해, `Android`와 `ios` configuration을 수정하겠습니다.

### Android

다음의 권한을 `AndroidManifest.xml` 파일의 `<application>` 바로 다음에 추가하세요.
`AndroidManifest.xml`은 `<projectroot>/android/app/src/main/AndroidManifest.xml`에서
찾을 수 있습니다.

<!-- skip -->
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application ...>
        
    </application>

    <uses-permission android:name="android.permission.INTERNET"/>
</manifest>
```

### iOS

iOS는 아래 내용을 `<project root>/ios/Runner/Info.plist`에 파일에 추가해야 합니다.

<!-- skip -->
```xml
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
</dict>
```

{{site.alert.warning}}
`video_player` 플러그인은 iOS 시뮬레이터에서는 동작하지 않으므로, 영상 재생 테스트는
반드시 실제 iOS 기기에서 해야 합니다.
{{site.alert.end}}

## 3. `VideoPlayerController` 생성 및 초기화하기

이제 `video_player`과 올바른 권한 모두 준비되었고, `VideoPlayerController`를 만들
차례입니다. `VideoPlayerController` 클래스를 통해 다른 여러 유형의 영상에 연결하고
제어할 수 있습니다.

영상을 재생하기 전에, controller를 `초기화`해야 합니다. 이 작업을 통해 영상과 연결하고
영상 제어를 위한 준비 작업을 하게 됩니다.

`VideoPlayerController`를 초기화하려면,

  1. `StatefulWidget`과 `State` 클래스를 생성하세요.
  2. `State` 클래스에 `VideoPlayerController`를 저장하기 위한 변수를 추가하세요.
  3. `State` 클래스에 `VideoPlayerController.initialize`로부터 반환되는 `Future`를
  저장하기 위한 변수를 추가하세요.
  4. `initState` 메서드에서 controller를 생성하고 초기화하세요.
  5. `dispose` 메서드에서 controller를 dispose 시키세요.
  
<!-- skip -->
```dart
class VideoPlayerScreen extends StatefulWidget {
  VideoPlayerScreen({Key key}) : super(key: key);

  @override
  _VideoPlayerScreenState createState() => _VideoPlayerScreenState();
}

class _VideoPlayerScreenState extends State<VideoPlayerScreen> {
  VideoPlayerController _controller;
  Future<void> _initializeVideoPlayerFuture;

  @override
  void initState() {
    // VideoPlayerController를 저장하기 위한 변수를 만듭니다. VideoPlayerController는
    // asset, 파일, 인터넷 등의 영상들을 제어하기 위해 다양한 생성자를 제공합니다.
    _controller = VideoPlayerController.network(
      'https://flutter.github.io/assets-for-api-docs/assets/videos/butterfly.mp4',
    );

    _initializeVideoPlayerFuture = _controller.initialize();

    super.initState();
  }

  @override
  void dispose() {
    // 자원을 반환하기 위해 VideoPlayerController를 dispose 시키세요.
    _controller.dispose();

    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // 다음 단계에서 영상을 보여줄 것입니다.
  }
}
```

## 4. video player 화면에 보여주기

이제 영상을 보여줄 차례입니다. `video_player` 플러그인은 `VideoPlayerController`에
의해 초기화된 영상을 보여주기 위해
[`VideoPlayer`]({{site.pub-api}}/video_player/latest/video_player/VideoPlayer-class.html)
위젯을 제공합니다. 기본적으로 `VideoPlayer` 위젯은 가능한한 넓은 면적을 차지하려고 합니다.
이러한 점은 종종 영상들이 16x9나 4x3과 같이 특정 종횡비로 표시되기 때문에 이상적이지는 않습니다.


영상이 제대로된 비율로 보여지도록 `VideoPlayer` 위젯을 
[`AspectRatio`]({{site.api}}/flutter/widgets/AspectRatio-class.html)
위젯으로 감쌀 수 있습니다.

추가로 `_initializeVideoPlayerFuture`가 완료된 후에 `VideoPlayer`를 보여줘야 합니다.
`FutureBuilder`를 사용하여 초기화가 진행되는 동안 로딩 스피너를 보여줄 수 있습니다. 참고:
컨트롤러 초기화가 영상을 재생시키지는 않습니다.

<!-- skip -->
```dart
// VideoPlayerController가 초기화를 진행하는 동안 로딩 스피너를 보여주기 위해
// FutureBuilder를 사용합니다.
FutureBuilder(
  future: _initializeVideoPlayerFuture,
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.done) {
      // 만약 VideoPlayerController 초기화가 끝나면, 제공된 데이터를 사용하여
      // VideoPlayer의 종횡비를 제한하세요.
      return AspectRatio(
        aspectRatio: _controller.value.aspectRatio,
        // 영상을 보여주기 위해 VideoPlayer 위젯을 사용합니다.
        child: VideoPlayer(_controller),
      );
    } else {
      // 만약 VideoPlayerController가 여전히 초기화 중이라면, 
      // 로딩 스피너를 보여줍니다.
      return Center(child: CircularProgressIndicator());
    }
  },
)
```

## 5. 영상을 재생 및 일시 중지하기

기본적으로 영상은 일시 중지 상태로 시작합니다. 재생을 시작하려면 `VideoPlayerController`가
제공하는 
[`play`]({{site.pub-api}}/video_player/latest/video_player/VideoPlayerController/play.html)
메서드를 호출하세요. 재생을 일시 중지 시키려면 
[`pause`]({{site.pub-api}}/video_player/latest/video_player/VideoPlayerController/pause.html)
메서드를 호출하면 됩니다.

본 예제에서는 상황에 따라 재생 혹은 일시 중지 아이콘을 보여주기 위해
`FloatingActionButton`을 추가하였습니다. 사용자가 이 버튼을 눌렀을 때, 영상이 일시 중지
상태였다면 재생할 것이고, 재생 중이었다면 일시 중지시킬 것입니다.

<!-- skip -->
```dart
FloatingActionButton(
  onPressed: () {
    // 재생/일시 중지 기능을 `setState` 호출로 감쌉니다. 이렇게 함으로써 올바른 아이콘이
    // 보여집니다.
    setState(() {
      // 영상이 재생 중이라면, 일시 중지 시킵니다.
      if (_controller.value.isPlaying) {
        _controller.pause();
      } else {
        // 만약 영상이 일시 중지 상태였다면, 재생합니다.
        _controller.play();
      }
    });
  },
  // 플레이어의 상태에 따라 올바른 아이콘을 보여줍니다.
  child: Icon(
    _controller.value.isPlaying ? Icons.pause : Icons.play_arrow,
  ),
)
``` 
 
## 완성된 예제

```dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:video_player/video_player.dart';

void main() => runApp(VideoPlayerApp());

class VideoPlayerApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Video Player Demo',
      home: VideoPlayerScreen(),
    );
  }
}

class VideoPlayerScreen extends StatefulWidget {
  VideoPlayerScreen({Key key}) : super(key: key);

  @override
  _VideoPlayerScreenState createState() => _VideoPlayerScreenState();
}

class _VideoPlayerScreenState extends State<VideoPlayerScreen> {
  VideoPlayerController _controller;
  Future<void> _initializeVideoPlayerFuture;

  @override
  void initState() {
    // VideoPlayerController를 저장하기 위한 변수를 만듭니다. VideoPlayerController는
    // asset, 파일, 인터넷 등의 영상들을 제어하기 위해 다양한 생성자를 제공합니다.
    _controller = VideoPlayerController.network(
      'https://flutter.github.io/assets-for-api-docs/assets/videos/butterfly.mp4',
    );

    // 컨트롤러를 초기화하고 추후 사용하기 위해 Future를 변수에 할당합니다.
    _initializeVideoPlayerFuture = _controller.initialize();

    // 비디오를 반복 재생하기 위해 컨트롤러를 사용합니다.
    _controller.setLooping(true);

    super.initState();
  }

  @override
  void dispose() {
    // 자원을 반환하기 위해 VideoPlayerController를 dispose 시키세요.
    _controller.dispose();

    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Butterfly Video'),
      ),
      // VideoPlayerController가 초기화를 진행하는 동안 로딩 스피너를 보여주기 위해
      // FutureBuilder를 사용합니다.
      body: FutureBuilder(
        future: _initializeVideoPlayerFuture,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.done) {
            // 만약 VideoPlayerController 초기화가 끝나면, 제공된 데이터를 사용하여
            // VideoPlayer의 종횡비를 제한하세요.
            return AspectRatio(
              aspectRatio: _controller.value.aspectRatio,
              // 영상을 보여주기 위해 VideoPlayer 위젯을 사용합니다.
              child: VideoPlayer(_controller),
            );
          } else {
            // 만약 VideoPlayerController가 여전히 초기화 중이라면, 
            // 로딩 스피너를 보여줍니다.
            return Center(child: CircularProgressIndicator());
          }
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // 재생/일시 중지 기능을 `setState` 호출로 감쌉니다. 이렇게 함으로써 올바른 아이콘이
          // 보여집니다.
          setState(() {
            // 영상이 재생 중이라면, 일시 중지 시킵니다.
            if (_controller.value.isPlaying) {
              _controller.pause();
            } else {
              // 만약 영상이 일시 중지 상태였다면, 재생합니다.
              _controller.play();
            }
          });
        },
        // 플레이어의 상태에 따라 올바른 아이콘을 보여줍니다.
        child: Icon(
          _controller.value.isPlaying ? Icons.pause : Icons.play_arrow,
        ),
      ), // 이 마지막 콤마는 build 메서드에 자동 서식이 잘 적용될 수 있도록 도와줍니다.
    );
  }
}
```
