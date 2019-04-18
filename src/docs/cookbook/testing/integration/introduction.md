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

통합 테스트는 쌍으로 동작합니다: 먼저 instrumented 앱을 실제 기기나 에뮬레이터에 설치한 다음에
별도의 테스트 suite에서 앱을 "구동"하고, 모든 것이 올바로 동작하는지 확인합니다.

이러한 테스트 쌍을 만들기 위해, 
[flutter_driver]({{site.api}}/flutter/flutter_driver/flutter_driver-library.html) 
패키지를 사용할 수 있습니다. 이 패키지는 instrumented 앱을 만들고 테스트 suite에서
구동할 수 있는 도구를 제공합니다.

본 예제에서는 카운터 앱을 테스트하는 방법을 배울 것입니다. 통합 테스트를
셋업하는 방법, 앱에 특정 문구가 제대로 보이는지 확인하는 방법, 특정 위젯을 탭하는 방법,
통합 테스트를 구동하는 방법 등을 다룰 것입니다.

### 진행 단계

  1. 테스트할 앱 만들기
  2. `flutter_driver` 의존성 추가하기
  3. 테스트 파일 생성하기
  4. Instrument the app
  5. 통합 테스트 작성하기
  6. 통합 테스트 수행하기

### 1. 테스트할 앱 만들기

First, we'll create an app that we can test! In this example, we'll test the
counter app produced by the `flutter create` command. This app allows
a user to tap on a button to increase a counter.

Furthermore, we'll also need to provide a
[`ValueKey`]({{site.api}}/flutter/foundation/ValueKey-class.html) to
the `Text` and `FloatingActionButton` Widgets. This allows us to identify
and interact with these specific Widgets inside the test suite.

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
              // Provide a Key to this specific Text Widget. This allows us
              // to identify this specific Widget from inside our test suite and
              // read the text.
              key: Key('counter'),
              style: Theme.of(context).textTheme.display1,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        // Provide a Key to this the button. This allows us to find this
        // specific button and tap it inside the test suite.
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

Next, we'll need the `flutter_driver` package to write integration tests. We
can add the `flutter_driver` dependency to the `dev_dependencies` section of
our apps's `pubspec.yaml` file.

We also add the `test` dependency in order to use actual test functions and
assertions.

```yaml
dev_dependencies:
  flutter_driver:
    sdk: flutter
  test: any
```

### 3. 테스트 파일 생성하기

Unlike unit and widget tests, integration test suites do not run in the same
process as the app being tested. Therefore, we need to create two files that
reside in the same directory. By convention, the directory is named
`test_driver`.

  1. The first file contains an "instrumented" version of the app. The
  instrumentation allows us to "drive" the app and record performance profiles
  from a test suite. This file can be given any name that makes sense. For this
  example, create a file called `test_driver/app.dart`.
  2. The second file contains the test suite, which drives the app and verifies
  it works as expected. The test suite can also record performance profiles.
  The name of the test file must correspond to the name of the file that
  contains the instrumented app, with `_test` added at the end. Therefore,
  create a second file called `test_driver/app_test.dart`.

This leaves us with the following directory structure:

```
counter_app/
  lib/
    main.dart
  test_driver/
    app.dart
    app_test.dart
```


### 4. 앱 실행하기

Now, we can instrument the app. This will involve two steps:

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

Now that we have an instrumented app, we can write tests for it! This
will involve four steps:

  1. Create
  [`SeralizableFinders`]({{site.api}}/flutter/flutter_driver/CommonFinders-class.html)
  to locate specific Widgets
  2. Connect to the app before our tests run in the `setUpAll` function
  3. Test the important scenarios
  4. Disconnect from the app in the `teardownAll` function after our tests
  complete

```dart
// Imports the Flutter Driver API
import 'package:flutter_driver/flutter_driver.dart';
import 'package:test/test.dart';

void main() {
  group('Counter App', () {
    // First, define the Finders. We can use these to locate Widgets from the
    // test suite. 참고: the Strings provided to the `byValueKey` method must
    // be the same as the Strings we used for the Keys in step 1.
    final counterTextFinder = find.byValueKey('counter');
    final buttonFinder = find.byValueKey('increment');

    FlutterDriver driver;

    // Connect to the Flutter driver before running any tests
    setUpAll(() async {
      driver = await FlutterDriver.connect();
    });

    // Close the connection to the driver after the tests have completed
    tearDownAll(() async {
      if (driver != null) {
        driver.close();
      }
    });

    test('starts at 0', () async {
      // Use the `driver.getText` method to verify the counter starts at 0.
      expect(await driver.getText(counterTextFinder), "0");
    });

    test('increments the counter', () async {
      // First, tap on the button
      await driver.tap(buttonFinder);

      // Then, verify the counter text has been incremented by 1
      expect(await driver.getText(counterTextFinder), "1");
    });
  });
}
```

### 6. 테스트 수행하기

Now that we have an instrumented app and a test suite, we can run the tests!
First, be sure to launch an Android Emulator, iOS Simulator, or connect your
computer to a real iOS / Android device.

Then, run the following command from the root of the project:

```
flutter drive --target=test_driver/app.dart
```

This command:

  1. builds the `--target` app and installs it on the emulator / device
  2. launches the app
  3. runs the `app_test.dart` test suite located in `test_driver/` folder
