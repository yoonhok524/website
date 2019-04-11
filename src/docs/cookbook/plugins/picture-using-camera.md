---
title: 카메라 사진 찍기
prev:
  title: 영상 재생 및 일시 중지
  path: /docs/cookbook/plugins/play-video
next:
  title: 통합 테스팅 소개
  path: /docs/cookbook/testing/integration/introduction
---

사진을 찍거나 영상을 촬영하기 위해 디바이스의 카메라를 사용하는 많은 앱들이 있습니다.
이러한 목적을 위해 Flutter는 [`camera`](https://pub.dartlang.org/packages/camera) 
플러그인을 제공합니다. `camera` 플러그인을 통해 이용가능한 카메라 리스트를 얻고, 특정 
카메라라의 프리뷰를 보여주거나 사진, 영상을 촬영하는 등의 작업을 할 수 있습니다.

본 예제에서는 `camera` 플러그인을 사용하여 프리뷰를 보여주고, 사진을 촬영한 뒤 결과물을 
사용자에게 보여주는 방법을 소개합니다.

## 진행 단계

  1. 필요한 의존성 추가하기
  2. 이용가능한 카메라 목록 가져오기
  3. `CameraController` 생성하고 초기화하기
  4. `CameraPreview`를 사용하여 카메라 피드 보여주기
  5. `CameraController`를 사용하여 사진 찍기
  6. 촬영한 사진을 `Image` 위젯으로 보여주기

## 1. 필요한 의존성 추가하기

본 예제를 수행하기 위해, 아래 세 개의 의존성을 추가해야 합니다:

  - [`camera`](https://pub.dartlang.org/packages/camera) - 디바이스의 카메라를 제어하기 위한 도구를 제공합니다.
  - [`path_provider`](https://pub.dartlang.org/packages/path_provider) - 이미지를 저장할 적합한 경로를 찾습니다.
  - [`path`](https://pub.dartlang.org/packages/path) - 플랫폼에 상관없이 경로를 생성합니다.

```yaml
dependencies:
  flutter:
    sdk: flutter
  camera:
  path_provider:
  path:
```

## 2. 이용가능한 카메라 목록 가져오기

이제 `camera` 플러그인을 사용하여 이용가능한 카메라 목록을 받아올 수 있습니다.

<!-- skip -->
```dart
// 디바이스에서 이용가능한 카메라 목록을 받아옵니다.
final cameras = await availableCameras();

// 이용가능한 카메라 목록에서 특정 카메라를 얻습니다.
final firstCamera = cameras.first; 
```

## 3. `CameraController` 생성하고 초기화하기

카메라가 준비되었으면, `CameraController`를 생성하고 초기화할 차례입니다. 이 과정을 통해
디바이스의 카메라와 연결하여 카메라를 제어하고 카메라 피드의 프리뷰를 보여줄 수 있게 됩니다. 

이를 위해 다음의 과정을 수행합니다:

  1. `StatefulWidget`과 그에 상응하는 `State` 클래스를 생성합니다.
  2. `CameraController`를 저장할 변수를 `State` 클래스에 정의합니다.
  3. `CameraController.initialize`가 반환하는 `Future`를 저장할 변수를 `State`에
  정의합니다.
  4. `initState` 메서드에서 컨트롤러를 생성하고 초기화합니다.
  5. `dispose` 메서드에서 컨트롤러를 해제합니다.
  
<!-- skip -->
```dart
// A screen that takes in a list of Cameras and the Directory to store images.
class TakePictureScreen extends StatefulWidget {
  final CameraDescription camera;

  const TakePictureScreen({
    Key key,
    @required this.camera,
  }) : super(key: key);

  @override
  TakePictureScreenState createState() => TakePictureScreenState();
}

class TakePictureScreenState extends State<TakePictureScreen> {
  // CameraController와 Future를 저장하기 위해 두 개의 변수를 state 클래스에 
  // 정의합니다.
  CameraController _controller;
  Future<void> _initializeControllerFuture;

  @override
  void initState() {
    super.initState();
    // 카메라의 현재 출력물을 보여주기 위해 CameraController를 생성합니다.
    _controller = CameraController(
      // 이용 가능한 카메라 목록에서 특정 카메라를 가져옵니다.
      widget.camera,
      // 적용할 해상도를 지정합니다.
      ResolutionPreset.medium,
    );

    // 다음으로 controller를 초기화합니다. 초기화 메서드는 Future를 반환합니다.
    _initializeControllerFuture = _controller.initialize();
  }

  @override
  void dispose() {
    // 위젯의 생명주기 종료시 컨트롤러 역시 해제시켜줍니다.
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // 다음 단계에서 이 부분을 작성하겠습니다.
  }
}
```

{{site.alert.warning}}
만약 `CameraController`를 초기화하지 않으면, 프리뷰를 보여주거나 사진 촬영을 할 수 없습니다.
{{site.alert.end}}

## 4. `CameraPreview`를 사용하여 카메라 피드 보여주기

다음으로 `camera` 패키지의 `CameraPreview` 위젯을 사용하여 카메라 피드의 프리뷰를
보여줄 것입니다.

참고: 카메라를 사용하기 전에 컨트롤러의 초기화가 완료될 때까지 기다려야 합니다. 그러므로
`CameraPreview`를 보여주기 전에 이전 단계에서 만든 `_initializeControllerFuture`가
완료될 때까지 기다려야 합니다.

이를 위해 
[`FutureBuilder`](https://docs.flutter.io/flutter/widgets/FutureBuilder-class.html)를
사용하겠습니다.

<!-- skip -->
```dart
// 카메라 프리뷰를 보여주기 전에 컨트롤러 초기화를 기다려야 합니다. 컨트롤러 초기화가 
// 완료될 때까지 FutureBuilder를 사용하여 로딩 스피너를 보여주세요.
FutureBuilder<void>(
  future: _initializeControllerFuture,
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.done) {
      // Future가 완료되면, 프리뷰를 보여줍니다.
      return CameraPreview(_controller);
    } else {
      // 그렇지 않다면, 진행 표시기를 보여줍니다.
      return Center(child: CircularProgressIndicator());
    }
  },
)
```

## 5. `CameraController`를 사용하여 사진 찍기

`CameraController`의 
[`takePicture`](https://pub.dartlang.org/documentation/camera/latest/camera/CameraController/takePicture.html)
메서드를 사용하면 사진 촬영을 할 수 있습니다. 본 예제에서는 사용자가 눌렀을 때
`CameraController`를 사용하여 사진 촬영을 하는 `FloatingActionButton`을 
만들겠습니다.

촬영한 사진을 저장하기 위해 3 단계가 필요합니다:

  1. 카메라가 초기화되었는지 확인합니다.
  2. 사진이 저장될 경로를 지정합니다.
  3. 컨트롤러를 사용하여 사진을 촬영하고 그 결과물을 위에서 지정한 경로에 저장합니다.
  
발생할 수 있는 에러에 대처하기 위해 위 수행 단게들을 `try / catch`로 감싸는 것이 좋습니다.

<!-- skip -->
```dart
FloatingActionButton(
  child: Icon(Icons.camera_alt),
  // onPressed 콜백을 제공합니다.
  onPressed: () async {
    // try / catch 블럭에서 사진을 촬영합니다. 만약 뭔가 잘못된다면 에러에
    // 대응할 수 있습니다.
    try {
      // 카메라 초기화가 완료됐는지 확인합니다.
      await _initializeControllerFuture;

      // path 패키지를 사용하여 이미지가 저장될 경로를 지정합니다.
      final path = join(
        // 본 예제에서는 임시 디렉토리에 이미지를 저장합니다. `path_provider`
        // 플러그인을 사용하여 임시 디렉토리를 찾으세요.
        (await getTemporaryDirectory()).path,
        '${DateTime.now()}.png',
      );

      // 사진 촬영을 시도하고 저장되는 경로를 로그로 남깁니다.
      await _controller.takePicture(path);
    } catch (e) {
      // 만약 에러가 발생하면, 콘솔에 에러 로그를 남깁니다.
      print(e);
    }
  },
)
```
## 6. 촬영한 사진을 `Image` 위젯으로 보여주기

성공적으로 사진을 촬영했다면, `Image` 위젯을 사용하여 저장된 사진을 보여줄 수 있습니다.
이 경우 사진은 디바이스에 파일로 저장되어 있습니다.

그러므로 `Image.file` 생성자에 `File`을 제공해야 합니다. 이전 단계에서 생성한 경로를 
인자로 넘겨 주어 `File` 클래스의 인스턴스를 생성할 수 있습니다.

<!-- skip -->
```dart
Image.file(File('path/to/my/picture.png'))
```

## 완성된 예제

```dart
import 'dart:async';
import 'dart:io';

import 'package:camera/camera.dart';
import 'package:flutter/material.dart';
import 'package:path/path.dart' show join;
import 'package:path_provider/path_provider.dart';

Future<void> main() async {
  // 디바이스에서 이용가능한 카메라 목록을 받아옵니다.
  final cameras = await availableCameras();

  // 이용가능한 카메라 목록에서 특정 카메라를 얻습니다.
  final firstCamera = cameras.first;

  runApp(
    MaterialApp(
      theme: ThemeData.dark(),
      home: TakePictureScreen(
        // 적절한 카메라를 TakePictureScreen 위젯에게 전달합니다.
        camera: firstCamera,
      ),
    ),
  );
}

// 사용자가 주어진 카메라를 사용하여 사진을 찍을 수 있는 화면
class TakePictureScreen extends StatefulWidget {
  final CameraDescription camera;

  const TakePictureScreen({
    Key key,
    @required this.camera,
  }) : super(key: key);

  @override
  TakePictureScreenState createState() => TakePictureScreenState();
}

class TakePictureScreenState extends State<TakePictureScreen> {
  CameraController _controller;
  Future<void> _initializeControllerFuture;

  @override
  void initState() {
    super.initState();
    // 카메라의 현재 출력물을 보여주기 위해 CameraController를 생성합니다.
    _controller = CameraController(
      // 이용 가능한 카메라 목록에서 특정 카메라를 가져옵니다.
      widget.camera,
      // 적용할 해상도를 지정합니다.
      ResolutionPreset.medium,
    );

    // 다음으로 controller를 초기화합니다. 초기화 메서드는 Future를 반환합니다.
    _initializeControllerFuture = _controller.initialize();
  }

  @override
  void dispose() {
    // 위젯의 생명주기 종료시 컨트롤러 역시 해제시켜줍니다.
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Take a picture')),
      // 카메라 프리뷰를 보여주기 전에 컨트롤러 초기화를 기다려야 합니다. 컨트롤러 초기화가 
      // 완료될 때까지 FutureBuilder를 사용하여 로딩 스피너를 보여주세요.
      body: FutureBuilder<void>(
        future: _initializeControllerFuture,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.done) {
            // Future가 완료되면, 프리뷰를 보여줍니다.
            return CameraPreview(_controller);
          } else {
            // 그렇지 않다면, 진행 표시기를 보여줍니다.
            return Center(child: CircularProgressIndicator());
          }
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.camera_alt),
        // onPressed 콜백을 제공합니다.
        onPressed: () async {
          // try / catch 블럭에서 사진을 촬영합니다. 만약 뭔가 잘못된다면 에러에
          // 대응할 수 있습니다.
          try {
            // 카메라 초기화가 완료됐는지 확인합니다.
            await _initializeControllerFuture;

            // path 패키지를 사용하여 이미지가 저장될 경로를 지정합니다.
            final path = join(
              // 본 예제에서는 임시 디렉토리에 이미지를 저장합니다. `path_provider`
              // 플러그인을 사용하여 임시 디렉토리를 찾으세요.
              (await getTemporaryDirectory()).path,
              '${DateTime.now()}.png',
            );

            // 사진 촬영을 시도하고 저장되는 경로를 로그로 남깁니다.
            await _controller.takePicture(path);

            // 사진을 촬영하면, 새로운 화면으로 넘어갑니다.
            Navigator.push(
              context,
              MaterialPageRoute(
                builder: (context) => DisplayPictureScreen(imagePath: path),
              ),
            );
          } catch (e) {
            // 만약 에러가 발생하면, 콘솔에 에러 로그를 남깁니다.
            print(e);
          }
        },
      ),
    );
  }
}

// 사용자가 촬영한 사진을 보여주는 위젯
class DisplayPictureScreen extends StatelessWidget {
  final String imagePath;

  const DisplayPictureScreen({Key key, this.imagePath}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Display the Picture')),
      // 이미지는 디바이스에 파일로 저장됩니다. 이미지를 보여주기 위해 주어진 
      // 경로로 `Image.file`을 생성하세요.
      body: Image.file(File(imagePath)),
    );
  }
}
```
