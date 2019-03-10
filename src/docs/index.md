---
title: Flutter 문서
short-title: 문서
description: The landing page for Flutter documentation.
---

<div class="card-deck">
{% for card in site.data.docs_cards -%}
  <a class="card" href="{{card.url}}">
    <div class="card-body">
      <header class="card-title">{{card.name}}</header>
      <p class="card-text">{{card.description}}</p>
    </div>
  </a>
{% endfor -%}
</div>

## 사이트 개편 내역

**2019년 2월 26일**

바르셀로나에서 열리는 Mobile World Congress (MWC)에서 Flutter [1.2 버전](https://developers.googleblog.com/2019/02/launching-flutter-12-at-mobile-world.html) 출시!

추가적으로, 최근 업데이트된 몇몇 새로운 문서도 있습니다:

* [상태 관리 가이드](/docs/development/data-and-backend/state-mgmt/intro)가 업데이트 됐습니다.
  새로운 페이지는 
  [개요](/docs/development/data-and-backend/state-mgmt/intro),
  [선언적으로 생각하기](/docs/development/data-and-backend/state-mgmt/declarative), 
  [일시적인 상태 vs 앱 상태](/docs/development/data-and-backend/state-mgmt/ephemeral-vs-app),
  [간단한 앱 상태 관리](/docs/development/data-and-backend/state-mgmt/simple),
  [다른 상태 관리 방안](/docs/development/data-and-backend/state-mgmt/options)을 포함하고 있습니다.
  모든 경우에 옳은 일률적인 접근 방식이 없기 때문에, 상태 관리에 관한 문서화는 까다로운 일입니다. 
  새로운 문서에 대한 여러분의 의견을 환영합니다!
* [성능 모범 사례](/docs/testing/best-practices)의 새로운 페이지.
* 또한 MWC에서는 다트와 Flutter 앱 프로파일링 및 디버깅을 위한 새로운 다트 개발도구의 프리뷰 버전을 발표했습니다. 
  [개발도구 wiki](https://flutter.github.io/devtools/)에서 확인하실 수 있습니다.
  특히, 사용자 UI 디버깅을 위한
  [Flutter 위젯 검사기](https://flutter.github.io/devtools/inspector) 또는 
  Flutter 앱의 프로파일링을 위한 [타임라인 보기](https://flutter.github.io/devtools/timeline)
  기능을 확인해보세요.
  써보시고 어떻게 생각하는지 알려주세요!
* 새로운 다트 개발도구 UI와 통합하여 [성능 프로파일링](/docs/testing/ui-performance)
  페이지를 업데이트 했습니다.
* 새로운 다트 개발도구 UI에 관한 정보를 추가해 [안드로이드 스튜디오/IntelliJ](/docs/development/tools/android-studio)와
  [VS Code](/docs/development/tools/vs-code) 페이지를 
  업데이트 했습니다.

이 문서에 대해 질문이나 의견이 있으시면, 
[이슈를 등록해주세요]({{site.repo.this}}/issues).

한글 사이트에 대한 [이슈도 등록해주세요](https://github.com/bro-jo/website/issues).

[개편 아카이브](/docs/whats-new-archive)

## Flutter가 처음인가요?

[시작하기](/docs/get-started/install)와
 [첫 번째 앱 작성하기](/docs/get-started/codelab)를 완료했다면, 
다음 단계가 준비되어 있습니다.

[안드로이드 개발자를 위한 Flutter](/docs/get-started/flutter-for/android-devs)
: 안드로이드 개발 경험이 있다면 이 팁을 봐보세요.

[iOS 개발자를 위한 Flutter](/docs/get-started/flutter-for/ios-devs)
: iOS 개발 경험이 있다면 이 팁을 봐보세요.

[웹 개발자를 위한 Flutter](/docs/get-started/flutter-for/web-devs)
: 웹 경험이 있다면 HTML과 Flutter의 유사한 점을 살펴보세요.

[리액트 네이티브 개발자를 위한 Flutter](/docs/get-started/flutter-for/react-native-devs)
: 리액트 네이티브 개발 경험이 있다면 이 팁을 봐보세요.

[Xamarin.Forms 개발자를 위한 Flutter](/docs/get-started/flutter-for/xamarin-forms-devs)
: Xamarin.Forms 개발 경험이 있다면 이 팁을 봐보세요.

[Flutter에서 레이아웃](/docs/development/ui/layout)
: 레이아웃을 만드는 방법을 배워보세요. Flutter에서는 모든 게 위젯입니다.

[Flutter 앱에 상호작용 추가하기](/docs/development/ui/interactive)
: 상태를 저장할 수 있는 위젯을 만드는 방법을 배워보세요.

[Flutter 위젯 프레임워크 둘러보기](/docs/development/ui/widgets-intro)
: react-style 프레임워크 Flutter에 대해 더 알아봅시다.

[자주 묻는 질문](/docs/resources/faq)
: 자주 묻는 질문에 대한 답변을 살펴보세요.


## 한 단계 더 나아가고 싶나요?

기본적인 내용을 숙달했으면 이제 다음 페이지를 봐보세요.

[Cookbook](/docs/cookbook)
: 일반적인 Flutter 사용 사례를 다루는 레시피 모음. (개선 중)

[깃허브에 있는 예제 앱]({{site.github}}/flutter/samples/blob/master/INDEX.md)
: Flutter를 잘 활용한 샘플 앱 모음. (개선 중)

[Flutter에서 리소스와 이미지 파일 추가하기](/docs/development/ui/assets-and-images)
: Flutter 앱에 리소스를 추가하는 방법.

[Flutter에서 애니메이션](/docs/development/ui/animations)
: 기본 애니메이션과, hero 애니메이션, staggered 애니메이션 등 Flutter가 지원하는 애니메이션 생성 방법.

[내비게이션 및 라우팅](/docs/development/ui/navigation)
: 새로운 스크린을 만들고 새로운 스크린으로 이동하는 방법 (Flutter에서는 _route_ 라고 부릅니다).

[국제화](/docs/development/accessibility-and-localization/internationalization)
: 글로벌화하세요! Flutter 앱을 국제화하는 방법.

[Effective Dart]({{site.dart-site}}/guides/language/effective-dart)
: 더 나은 다트 코드를 작성법 가이드.

## 전문적인 주제

관심있는 주제를 더 깊게 살펴보세요.

[위젯 검사 도구](/docs/development/tools/inspector)
: 위젯 트리를 탐색하거나, DEBUG 배너를 없애거나, 성능 오버레이 표시해주는 등 다양한 기능을 가능하게 해주는 강력한 도구인 위젯 검사 도구를 사용하는 방법.

[사용자 지정 글꼴](/docs/cookbook/design/fonts)
: 새로운 폰트를 앱에 추가하는 방법.

[문자 입력](/docs/cookbook/forms/text-input)
: 문자를 입력받는 기본적인 방법.

[Flutter 앱 디버깅](/docs/testing/debugging)
: 앱 디버깅을 위한 도구와 팁

완성된 목록은 아닙니다.
왼쪽 탐색 메뉴를 사용해주세요. 다른 주제를 찾고 싶으시면 검색창을 이용해주세요. 
