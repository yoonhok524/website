---
title: Flutter 앱 테스트하기
---

앱의 기능이 많아질수록, 수동으로 테스트하기는 더 어려워집니다.
자동 테스트는 기능과 버그 개선 속도를 유지하여 앱을 출시하기 전에 정상적으로 작동할 수 있도록 보장해드립니다.

자동 테스트는 아래의 몇 가지 카테고리로 나뉩니다:

- [_단위 테스트_](#unit-tests) 는 단위 기능, 방법 또는 클래스를 테스트합니다.
- [_위젯 테스트_](#widget-tests) (다른 UI 프레임워크에서는 _컴포넌트 테스트_ 라 불립니다) 는 단일 위젯을 테스트합니다.
- [_통합 테스트_](#integration-tests) 는 완성된 앱이나 앱의 상당 부분을 테스트합니다.
  
일반적으로, 철저히 테스트된 앱은 코드 적용범위[(code coverage)](https://en.wikipedia.org/wiki/Code_coverage)에 의해 추적되는 많은 단위 테스트와 위젯 테스트를 가지고 있으며, 모든 중요한 사용 사례를 커버하기에 충분한 통합 테스트를 가지고 있습니다. 이러한 권고의 근거는 테스트의 종류에 따라 아래 표와 같은 상충관계가 있다는 사실에 있습니다.

 
다음은 다른 종류의 테스트들 간의 선택에 관한 표입니다

|                      | 단위   | 위젯 | 통합 |
|----------------------|--------|--------|-------------|
| **신뢰도**       | 낮음    | 높음 | 가장 높음     |
| **유지 비용** | 낮음    | 높음 | 가장 높음     |
| **의존성**     | 조금    | 높음   | 가장 높음        |
| **실행 속도**  | 빠름  | 느림 | 가장 느림     |
{:.table.table-striped}


## 단위 테스트

_단위 테스트_ 는 단위 기능, 방법 또는 클래스를 테스트합니다. 단위 테스트의 목적은 다양한 조건 아래에서 로직 단위의 정확성을 검증하는 것입니다.
일반적으로 테스트 되는 단위 로직의 외부 의존성은 모의 객체로 대체[(mocked out)](/cookbook/testing/mocking)합니다.
일반적으로 단위 테스트는 테스트가 실행되고 있는 동안 저장소를 읽거나 쓰지 않으며, 스크린 렌더링을 하지 않고, 외부에서 사용자 액션을 받지 않습니다.

### 사용 방법

{% include testing_toc.md type='unit' %} 

## 위젯 테스트

_위젯 테스트_(다른 UI 프레임워크에서는 _컴포넌트 테스트_ 라 불립니다) 는 단일 위젯을 테스트합니다. 위젯 테스트의 목적은 위젯 UI가 예상한대로 보이고 상호 작용하는지 검증하는 것입니다.
위젯 테스트는 다수의 클래스를 포함하며 적합한 위젯 생명주기 컨텍스트를 제공하는 테스트 환경을 요구합니다.

예를 들어, 테스트되는 위젯은 사용자의 행동과 이벤트를 받고 응답할 수 있어야 하고 레이아웃이 동작하는게 가능해야 하며, 자식 위젯을 인스턴스화 할 수 있어야 합니다.
그러므로 위젯 테스트는 단위 테스트보다 더 포괄적이라고 볼 수 있습니다.
하지만 단위 테스트처럼 위젯 테스트의 환경은 전체 UI 시스템보다 훨씬 간단한 구현으로 대체됩니다.

### 사용 방법

{% include testing_toc.md type='widget' %} 

## 통합 테스트

_통합 테스트_는 완성된 앱이나 앱의 상당 부분을 테스트합니다. 통합 테스트의 목표는 테스트는 앱이 전체적으로 올바르게 작동하는지, 모든 위젯이 예상대로 서로 통합되어있는지 확인하는 것입니다.
더불어 통합 테스트를 앱의 성능 검증을 위해 사용할 수 있습니다.

일반적으로 _통합 테스트_는 실제 장치나 
iOS 시뮬레이터와 Android 에뮬레이터와 같은 OS 에뮬레이터에서 실행됩니다.
테스트 대상 앱은 일반적으로 결과가 왜곡되지 않도록 테스트 드라이버 코드에서 격리됩니다.

### 사용 방법

{% include testing_toc.md type='integration' %}
  
## 연속 통합 서비스

연속 통합 (CI) 서비스는 새로운 코드 변경을 푸시할 때 당신의 테스트를 자동적으로 시행하는 것을 허용합니다.
이는 코드 변경이 예상대로 작동하는지, 버그가 생기지 않는지에 대한 여부를 때에 맞춰 피드백합니다.


다양한 연속 통합 서비스에 대한 테스트 실행에 대한 정보는 다음을 참조하시기 바랍니다.

* [fastlane을 이용한 Flutter의 지속적인 업데이트
  Flutter](/docs/deployment/fastlane-cd/)
* [Travis에서 Flutter 앱 테스트하기]({{site.flutter-medium}}/test-flutter-apps-on-travis-3fd5142ecd8c)
* [GitLab 연속 통합
  (GitLab CI/CD)](https://docs.gitlab.com/ee/ci/README.html#doc-nav).
  당신은 `.gitlab-ci.yml`파일을 생성하고 설정해야 할 필요가 있습니다. [예제 찾기](https://raw.githubusercontent.com/brianegan/flutter_redux/master/.gitlab-ci.yml)를 [flutter_redux library]({{site.github}}/brianegan/flutter_redux) 에서 확인할 수 있습니다.
* [Flutter를 위한 Codemagic CI/CD](https://blog.codemagic.io/getting-started-with-codemagic/)
* [Bitrise의 Flutter CI/CD](https://devcenter.bitrise.io/getting-started/getting-started-with-flutter-apps/)
