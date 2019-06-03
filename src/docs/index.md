---
title: Flutter 문서
short-title: 문서
description: Flutter 문서의 랜딩 페이지
---

{% for card in site.data.docs_cards -%}
  {% capture index0Modulo3 %}{{ forloop.index0 | modulo:3 }}{% endcapture %}
  {% capture indexModulo3 %}{{ forloop.index | modulo:3 }}{% endcapture %}
  {% if index0Modulo3 == '0' %}
  <div class="card-deck mb-4">
  {% endif %}
    <a class="card" href="{{card.url}}">
      <div class="card-body">
        <header class="card-title">{{card.name}}</header>
        <p class="card-text">{{card.description}}</p>
      </div>
    </a>
  {% if indexModulo3 == '0' %}
  </div>
  {% endif %}
{% endfor -%}

## 사이트 개편 내역

**2019년 5월 7일, 구글 I/O 에디션**

[Flutter 1.5](https://developers.googleblog.com/2019/05/Flutter-io19.html)가 출시됐습니다!

업데이트에 관한 자세한 정보는 
[릴리스 노트](https://github.com/flutter/flutter/wiki/Release-Notes-Flutter-1.5.4)
또는 [릴리스 다운로드](/docs/development/tools/sdk/archive)를 참조하세요.

[What's new archive](/docs/whats-new-archive)

## Flutter가 처음인가요?

[시작하기](/docs/get-started/install)와
 [첫 번째 앱 작성하기](/docs/get-started/codelab)를 완료했다면, 
다음 단계가 준비되어 있습니다.

### 문서

다른 플랫폼 경험이 있으신가요? 아래 문서를 확인해보세요:
[Android](/docs/get-started/flutter-for/android-devs),
[iOS](/docs/get-started/flutter-for/ios-devs),
[웹](/docs/get-started/flutter-for/web-devs),
[React Native](/docs/get-started/flutter-for/react-native-devs),
[Xamarin.Forms](/docs/get-started/flutter-for/xamarin-forms-devs)

[Flutter에서 레이아웃](/docs/development/ui/layout)
: 레이아웃을 만드는 방법을 배워보세요. Flutter에서는 모든 게 위젯입니다.

[Flutter 앱에 상호작용 추가하기](/docs/development/ui/interactive)
: 상태를 저장할 수 있는 위젯을 만드는 방법을 배워보세요.

[Flutter 위젯 프레임워크 둘러보기](/docs/development/ui/widgets-intro)
: react-style 프레임워크 Flutter에 대해 더 알아봅시다.

[자주 묻는 질문](/docs/resources/faq)
: 자주 묻는 질문에 대한 답변을 살펴보세요.

### 영상

도움이 될만한 영상을 [Flutter 유튜브 채널]({{site.social.youtube}})을 통해 제공합니다!
특히 Flutter in Focus 시리즈를 보고, 
비디오 페이지에서 다른 시리즈를 통해 배워보세요.

<iframe style="max-width: 100%" width="560" height="315" src="https://www.youtube.com/embed/wgTBLj7rMPM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
Flutter in Focus: Learn Flutter features in 10 minutes or less.<br>
[Flutter in Focus 재생 목록](https://www.youtube.com/playlist?list=PLjxrf2q8roU2HdJQDjJzOeO6J3FoFLWr2)

Flutter에서는 "모든 것이 위젯"입니다!
Stateless과 Stateful라는 2가지 종류의 위젯을 보다 잘 이해하기 위해,
[Flutter in Focus](https://www.youtube.com/playlist?list=PLjxrf2q8roU2HdJQDjJzOeO6J3FoFLWr2) 시리즈
를 확인해보세요.

<iframe style="max-width: 100%" width="560" height="315" src="https://www.youtube.com/embed/wE7khGHVkYY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> <iframe width="560" height="315" src="https://www.youtube.com/embed/AqCMFXEmf3w" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## 한 단계 더 나아가고 싶나요?

엔지니어가 코드를 작성하고, 실수하고, 고치는 모습을 보면서 배우길 원한다면
[Boring Flutter Show](https://www.youtube.com/watch?v=vqPG1tU6-c0&list=PLjxrf2q8roU28W3pXbISJbVA5REsA41Sx&index=3&t=9s)
영상 시리즈를 확인하세요:

<iframe style="max-width: 100%" width="560" height="315" src="https://www.youtube.com/embed/vqPG1tU6-c0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
[Boring Flutter Show 재생 목록](https://www.youtube.com/watch?v=vqPG1tU6-c0&list=PLjxrf2q8roU28W3pXbISJbVA5REsA41Sx&index=3&t=9s)

아래 문서들도 유용할 것입니다:

* [패키지 사용하기](/docs/development/packages-and-plugins/using-packages)
* [리소스 및 이미지 추가하기](/docs/development/ui/assets-and-images)
* [내비게이션 및 라우팅](/docs/development/ui/navigation)
* [상태 관리](/docs/development/data-and-backend/state-mgmt/intro)
* [애니메이션](/docs/development/ui/animations)

