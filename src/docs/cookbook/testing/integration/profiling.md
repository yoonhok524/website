---
title: 성능 프로파일링
prev:
  title: 통합 테스트 소개
  path: /docs/cookbook/testing/integration/introduction
next:
  title: 스크롤링
  path: /docs/cookbook/testing/integration/scrolling
---

모바일 앱에 있어 성능은 사용자 경험에 아주 중요한 요소입니다. 사용자는 끊김 현상이나 
"jank"와 같은 프레임을 건너뛰는 현상없이 부드러운 스크롤과 의미있는 애니메이션을 기대합니다.
어떻게 하면 우리가 만든 앱이 다양한 디바이스에서 끊김 현상없이 동작한다는 것을 보장할 수 있을까요?

두 가지 옵션이 있습니다. 먼저 여러 디바이스에서 직접 테스트해보는 것입니다. 이러한 
접근법은 작은 앱에서는 가능할지 몰라도, 앱이 커지면 점점 번거로워집니다. 다음으로,
특정 작업을 수행하고 그에 따른 성능 타임라인을 기록하는 통합 테스트를 수행하는 방법이 
있습니다. 그리고 그 결과를 통해 우리 앱의 특정 세션이 개선될 필요가 있는지 여부를 
결정할 수 있습니다.

본 예제에서는 특정 작업을 수행하는 동안 성능 타임라인을 측정하고 그 결과를 로컬 파일에 
저장하는 테스트 코드를 작성해볼 것입니다.

### 진행 단계

  1. 아이템 리스트를 스크롤하는 테스트 코드 작성
  2. 앱의 성능 기록
  3. 디스크에 결과 저장
  4. 테스트 수행
  5. 결과 검토

### 1. 아이템 리스트를 스크롤하는 테스트 코드 작성

In this recipe, we'll record the performance of an app as it scrolls through a
list of items. In order to focus on performance profiling, this recipe builds
upon the
[Scrolling in integration tests](/docs/cookbook/testing/integration/scrolling)
recipe.

Please follow the instructions in that recipe to create an app, instrument the
app, and write a test to verify everything works as expected.

### 2. 앱의 성능 기록

Next, we need to record the performance of the app as it scrolls through the
list. To achieve this task, we can use the
[`traceAction`]({{site.api}}/flutter/flutter_driver/FlutterDriver/traceAction.html)
method provided by the
[`FlutterDriver`]({{site.api}}/flutter/flutter_driver/FlutterDriver-class.html)
class.

This method runs the provided function and records a
[`Timeline`]({{site.api}}/flutter/flutter_driver/Timeline-class.html)
with detailed information about the performance of the app. In this example, we
provide a function that scrolls through the list of items, ensuring a specific
item is displayed. When the function completes, the `traceAction` method returns
a `Timeline`.

<!-- skip -->
```dart
// Record a performance timeline as we scroll through the list of items
final timeline = await driver.traceAction(() async {
  await driver.scrollUntilVisible(
    listFinder,
    itemFinder,
    dyScroll: -300.0,
  );

  expect(await driver.getText(itemFinder), 'Item 50');
});
```

### 3. 디스크에 결과 저장

Now that we've captured a performance timeline, we need a way to review it!
The `Timeline` object provides detailed information about all of the events that
took place, but it does not provide a convenient way to review the results.

Therefore, we can convert the `Timeline` into a
[`TimelineSummary`]({{site.api}}/flutter/flutter_driver/TimelineSummary-class.html).
The `TimelineSummary` can perform two tasks that make it easier to review the
results:

  1. It can write a json document on disk that summarizes the data contained
  within the `Timeline`. This summary includes information about the number of
  skipped frames, slowest build times, and more.
  2. It can save the complete `Timeline` as a json file on disk. This file can
  be opened with the Chrome browser's tracing tools found at
  [chrome://tracing](chrome://tracing).

<!-- skip -->
```dart
// Convert the Timeline into a TimelineSummary that's easier to read and
// understand.
final summary = new TimelineSummary.summarize(timeline);

// Then, save the summary to disk
summary.writeSummaryToFile('scrolling_summary', pretty: true);

// Optionally, write the entire timeline to disk in a json format. This
// file can be opened in the Chrome browser's tracing tools found by
// navigating to chrome://tracing.
summary.writeTimelineToFile('scrolling_timeline', pretty: true);
```

### 4. 테스트 수행

After we've configured our test to capture a performance `Timeline` and save a
summary of the results to disk, we can run the test with the following command:

```
flutter drive --target=test_driver/app.dart
```

### 5. 결과 검토

After the test completes successfully, the `build` directory at the root of
the project contains two files:

  1. `scrolling_summary.timeline_summary.json` contains the summary. Open
  the file with any text editor to review the information contained within.
  With a more advanced setup, we could save a summary every time the test
  runs and create a graph of the results.
  2. `scrolling_timeline.timeline.json` contains the complete timeline data.
  Open the file using the Chrome browser's tracing tools found at
  [chrome://tracing](chrome://tracing). The tracing tools provide a
  convenient interface for inspecting the timeline data in order to discover
  the source of a performance issue.

#### Summary Example

```json
{
  "average_frame_build_time_millis": 4.2592592592592595,
  "worst_frame_build_time_millis": 21.0,
  "missed_frame_build_budget_count": 2,
  "average_frame_rasterizer_time_millis": 5.518518518518518,
  "worst_frame_rasterizer_time_millis": 51.0,
  "missed_frame_rasterizer_budget_count": 10,
  "frame_count": 54,
  "frame_build_times": [
    6874,
    5019,
    3638
  ],
  "frame_rasterizer_times": [
    51955,
    8468,
    3129
  ]
}
```

### Complete example

```dart
import 'package:flutter_driver/flutter_driver.dart';
import 'package:test/test.dart';

void main() {
  group('Scrollable App', () {
    FlutterDriver driver;

    setUpAll(() async {
      driver = await FlutterDriver.connect();
    });

    tearDownAll(() async {
      if (driver != null) {
        driver.close();
      }
    });

    test('verifies the list contains a specific item', () async {
      final listFinder = find.byValueKey('long_list');
      final itemFinder = find.byValueKey('item_50_text');

      // Record a performance profile as we scroll through the list of items
      final timeline = await driver.traceAction(() async {
        await driver.scrollUntilVisible(
          listFinder,
          itemFinder,
          dyScroll: -300.0,
        );

        expect(await driver.getText(itemFinder), 'Item 50');
      });

      // Convert the Timeline into a TimelineSummary that's easier to read and
      // understand.
      final summary = new TimelineSummary.summarize(timeline);

      // Then, save the summary to disk
      summary.writeSummaryToFile('scrolling_summary', pretty: true);

      // Optionally, write the entire timeline to disk in a json format. This
      // file can be opened in the Chrome browser's tracing tools found by
      // navigating to chrome://tracing.
      summary.writeTimelineToFile('scrolling_timeline', pretty: true);
    });
  });
}
```
