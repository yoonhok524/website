---
title: 단위 테스트 소개
short-title: 소개
prev:
  title: 스크롤
  path: /docs/cookbook/testing/integration/scrolling
next:
  title: Mockito를 사용하여 의존성들에 대해 mock 객체 생성하기
  path: /docs/cookbook/testing/unit/mocking
---

새로운 기능을 추가하거나 기존 기능을 변경했을 때, 앱이 여전히 제대로 동작한다는 것을
어떻게 보장할 수 있을까요? 테스트 코드를 작성하세요.

단위 테스트는 하나의 함수, 메서드 혹은 클래스의 동작을 검증하기 위해 아주 유용합니다.
[`test`]({{site.pub-pkg}}/test) 패키지는 단위 테스트 작성을 위해 핵심 프레임워크를
제공하며,
[`flutter_test`]({{site.api}}/flutter/flutter_test/flutter_test-library.html)
패키지는 위젯 테스트를 위한 추가적인 유틸리티를 제공합니다.

본 예제에서는 `test` 패키지가 제공하는 핵심 기능들을 살펴볼 것입니다.
test 패키지에 대해 좀더 자세한 정보는 
[test 패키지 문서]({{site.github}}/dart-lang/test/blob/master/README.md)를
참고하세요.

## 진행 단계

  1. `test` 혹은 `flutter_test` 의존성 추가하기
  2. 테스트 파일 생성하기
  3. 테스트를 위한 클래스 생성하기
  4. 클래스 테스트를 위한 `test` 코드 작성하기
  5. 여러 테스트를 `group`으로 결합하기
  6. 테스트 수행하기

## 1. test 의존성 추가하기

만약 Flutter에 의존하지 않는 다트 패키지에서 작업하고 있다면 `test` 패키지를 import
하면 됩니다. test 패키지는 다트에서 테스트 코드를 작성하기 위한 핵심 기능을 제공합니다.
이러한 접근법은 웹, 서버, Flutter 앱에서 사용할 패키지를 개발할 때 아주 좋습니다.

```yaml
dev_dependencies:
  test: <latest_version>
```

## 2. 테스트 파일 생성하기

본 예제에서는 두 개의 파일을 만들 것입니다: `counter.dart`, `counter_test.dart`.

`counter.dart` 파일은 테스트 대상이 될 클래스를 포함하며, `lib` 폴더에 위치하게
됩니다. `counter_test.dart` 파일은 테스트 코드를 포함하며, `test` 폴더에
위치합니다.

일반적으로 테스트 파일들은 Flutter 앱이나 패키지의 최상단에 있는 `test` 폴더에 
위치해야 합니다.

여기까지 진행하였으면, 다음과 같은 폴더 구조가 되었을 것입니다:

```
counter_app/
  lib/
    counter.dart
  test/
    counter_test.dart
```

## 3. 테스트를 위한 클래스 생성하기

이제 테스트할 대상이 필요합니다. 기억하세요: 함수, 메서드, 클래스 모두 단위 
테스트의 최소 단위가 될 수 있습니다. 본 예제에서는 `lib/counter.dart` 파일에 
`Counter` 클래스를 생성합니다. 이 클래스는 `0`부터 시작하는 `value`의 증가, 
감소를 담당할 것입니다.

<!-- skip -->
```dart
class Counter {
  int value = 0;

  void increment() => value++;

  void decrement() => value--;
}
```

**참고:** 단순화하기 위해, 이 예제에서는 "테스트 주도 개발" 접근법을 따르지
않습니다. 만약 테스트 주도 개발에 더 익숙하다면, 그렇게 해도 좋습니다.

## 4. 클래스 테스트를 위한 `test` 코드 작성하기

`counter_test.dart` 파일에 첫 번째 단위 테스트를 작성해봅시다. 테스트 코드는
최상위 레벨의 `test` 함수를 통해 작성되며, 최상위 레벨의 `expect` 함수를 
사용하여 그 결과가 올바른지 체크할 수 있습니다. 이 함수들은 `test` 패키지의 
것들입니다.

<!-- skip -->
```dart
// test 패키지와 Counter 클래스를 import합니다.
import 'package:test/test.dart';
import 'package:counter_app/counter.dart';

void main() {
  test('Counter value should be incremented', () {
    final counter = Counter();

    counter.increment();

    expect(counter.value, 1);
  });
}
```

## 5. 여러 테스트를 `group`으로 결합하기

만약 관련있는 다른 테스트 코드들이 있다면, `test` 패키지에서 제공하는 
`group` 함수를 사용하여 결합할 수 있습니다.

<!-- skip -->
```dart
import 'package:test/test.dart';
import 'package:counter_app/counter.dart';

void main() {
  group('Counter', () {
    test('value should start at 0', () {
      expect(Counter().value, 0);
    });

    test('value should be incremented', () {
      final counter = Counter();

      counter.increment();

      expect(counter.value, 1);
    });

    test('value should be decremented', () {
      final counter = Counter();

      counter.decrement();

      expect(counter.value, -1);
    });
  });
}
```

### 6. 테스트 수행하기

이제 `Counter` 클래스와 함께 테스트 코드가 준비되었으니 실행해볼 차례입니다.

#### IntelliJ나 VSCode에서 테스트 수행하기

IntelliJ와 VSCode 용 Flutter 플러그인은 테스트 수행을 지원합니다. breakpoint 
설정 기능과 함께 가장 빠른 피드백 루프를 주기 때문에 테스트 코드를 작성할 때 
가장 좋은 옵션입니다.

  * **IntelliJ**
    1. `counter_test.dart` 파일을 엽니다.
    2. `Run` 메뉴를 선택합니다.
    3. `Run 'tests in counter_test.dart'` 옵션을 클릭합니다.
    4. *다른 대안으로, 플랫폼에 맞게 적절한 키보드 단축키를 사용할 수 있습니다.*
  * **VSCode**
    1. `counter_test.dart` 파일을 엽니다.
    2. `Debug` 메뉴를 선택합니다.
    3. `Start Debugging` 옵션을 클릭합니다.
    4. *다른 대안으로, 플랫폼에 맞게 적절한 키보드 단축키를 사용할 수 있습니다.*

#### 터미널에서 테스트 수행하기

프로젝트의 최상위 위치에서 아래 명령어를 사용하면 터미널에서도 테스트를 수행할 
수 있습니다:

```
flutter test test/counter_test.dart
```
