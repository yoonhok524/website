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
[`video_player`](https://pub.dartlang.org/packages/video_player) 플러그인을
제공하고 있습니다. `video_player` 플러그인을 통해 파일 시스템에 저장된 영상이나
인터넷의 영상을 재생할 수 있습니다.

iOS의 경우 `video_player` 플러그인은
[`AVPlayer`](https://developer.apple.com/documentation/avfoundation/avplayer)를
사용하여 영상을 재생합니다. 안드로이드의 경우는 
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
이를 위해, `안드로이드`와 `ios` configuration을 수정하겠습니다.

### 안드로이드

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

Now, it's time to display the video. The `video_player` plugin provides the
[`VideoPlayer`](https://pub.dartlang.org/documentation/video_player/latest/video_player/VideoPlayer-class.html)
Widget to display the video initialized by the `VideoPlayerController`. By
default, the `VideoPlayer` Widget takes up as much space as possible. This
often isn't ideal for videos because they are meant to be displayed in a
specific aspect ratio, such as 16x9 or 4x3.

Therefore, you can wrap the `VideoPlayer` widget in an
[`AspectRatio`](https://docs.flutter.io/flutter/widgets/AspectRatio-class.html)
widget to ensure the video is the correct proportions.

Furthermore, you must display the `VideoPlayer` widget after the
`_initializeVideoPlayerFuture` completes. You can use a `FutureBuilder` to
display a loading spinner until finishes initializing. Note: initializing the
controller does not begin playback.

<!-- skip -->
```dart
// Use a FutureBuilder to display a loading spinner while you wait for the
// VideoPlayerController to finish initializing.
FutureBuilder(
  future: _initializeVideoPlayerFuture,
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.done) {
      // If the VideoPlayerController has finished initialization, use
      // the data it provides to limit the Aspect Ratio of the VideoPlayer
      return AspectRatio(
        aspectRatio: _controller.value.aspectRatio,
        // Use the VideoPlayer widget to display the video
        child: VideoPlayer(_controller),
      );
    } else {
      // If the VideoPlayerController is still initializing, show a
      // loading spinner
      return Center(child: CircularProgressIndicator());
    }
  },
)
```

## 5. 영상을 재생 및 일시 중지하기

By default, the video starts in a paused state. To begin playback,
call the
[`play`](https://pub.dartlang.org/documentation/video_player/latest/video_player/VideoPlayerController/play.html)
method provided by the `VideoPlayerController`. To pause playback, call the
[`pause`](https://pub.dartlang.org/documentation/video_player/latest/video_player/VideoPlayerController/pause.html)
method.

For this example, add a `FloatingActionButton` to your app that displays a play
or pause icon depending on the situation. When the user taps the button, play
the video if it's currently paused, or pause the video if it's playing.

<!-- skip -->
```dart
FloatingActionButton(
  onPressed: () {
    // Wrap the play or pause in a call to `setState`. This ensures the correct 
    // icon is shown
    setState(() {
      // If the video is playing, pause it.
      if (_controller.value.isPlaying) {
        _controller.pause();
      } else {
        // If the video is paused, play it
        _controller.play();
      }
    });
  },
  // Display the correct icon depending on the state of the player.
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

    // Initialize the controller and store the Future for later use
    _initializeVideoPlayerFuture = _controller.initialize();

    // Use the controller to loop the video
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
      // Use a FutureBuilder to display a loading spinner while you wait for the
      // VideoPlayerController to finish initializing.
      body: FutureBuilder(
        future: _initializeVideoPlayerFuture,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.done) {
            // If the VideoPlayerController has finished initialization, use
            // the data it provides to limit the Aspect Ratio of the Video
            return AspectRatio(
              aspectRatio: _controller.value.aspectRatio,
              // Use the VideoPlayer widget to display the video
              child: VideoPlayer(_controller),
            );
          } else {
            // If the VideoPlayerController is still initializing, show a
            // loading spinner
            return Center(child: CircularProgressIndicator());
          }
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // Wrap the play or pause in a call to `setState`. This ensures the
          // correct icon is shown
          setState(() {
            // If the video is playing, pause it.
            if (_controller.value.isPlaying) {
              _controller.pause();
            } else {
              // If the video is paused, play it
              _controller.play();
            }
          });
        },
        // Display the correct icon depending on the state of the player.
        child: Icon(
          _controller.value.isPlaying ? Icons.pause : Icons.play_arrow,
        ),
      ), // This trailing comma makes auto-formatting nicer for build methods.
    );
  }
}
```
