---
title: 명명된 경로로의 화면 전환
prev:
  title: 새로운 화면으로 이동하고, 되돌아오기
  path: /docs/cookbook/navigation/navigation-basics
next:
  title: Return data from a screen
  path: /docs/cookbook/navigation/returning-data
---

[새로운 화면으로 이동하고, 되돌아오기](/docs/cookbook/navigation/navigation-basics/)
예제에서는 새로 생성한 경로를 
[`Navigator`]({{site.api}}/flutter/widgets/Navigator-class.html)
에 전달하여 새로운 화면으로 전환하는 방법을 배웠습니다.

하지만 만약 앱의 다른 많은 부분들에서 동일한 화면으로 이동하고자 한다면, 중복된 코드가
생기게 되는데, 이러한 경우 "명명된 경로"를 정의하여 화면 전환에 사용하는 방법이 해결책이
될 수 있습니다.

명명된 경로를 사용하기 위해 [`Navigator.pushNamed`]({{site.api}}/flutter/widgets/Navigator/pushNamed.html)
함수를 사용할 수 있습니다. 이 예제에서는 명명된 경로를 사용하는 방법을 보여주기 위해
기존 예제의 기능을 사용할 것입니다.

## 진행 단계

  1. 두 개의 화면 만들기
  2. 경로 정의하기
  3. `Navigator.pushNamed`를 사용하여 두 번째 화면으로 전환하기
  4. `Navigator.pop`을 사용하여 첫 번째 화면으로 돌아가기

## 1. 두 개의 화면 만들기

우선, 본 예제를 시작하기 위해 두 개의 화면이 필요합니다. 첫 번째 화면에는 두 번째 화면으로 
이동하기 위한 버튼 하나가 있고, 두 번째 화면에는 다시 첫 번째 화면으로 돌아가기 위한 버튼이
있습니다.

```dart
class FirstScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('First Screen'),
      ),
      body: Center(
        child: RaisedButton(
          child: Text('Launch screen'),
          onPressed: () {
            // 클릭하면 두 번째 화면으로 전환합니다!
          },
        ),
      ),
    );
  }
}

class SecondScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Second Screen"),
      ),
      body: Center(
        child: RaisedButton(
          onPressed: () {
            // 클릭하면 첫 번째 화면으로 돌아갑니다!
          },
          child: Text('Go back!'),
        ),
      ),
    );
  }
}
```

## 2. 경로 정의하기

다음으로, [`MaterialApp`]({{site.api}}/flutter/material/MaterialApp-class.html) 생성자에 
`initialRoute`와 `routes` 이름의 추가 프로퍼티를 제공하여 경로를 정의하겠습니다. 

`initialRoute` 프로퍼티는 앱의 시작점을 나타내는 경로를 정의하고, `routes` 프로퍼티는 이용가능한 
명명된 경로와 해당 경로로 이동했을 때 빌드될 위젯을 정의합니다.

<!-- skip -->
```dart
MaterialApp(
  // "/"으로 명명된 경로와 함께 시작합니다. 본 예제에서는 FirstScreen 위젯에서 시작합니다.
  initialRoute: '/',
  routes: {
    // "/" 경로로 이동하면, FirstScreen 위젯을 생성합니다.
    '/': (context) => FirstScreen(),
    // "/second" 경로로 이동하면, SecondScreen 위젯을 생성합니다.
    '/second': (context) => SecondScreen(),
  },
);
```

참고: `initialRoute`를 사용한다면, `home`프로퍼티를 정의하면 안됩니다.

## 3. 두 번째 화면으로 전환하기

위젯과 경로를 정의했다면, 화면 전환을 시작할 수 있습니다! 본 예제에서는, 
[`Navigator.pushNamed`]({{site.api}}/flutter/widgets/Navigator/pushNamed.html)
함수를 사용할 것입니다. 이 함수는 Flutter에게 앞서 `routes` 테이블에 정의한 위젯을 생성하고 
그 화면을 시작하도록 요청합니다.


`FirstScreen`위젯의 `build` 메서드에 `onPressed` 콜백을 다음과 같이 업데이트하세요:

<!-- skip -->
```dart
// `FirstScreen` 위젯의 콜백
onPressed: () {
  // 명명된 경로를 사용하여 두 번째 화면으로 전환합니다.
  Navigator.pushNamed(context, '/second');
}
```

## 4. 첫 번째 화면으로 돌아가기

첫 번째 페이지로 되돌아가기 위해
[`Navigator.pop`]({{site.api}}/flutter/widgets/Navigator/pop.html)
함수를 사용합니다.

<!-- skip -->
```dart
// SecondScreen 위젯의 콜백
onPressed: () {
  // 현재 경로를 스택에서 제거함으로써 첫 번째 스크린으로 되돌아 갑니다.
  Navigator.pop(context);
}
```

## 완성된 예제

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(
    title: 'Named Routes Demo',
    // "/"을 앱이 시작하게 될 경로로 지정합니다. 본 예제에서는 FirstScreen 위젯이 첫 번째 페이지가
    // 될 것입니다.
    initialRoute: '/',
    routes: {
      // When we navigate to the "/" route, build the FirstScreen Widget
      // "/" 경로로 이동하면, FirstScreen 위젯을 생성합니다.
      '/': (context) => FirstScreen(),
      // "/second" 경로로 이동하면, SecondScreen 위젯을 생성합니다.
      '/second': (context) => SecondScreen(),
    },
  ));
}

class FirstScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('First Screen'),
      ),
      body: Center(
        child: RaisedButton(
          child: Text('Launch screen'),
          onPressed: () {
            // 명명된 경로를 사용하여 두 번째 화면으로 전환합니다.
            Navigator.pushNamed(context, '/second');
          },
        ),
      ),
    );
  }
}

class SecondScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Second Screen"),
      ),
      body: Center(
        child: RaisedButton(
          onPressed: () {
            // 현재 경로를 스택에서 제거함으로써 첫 번째 스크린으로 되돌아 갑니다.
            Navigator.pop(context);
          },
          child: Text('Go back!'),
        ),
      ),
    );
  }
}
```

![Navigation Basics Demo](/images/cookbook/navigation-basics.gif){:.site-mobile-screenshot}
