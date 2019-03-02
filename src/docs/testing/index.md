---
title: Flutter 앱 테스트하기
---

앱의 기능이 많아질수록, 수동으로 테스트하기는 더 어려워집니다.
자동 테스트는 기능과 버그 개선 속도를 유지하여 앱을 출시하기 전에 정상적으로 작동할 수 있도록 보장해드립니다.


자동 테스트는 아래의 몇 가지 카테고리로 나뉩니다:

- _단위 테스트_ 는 단위 기능, 방법 또는 클래스를 테스트합니다. 테스트 유닛의 외부 의존성은 일반적으로 흉내내어 집니다 ([mocked out](/cookbook/testing/mocking)).
  일반적으로 단위 테스트는 테스트가 실행되고 있는 동안 저장소를 읽거나 쓰지 않으며, 스크린 렌더링을 하지 않고, 외부에서 사용자 액션을 받지 않습니다.
  단위 테스트의 목적은 다양한 조건 아래에서 논리 단위의 정확성을 검증하는 것입니다.
  _위젯 테스트_(다른 UI 프레임워크에서는 _컴포넌트 테스트_ 라 불립니다) 는 단일 위젯을 테스트합니다.
  위젯 테스트는 다수의 클래스를 포함하며 적합한 위젯 생명주기 컨텍스트를 제공하는 테스트 환경을 요구합니다.
  예를 들어 사용자의 행동과 이벤트를 받고 응답할 수 있어야 하며 레이아웃을 수행하고 자식 위젯을 인스턴스화 하는 것이 있습니다.
  그러므로 위젯 테스트는 단위 테스트보다 더 포괄적이라고 볼 수 있습니다.
  하지만 단위 테스트처럼 위젯 테스트의 환경은 전체 UI 시스템보다 훨씬 간단한 구현으로 대체됩니다.
  위젯 테스트의 목적은 위젯 UI가 예상한대로 보이는지, 상호 작용하는지 검증하는 것입니다.

- [_통합 테스트_](https://en.wikipedia.org/wiki/Integration_testing)
  완성된 앱이나 앱의 상당 부분을 테스트합니다. 일반적으로 _통합 테스트_는 실제 장치나 
  iOS 시뮬레이터와 안드로이드 에뮬레이터와 같은 OS 에뮬레이터에서 실행됩니다.
  테스트 대상 앱은 일반적으로 결과가 왜곡되지 않도록 테스트 드라이버 코드에서 격리됩니다.
  
  통합의 목표
  테스트는 앱이 전체적으로 올바르게 작동하는지, 모든 위젯이 예상대로 서로 통합되어있는지 확인하는 것입니다.
  또한 통합 테스트를 앱의 성능 검증을 위해 사용할 수 있습니다.

다음은 다른 종류의 테스트들 간의 선택에 관한 표입니다

|                      | 단위   | 위젯 | 통합 |
|----------------------|--------|--------|-------------|
| **신뢰도**       | 낮음    | 높음 | 가장 높음     |
| **유지 비용** | 낮음    | 높음 | 가장 높음     |
| **의존성**     | 조금    | 높음   | 가장 높음        |
| **실행 속도**  | 빠름  | 느림 | 가장 느림     |
{:.table.table-striped}

{{site.alert.tip}}
  철저히 테스트된 앱은 코드 적용범위[(code coverage)](https://en.wikipedia.org/wiki/Code_coverage)에 의해 추적되는 많은 단위와 위젯테스트를 가지고 있으며,
  모든 중요한 사용 사례를 커버하기에 충분한 통합 테스트를 가지고 있습니다.
{{site.alert.end}}


## Unit testing

Some Flutter libraries, such as `dart:ui`, aren't available in the standalone
Dart VM that ships with the default Dart SDK. The `flutter test` command lets
you run your tests in a local Dart VM with a headless version of the Flutter
Engine, which supplies these libraries. Using this command you can run any test,
whether it depends on Flutter libraries or not.

Write a Flutter unit test as a normal `package:test` test. Writing unit tests
using `package:test` is documented on the [Dart
testing]({{site.github}}/dart-lang/test/blob/master/README.md)
repo.

예제:

Add this file to `test/unit_test.dart`:

{% prettify dart %}
import 'package:test/test.dart';

void main() {
  test('my first unit test', () {
    /**highlight*/var answer = 42;/*-highlight*/
    expect(answer, 42);
  });
}
{% endprettify %}

In addition, you must add the following block to your `pubspec.yaml`:

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  test: ^1.5.1
```

{{site.alert.note}}
  The dev dependency on `flutter_test` is required even if
  your test doesn't explicitly import `flutter_test`,
  because the test framework uses `flutter_test` behind the scenes.
{{site.alert.end}}

To run the test, run `flutter test test/unit_test.dart` from your
project directory (not from the `test` subdirectory).

To run all your tests, run `flutter test` from your project directory.

For information on how to create mock services, see
[Mock dependencies using Mockito](/cookbook/testing/mocking).


## Widget testing

You implement a widget test in a similar way as a unit test. To perform an
interaction with a widget in your test, use the
[`WidgetTester`]({{site.api}}/flutter/flutter_test/WidgetTester-class.html)
utility that Flutter provides. For example, you can send tap and scroll
gestures. You can also use
[`WidgetTester`]({{site.api}}/flutter/flutter_test/WidgetTester-class.html)
to find child widgets in the widget tree, read text, and verify that the values
of widget properties are correct.

Example:

Add this file to `test/widget_test.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  testWidgets('my first widget test', (WidgetTester tester) async {
    // You can use keys to locate the widget you need to test
    var sliderKey = UniqueKey();
    var value = 0.0;

    // Tells the tester to build a UI based on the widget tree passed to it
    await tester.pumpWidget(
      StatefulBuilder(
        builder: (BuildContext context, StateSetter setState) {
          return MaterialApp(
            home: Material(
              child: Center(
                child: Slider(
                  key: sliderKey,
                  value: value,
                  onChanged: (double newValue) {
                    setState(() {
                      value = newValue;
                    });
                  },
                ),
              ),
            ),
          );
        },
      ),
    );
    expect(value, equals(0.0));

    // Taps on the widget found by key
    await tester.tap(find.byKey(sliderKey));

    // Verifies that the widget updated the value correctly
    expect(value, equals(0.5));
  });
}
```

Run `flutter test test/widget_test.dart`.

Check out [`package:flutter_test` API]({{site.api}}/flutter/flutter_test/flutter_test-library.html)
for all the utilities available for widget testing.

To help debug widget tests, you can use the [`debugDumpApp()`]({{site.api}}/flutter/widgets/debugDumpApp.html) function to visualize the UI state of your test or simply `flutter run test/widget_test.dart` to see your test run in your preferred runtime environment such as a simulator or a device. During a `flutter run` session on a widget test, you can also interactively tap parts of the screen for the Flutter tool to print the suggested [`Finder`]({{site.api}}/flutter/flutter_test/CommonFinders-class.html).


## Integration testing

A Flutter integration test is also written using `package:test`. A full test is a
pair - a test script and a Flutter app instrumented to receive commands
from the test. Unlike unit and widget tests, integration test code does not run
in the same process as the app that's being tested. Instead, the tested
app is launched on a _real device_ or in an _emulator_ (e.g. Android
Emulator or iOS Simulator). The test script runs on your computer. It connects
to the app and issues commands to the app to perform various
user actions. This is known as "driving" the app. Flutter provides tools
and APIs, collectively referred to as _Flutter Driver_, to do just that.

> If you are familiar with Selenium/WebDriver (web), Espresso (Android) or UI
> Automation (iOS), then Flutter Driver is Flutter's equivalent to those
> integration testing tools. In addition, Flutter Driver provides API for
> recording performance traces (a.k.a. the _timeline_) from actions performed
> by the test.

Flutter Driver is:

* a command-line tool `flutter drive`
* a package `package:flutter_driver` ([API]({{site.api}}/flutter/flutter_driver/FlutterDriver-class.html))

Together, the two allow you to:

* create instrumented app for integration testing
* write a test
* run the test

### Adding the flutter_driver dependency

To use `flutter_driver`, you must add the following block to your `pubspec.yaml`:

```yaml
dev_dependencies:
  flutter_driver:
    sdk: flutter
  flutter_test:
    sdk: flutter
  test: ^1.5.1
```

### Creating instrumented Flutter apps

An instrumented app is a Flutter app that has the Flutter Driver
_extension_ enabled. To enable the extension call `enableFlutterDriverExtension()`.

Example:

Let's assume you have an app with the entry point in
`my_app/lib/main.dart`. To create an instrumented version of it, create a Dart
file under `my_app/test_driver/`. Name it after the feature you are testing;
let's go for `user_list_scrolling.dart` located in my_app/test_driver/:

```dart
// This line imports the extension
import 'package:flutter_driver/driver_extension.dart';

void main() {
  // This line enables the extension
  enableFlutterDriverExtension();

  // Call the `main()` of your app or call `runApp` with whatever widget
  // you are interested in testing.
}
```

### Writing integration tests

An integration test is a plain `package:test` test that uses the Flutter Driver
API to tell the app what to do and then verifies that the app
did it.

Example:

Just for fun let's also make our test record the performance timeline. Let's
create a test file `user_list_scrolling_test.dart` located in `my_app/test_driver/`:

```dart
import 'dart:async';

// Imports the Flutter Driver API
import 'package:flutter_driver/flutter_driver.dart';
import 'package:test/test.dart';

void main() {
  group('scrolling performance test', () {
    FlutterDriver driver;

    setUpAll(() async {
      // Connects to the app
      driver = await FlutterDriver.connect();
    });

    tearDownAll(() async {
      if (driver != null) {
        // Closes the connection
        driver.close();
      }
    });

    test('measure', () async {
      // Record the performance timeline of things that happen inside the closure
      Timeline timeline = await driver.traceAction(() async {
        // Find the scrollable user list
        SerializableFinder userList = find.byValueKey('user-list');

        // Scroll down 5 times
        for (int i = 0; i < 5; i++) {
          // Scroll 300 pixels down, for 300 millis
          await driver.scroll(
              userList, 0.0, -300.0, Duration(milliseconds: 300));

          // Emulate a user's finger taking its time to go back to the original
          // position before the next scroll
          await Future<void>.delayed(Duration(milliseconds: 500));
        }

        // Scroll up 5 times
        for (int i = 0; i < 5; i++) {
          await driver.scroll(
              userList, 0.0, 300.0, Duration(milliseconds: 300));
          await Future<void>.delayed(Duration(milliseconds: 500));
        }
      });

      // The `timeline` object contains all the performance data recorded during
      // the scrolling session. It can be digested into a handful of useful
      // aggregate numbers, such as "average frame build time".
      TimelineSummary summary = TimelineSummary.summarize(timeline);

      // The following line saves the timeline summary to a JSON file.
      summary.writeSummaryToFile('scrolling_performance', pretty: true);

      // The following line saves the raw timeline data as JSON.
      summary.writeTimelineToFile('scrolling_performance', pretty: true);
    });
  });
}
```

### Running integration tests

To run the test on an Android device, connect the device via USB to your
computer and enable USB debugging. Then run the following command:

```
flutter drive --target=my_app/test_driver/user_list_scrolling.dart
```

This command:

* builds the `--target` app and install it on the device
* launches the app
* runs the `user_list_scrolling_test.dart` test located in `my_app/test_driver/`

You might be wondering how the command finds the correct test file. The
`flutter drive` command uses a convention to look for the test file in the same
directory as the instrumented `--target` app that has the same file name
but for the `_test` suffix in it.

## Continuous integration and testing

For information on continuous deployment and testing, see the following:
아래를 통해 지속 개발과 테스트에 관한 정보를 확인할 수 있습니다.

* [Continuous Delivery using fastlane with
  Flutter](/docs/deployment/fastlane-cd/)
* [Test Flutter apps on
  Travis]({{site.flutter-medium}}/test-flutter-apps-on-travis-3fd5142ecd8c)
* [GitLab Continuous Integration
  (GitLab CI/CD](https://docs.gitlab.com/ee/ci/README.html#doc-nav).
  You'll need to create and configure a `.gitlab-ci.yml` file. You can [find an
  example](https://raw.githubusercontent.com/brianegan/flutter_redux/master/.gitlab-ci.yml)
  in the [flutter_redux library]({{site.github}}/brianegan/flutter_redux).
* [Codemagic CI/CD for Flutter](https://blog.codemagic.io/getting-started-with-codemagic/)
* [Flutter CI/CD with Bitrise](https://devcenter.bitrise.io/getting-started/getting-started-with-flutter-apps/)
