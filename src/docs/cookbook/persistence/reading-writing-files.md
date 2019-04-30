---
title: File 읽고 쓰기
prev:
  title: SQLite에 데이터 저장하기
  path: /docs/cookbook/persistence/sqlite
next:
  title: 디스크에 키-값 데이터 저장하기
  path: /docs/cookbook/persistence/key-value
---

가끔은 디스크에 파일을 읽고 쓰는 것이 유용할 때가 있습니다. 앱 런처간에 데이터를
관리하거나 나중에 오프라인 모드에서 사용하기 위한 목적으로 인터넷에서 파일을
다운로드 받을 수도 있습니다.

[`dart:io`]({{site.api}}/flutter/dart-io/dart-io-library.html) 라이브러리와
함께 [`path_provider` plugin]({{site.pub-pkg}}/path_provider)을 조합하여
사용하면 파일을 디스크에 저장할 수 있습니다.

## 진행 단계

  1. 올바른 로컬 경로 찾기
  2. 파일 경로에 대한 참조 값 생성하기
  3. 파일에 데이터 쓰기
  4. 파일로부터 데이터 읽기

## 1. 올바른 로컬 경로 찾기

본 예제에서는 화면에 counter를 보여줄 것입니다. counter가 변경되면 데이터를 디스크에
쓰고 앱이 로드되었을 때 다시 읽을 수 있습니다.
그럼 다음과 같은 질문을 할 수 있습니다: 데이터를 어디에 저장해야 할까?

[`path_provider`]({{site.pub-pkg}}/path_provider) 플러그인은 기기의 파일 시스템에서
사용되는 일반적인 경로에 접근하기 위해 플랫폼에 무관한 방법을 제공합니다. 현재 두 개의 
파일 시스템 경로에 접근하도록 지원합니다:

  * *임시 디렉토리:* 시스템이 언제든지 삭제할 수 있는 임시 디렉토리 (캐시). iOS에서는 
    [`NSTemporaryDirectory()`](https://developer.apple.com/reference/foundation/1409211-nstemporarydirectory),
    Android에서는 [`getCacheDir()`]({{site.android-dev}}/reference/android/content/Context#getCacheDir())에
    해당합니다.
  * *문서 디렉토리:* 파일을 저장하기 위해 해당 앱에서만 유일하게 접근할 수 있는 디렉토리.
    시스템은 해당 디렉토리를 앱이 삭제될 때에만 지웁니다. iOS에서는 `NSDocumentDirectory`,
    Android에서는 `AppData` 디렉토리가 해당됩니다. 

본 예제에서는 문서 디렉토리에 정보를 저장할 것이며 해당 경로는 아래 코드를
통해 찾을 수 있습니다.

<!-- skip -->
```dart
Future<String> get _localPath async {
  final directory = await getApplicationDocumentsDirectory();

  return directory.path;
}
```

## 2. 파일 경로에 대한 참조 값 생성하기

일단 파일이 어디에 저장되는지 알게 되면, 파일의 전체 경로에 대한 참조 값을 생성해야 합니다.
이를 위해 [dart:io]({{site.api}}/flutter/dart-io/dart-io-library.html) 라이브러리에서
제공하는 [`File`]({{site.api}}/flutter/dart-io/File-class.html) 클래스를 사용하면 됩니다.

<!-- skip -->
```dart
Future<File> get _localFile async {
  final path = await _localPath;
  return File('$path/counter.txt');
}
```

## 3. 파일에 데이터 쓰기

이제 작업할 수 있는 `File`이 생겼습니다. 여기에 데이터를 읽고 써보겠습니다.
먼저 파일에 데이터를 작성해봅시다. 본 예제에서는 counter를 다루기 때문에,
간단히 integer 값을 String 형태로 저장해보겠습니다.

<!-- skip -->
```dart
Future<File> writeCounter(int counter) async {
  final file = await _localFile;

  // 파일 쓰기
  return file.writeAsString('$counter');
}
```

## 4. 파일로부터 데이터 읽기

이제 디스크에 읽을 수 있는 데이터가 존재합니다. 다시 한 번
`File` 클래스를 사용합시다.

<!-- skip -->
```dart
Future<int> readCounter() async {
  try {
    final file = await _localFile;

    // 파일 읽기
    String contents = await file.readAsString();

    return int.parse(contents);
  } catch (e) {
    // 에러가 발생할 경우 0을 반환
    return 0;
  }
}
```

## 테스팅

File과 상호작용하는 코드를 테스트하기 위해서는 `MethodChannel`를 Mock 호출을 해야 합니다.
`MethodChannel`은 Flutter가 호스트 플랫폼과 통신하기 위해 사용하는 클래스입니다.

이 테스트 코드에서는 디바이스의 파일 시스템과 상호 작용할 수 없습니다.
테스트 환경의 파일 시스템을 사용 해야 합니다.

테스트 파일에 메서드 콜을 흉내내는 `setupAll` 이라는 Mock 함수를 제공하겠습니다.
이 함수는 테스트가 시작되기 전에 수행합니다.

<!-- skip -->
```dart
setUpAll(() async {
  // 임시 디렉토리를 생성합니다.
  final directory = await Directory.systemTemp.createTemp();

  // path_provider 플러그인을 위해 MethodChannel Mock 객체를 생성하세요
  const MethodChannel('plugins.flutter.io/path_provider')
      .setMockMethodCallHandler((MethodCall methodCall) async {
    // 만약 앱의 문서 디렉토리가 필요하다면, 테스트 환경에서는 그 대신 임시 디렉토리
    // 경로를 반환하세요.
    if (methodCall.method == 'getApplicationDocumentsDirectory') {
      return directory.path;
    }
    return null;
  });
});
```

## 완성된 예제

```dart
import 'dart:async';
import 'dart:io';

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:path_provider/path_provider.dart';

void main() {
  runApp(
    MaterialApp(
      title: 'Reading and Writing Files',
      home: FlutterDemo(storage: CounterStorage()),
    ),
  );
}

class CounterStorage {
  Future<String> get _localPath async {
    final directory = await getApplicationDocumentsDirectory();

    return directory.path;
  }

  Future<File> get _localFile async {
    final path = await _localPath;
    return File('$path/counter.txt');
  }

  Future<int> readCounter() async {
    try {
      final file = await _localFile;

      // 파일 읽기
      String contents = await file.readAsString();

      return int.parse(contents);
    } catch (e) {
      // 에러가 발생할 경우 0을 반환
      return 0;
    }
  }

  Future<File> writeCounter(int counter) async {
    final file = await _localFile;

    // 파일 쓰기
    return file.writeAsString('$counter');
  }
}

class FlutterDemo extends StatefulWidget {
  final CounterStorage storage;

  FlutterDemo({Key key, @required this.storage}) : super(key: key);

  @override
  _FlutterDemoState createState() => _FlutterDemoState();
}

class _FlutterDemoState extends State<FlutterDemo> {
  int _counter;

  @override
  void initState() {
    super.initState();
    widget.storage.readCounter().then((int value) {
      setState(() {
        _counter = value;
      });
    });
  }

  Future<File> _incrementCounter() {
    setState(() {
      _counter++;
    });

    // 파일에 String 타입으로 변수 값 쓰기
    return widget.storage.writeCounter(_counter);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Reading and Writing Files')),
      body: Center(
        child: Text(
          'Button tapped $_counter time${_counter == 1 ? '' : 's'}.',
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ),
    );
  }
}
```
