---
title: 새로운 화면으로 이동하고, 되돌아오기
description: 화면간 이동하는 방법
prev:
  title: 화면을 넘나드는 위젯 애니메이션
  path: /docs/cookbook/navigation/hero-animations
next:
  title: Named-Route로의 화면 전환
  path: /docs/cookbook/navigation/named-routes
---

대부분의 앱은 여러 종류의 정보를 보여주기 위해 여러 화면을 갖고 있습니다.
예를 들어, 어떤 앱이 상품 목록을 보여주는 화면을 갖고 있다고 한다면, 사용자는 그 중 
한 상품을 선택하여 새로운 화면에서 해당 상품의 상세 정보를 볼 수 있습니다.

{{site.alert.info}}
  **용어**: Flutter에서 _screen_ 과 _page_ 는 _route_ 로 불립니다.
  이 문서의 나머지 부분에서는 Route라고 언급합니다.
{{site.alert.end}}

Route는 Android의 Activity, iOS의 ViewController와 동일합니다.
Flutter에서는 Route 역시 위젯입니다.

새로운 route로 어떻게 이동할까요? 
[`Navigator`]({{site.api}}/flutter/widgets/Navigator-class.html)를 사용하세요.

## 진행 단계

The next few sections show how to navigate between two routes,
다음 섹션에서 두 개의 route에서 화면 전환하는 방법을 보여주겠습니다.
아래 단계를 따라오세요:

  1. 두 개의 route를 생성합니다.
  2. Navigator.push()를 사용하여 두 번째 route로 전환합니다.
  3. Navigator.pop()을 사용하여 첫 번째 route로 되돌아 옵니다.

## 1. 두 개의 route를 생성합니다.

우선 두 개의 route를 생성합니다. 간단한 예제이기 때문에 각 route에는 버튼
하나 씩만 넣어줄 겁니다. 첫 번째 route의 버튼을 누르면 두 번째 route로 
화면 전환되며, 두 번째 route의 버튼을 누르면 첫 번째 route로 되돌아 옵니다.

우선 시각적 구조를 세팅합니다.

```dart
class FirstRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('First Route'),
      ),
      body: Center(
        child: RaisedButton(
          child: Text('Open route'),
          onPressed: () {
            // 눌렀을 때 두 번째 route로 이동합니다.
          },
        ),
      ),
    );
  }
}

class SecondRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Second Route"),
      ),
      body: Center(
        child: RaisedButton(
          onPressed: () {
            // 눌렀을 때 첫 번째 route로 되돌아 갑니다.
          },
          child: Text('Go back!'),
        ),
      ),
    );
  }
}
```

## 2. Navigator.push()를 사용하여 두 번째 route로 전환합니다.

새로운 route로 전환하기 위해 
[`Navigator.push()`]({{site.api}}/flutter/widgets/Navigator/push.html)
메서드를 사용하세요. `push()` 메서드는 `Route`를 Navigator에 의해 관리되는 
route 스택에 추가합니다. `Route`는 어디서 오는 걸까요?
직접 생성하거나 [`MaterialPageRoute`]({{site.api}}/flutter/material/MaterialPageRoute-class.html)의 
것을 사용할 수 있습니다. `MaterialPageRoute`는 새로운 route로 이동 시 
플랫폼 특화된 애니메이션을 사용하기 때문에 편리합니다.

`FirstRoute` 위젯의 `build()` 메서드에서 `onPressed()` 콜백을 수정하세요:

<!-- skip -->
```dart
// Within the `FirstRoute` widget
onPressed: () {
  Navigator.push(
    context,
    MaterialPageRoute(builder: (context) => SecondRoute()),
  );
}
```

## 3. Navigator.pop()을 사용하여 첫 번째 route로 되돌아 옵니다.

두 번째 route를 닫고 이전 route로 어떻게 되돌아 갈까요?
[`Navigator.pop()`]({{site.api}}/flutter/widgets/Navigator/pop.html)
메서드를 사용하세요. `pop()` 메서드는 Navigator에 의해 관리되는 route 스택에서 
현재 `Route`를 제거합니다.

이전 route로 되돌아 가기 위해, `SecondRoute` 위젯의 `onPressed()` 콜백을 수정하세요:

<!-- skip -->
```dart
// Within the SecondRoute widget
onPressed: () {
  Navigator.pop(context);
}
```

## 완성된 예제

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(
    title: 'Navigation Basics',
    home: FirstRoute(),
  ));
}

class FirstRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('First Route'),
      ),
      body: Center(
        child: RaisedButton(
          child: Text('Open route'),
          onPressed: () {
            Navigator.push(
              context,
              MaterialPageRoute(builder: (context) => SecondRoute()),
            );
          },
        ),
      ),
    );
  }
}

class SecondRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Second Route"),
      ),
      body: Center(
        child: RaisedButton(
          onPressed: () {
            Navigator.pop(context);
          },
          child: Text('Go back!'),
        ),
      ),
    );
  }
}
```

{% comment %}
We need a new GIF that shows "Route" instead of "Screen".
{% endcomment %}

![Navigation Basics Demo](/images/cookbook/navigation-basics.gif){:.site-mobile-screenshot}
