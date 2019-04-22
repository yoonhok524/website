---
title: 통합 테스트 소개
short-title: 소개
prev:
  title: 카메라 사진 찍기
  path: /docs/cookbook/plugins/picture-using-camera
next:
  title: 성능 프로파일링
  path: /docs/cookbook/testing/integration/profiling
---

단위 테스트와 위젯 테스트는 개별 클래스, 함수, 위젯을 테스트하기 유용합니다. 하지만,
이러한 각 개별 요소들이 실제 기기에서 어떻게 같이 어우러져 동작하는지 테스트하거나
실제 기기에서 동작하는 앱의 성능을 측정하는 것은 불가능합니다. 이러한 작업들은
*통합 테스트*를 통해 수행될 수 있습니다.

통합 테스트는 쌍으로 동작합니다: 먼저 계측 앱을 실제 기기나 에뮬레이터에 설치한 다음에
별도의 테스트 모음에서 앱을 "구동"하고, 모든 것이 올바로 동작하는지 확인합니다.

이러한 테스트 쌍을 만들기 위해, 
[flutter_driver]({{site.api}}/flutter/flutter_driver/flutter_driver-library.html) 
패키지를 사용할 수 있습니다. 이 패키지는 계측 앱을 만들고 테스트 모음에서
구동할 수 있는 도구를 제공합니다.

본 예제에서는 카운터 앱을 테스트하는 방법을 배울 것입니다. 통합 테스트를
셋업하는 방법, 앱에 특정 문구가 제대로 보이는지 확인하는 방법, 특정 위젯을 탭하는 방법,
통합 테스트를 구동하는 방법 등을 다룰 것입니다.

### 진행 단계

  1. 테스트할 앱 만들기
  2. `flutter_driver` 의존성 추가하기
  3. 테스트 파일 생성하기
  4. 앱 계측하기
  5. 통합 테스트 작성하기
  6. 통합 테스트 수행하기

### 1. 테스트할 앱 만들기

먼저 우리가 테스트할 앱을 만들어보겠습니다! 본 예제에서는 `flutter create` 명령어로
자동 생성되는 카운터 앱을 테스트하겠습니다. 사용자가 버튼을 누르면 카운터를 증가시키는
간단한 앱입니다.

또한, `Text`와 `FloatingActionButton` 위젯에
[`ValueKey`]({{site.api}}/flutter/foundation/ValueKey-class.html)를
제공해야 하는데, 이를 통해 테스트 모음 내부에서 특정 위젯을 식별하고 상호작용할 수 
있습니다.

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Counter App',
      home: MyHomePage(title: 'Counter App Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);

  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              // 특정 Text 위젯에 Key를 설정하세요. 이를 통해 테스트 모음에서
              // 이 위젯을 식별하고 문자열을 읽을 수 있게 해줍니다.
              key: Key('counter'),
              style: Theme.of(context).textTheme.display1,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        // 이 버튼에 Key를 설정하세요. 이를 통해 테스트 모음에서 버튼을 
        // 찾고 누를 수 있게 해줍니다.
        key: Key('increment'),
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ),
    );
  }
}
```

### 2. `flutter_driver` 의존성 추가하기

다음으로 통합 테스트를 작성하기 위해 `flutter_driver` 패키지가 필요합니다. 
앱의 `pubspec.yaml` 파일의 `dev_dependencies` 영역에 `flutter_driver` 의존성을
추가하세요.

실제 테스트 함수와 assertion을 사용하기 위해 `test` 의존성도 추가해주세요.

```yaml
dev_dependencies:
  flutter_driver:
    sdk: flutter
  test: any
```

### 3. 테스트 파일 생성하기

단위 테스트와 위젯 테스트와는 다르게, 통합 테스트 모음은 테스트 중인 앱과 동일한
프로세스에서 실행되지 않습니다. 그러므로 동일한 디렉토리에 두 개의 파일을 만들 필요가
있습니다. 관습적으로 디렉토리 이름은 `test_driver`로 합니다.

  1. 첫 번째 파일은 앱의 "계측" 버전을 포함합니다. 계측은 테스트 모음으로부터 앱을 
  "구동"하고 성능 프로파일을 기록합니다. 이 파일에는 의미있는 이름을 지정할 수 있는데, 
  본 예제에서는 `test_driver/app.dart` 이름으로 파일을 생성하겠습니다.
  2. 두 번째 파일은 앱을 구동하고 기대했던대로 동작하는지 검증하는 테스트 모음을 
  포함합니다. 테스트 모음은 성능 프로파일을 기록할 수도 있습니다. 테스트 파일의 이름은
  반드시 계측 앱을 포함하는 파일의 이름과 쌍을 이뤄서 `_test`로 끝나도록 지정해야
  합니다. 그러므로 두 번째 파일은 `test_driver/app_test.dart`로 생성하세요.

이로 인해 다음과 같은 디렉토리 구조가 생깁니다:

```
counter_app/
  lib/
    main.dart
  test_driver/
    app.dart
    app_test.dart
```


### 4. 앱 계측하기

이제 앱을 게측할 수 있습니다. 이를 위해 다음 두 단계가 필요합니다:

  1. Flutter 드라이버 extensions을 활성화합니다.
  2. 앱을 실행합니다.

아래 코드를 `test_driver/app.dart` 파일에 추가합니다.

<!-- skip -->
```dart
import 'package:flutter_driver/driver_extension.dart';
import 'package:counter_app/main.dart' as app;

void main() {
  // Extension을 활성화합니다.
  enableFlutterDriverExtension();

  // 앱의 `main()` 함수를 호출하거나 테스트하고 싶은 위젯을 인자로 넣어 `runApp`을
  // 호출합니다.
  app.main();
}
```

### 5. 통합 테스트 작성하기

계측 앱이 준비되었으므로, 테스트 코드를 작성할 수 있습니다! 이는 다음 네 단계로 
진행됩니다:

  1. 특정 위젯을 위치시키기 위해 
  [`SeralizableFinders`]({{site.api}}/flutter/flutter_driver/CommonFinders-class.html)
  를 생성합니다.
  2. 테스트를 실행하기 전에, setUpAll 함수에서 앱과 연결하세요.
  3. 중요한 시나리오를 테스트합니다.
  4. 테스트가 완료되면, `teardownAll` 함수에서 앱 연결을 끊습니다.

```dart
// Flutter Driver API를 import 합니다.
import 'package:flutter_driver/flutter_driver.dart';
import 'package:test/test.dart';

void main() {
  group('Counter App', () {
    // 먼저 Finders를 정의합니다. 테스트 모음의 위젯들을 위치시키기 위해 Finder를
    // 사용할 것입니다. 참고: `byValueKey` 메서드에 인자로 넘겨줄 문자열은 step 1에서
    // 사용한 Key와 동일해야 합니다. 
    final counterTextFinder = find.byValueKey('counter');
    final buttonFinder = find.byValueKey('increment');

    FlutterDriver driver;

    // 테스트를 수행하기 전에 Flutter driver와 연결합니다.
    setUpAll(() async {
      driver = await FlutterDriver.connect();
    });

    // 테스트 완료 후 driver와의 연결을 종료합니다.
    tearDownAll(() async {
      if (driver != null) {
        driver.close();
      }
    });

    test('starts at 0', () async {
      // `driver.getText` 메서드를 사용하여 카운터가 0부터 시작하는지 확인합니다.
      expect(await driver.getText(counterTextFinder), "0");
    });

    test('increments the counter', () async {
      // 먼저 버튼을 누릅니다.
      await driver.tap(buttonFinder);

      // 그리고 카운터 값이 1 만큼 증가했는지 확인합니다.
      expect(await driver.getText(counterTextFinder), "1");
    });
  });
}
```

### 6. 테스트 수행하기

계측 앱과 테스트 모음이 모두 준비되었습니다. 이제 테스트를 실행할 수 있습니다!
먼저, 컴퓨터와 안드로이드 에뮬레이터, iOS 시뮬레이터 혹은 실제 iOS / 안드로이드 기기가
연결되었는지 확인하세요.

확인되었으면, 프로젝트의 최상위 위치에서 아래 명령어를 수행하세요:

```
flutter drive --target=test_driver/app.dart
```

이 명령어는:

  1. `--target` 앱을 빌드하고 에뮬레이터 / 기기에 설치합니다.
  2. 앱을 시작시킵니다.
  3. `test_driver/` 폴더에 위치한 `app_test.dart` 테스트 모음을 실행합니다.
