---
title: 화면을 넘나드는 위젯 애니메이션
prev:
  title: 서비스에 에러 보고 하기
  path: /docs/cookbook/maintenance/error-reporting
next:
  title: 새로운 화면으로 이동하고, 되돌아오기
  path: /docs/cookbook/navigation/navigation-basics
---

한 화면에서 다른 화면으로 넘어갈 때, 사용자에게 어떠한 가이드를 주는 것은 종종 도움이 됩니다.
앱에서 사용할 수 있는 일반적인 기술은 다음 화면으로 전환할 때 위젯에 애니메이션 효과를 주는 것입니다.
이러한 방법은 두 화면을 이어주는 시각적 연결 고리를 만들어 줍니다.

Flutter에서는 한 화면에서 다음 화면으로 전환할 때 위젯에 어떻게 애니메이션 효과를 줄 수 있을 까요?
[`Hero`]({{site.api}}/flutter/widgets/Hero-class.html) 위젯이 그 비법입니다!

## 진행 단계

  1. 같은 이미지를 보여주는 2개의 화면을 만듭니다.
  2. 첫 번째 화면에 `Hero` 위젯을 추가합니다.
  3. 두 번째 화면에 `Hero` 위젯을 추가합니다.

## 1. 같은 이미지를 보여주는 2개의 화면을 만듭니다.

In this example, we'll display the same image on both screens. We'll want to
animate the image from the first screen to the second screen when the user taps
on the image. For now, we'll create the visual structure, and handle animations
in the next steps!

*참고:* This example builds upon the
[Navigate to a new screen and back](/docs/cookbook/navigation/navigation-basics)
and [Handling Taps](/docs/cookbook/gestures/handling-taps) recipes.

```dart
class MainScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Main Screen'),
      ),
      body: GestureDetector(
        onTap: () {
          Navigator.push(context, MaterialPageRoute(builder: (_) {
            return DetailScreen();
          }));
        },
        child: Image.network(
          'https://picsum.photos/250?image=9',
        ),
      ),
    );
  }
}

class DetailScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: GestureDetector(
        onTap: () {
          Navigator.pop(context);
        },
        child: Center(
          child: Image.network(
            'https://picsum.photos/250?image=9',
          ),
        ),
      ),
    );
  }
}
```

## 2. Add a `Hero` Widget to the first screen

In order to connect the two screens together with an animation, we need to wrap
the `Image` Widget on both screens in a `Hero` Widget. The `Hero` Widget
requires two arguments:

  1. `tag`: An object that identifies the `Hero`. It must be the same on both
  screens.
  2. `child`: The Widget we want to animate across screens.

<!-- skip -->
```dart
Hero(
  tag: 'imageHero',
  child: Image.network(
    'https://picsum.photos/250?image=9',
  ),
);
```

## 3. Add a `Hero` Widget to the second screen

To complete the connection with the first screen, we need to wrap the `Image`
on the second screen with a `Hero` Widget as well! It must use the same `tag`
as the first screen.

After you apply the `Hero` Widget to the second screen, the animation between
screens will work!

<!-- skip -->
```dart
Hero(
  tag: 'imageHero',
  child: Image.network(
    'https://picsum.photos/250?image=9',
  ),
);
```

참고: this code is identical to what we had on the first screen! In general, you
could create a reusable Widget instead of repeating code, but for this example,
we'll duplicate the code for demonstration purposes.

## Complete example

```dart
import 'package:flutter/material.dart';

void main() => runApp(HeroApp());

class HeroApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Transition Demo',
      home: MainScreen(),
    );
  }
}

class MainScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Main Screen'),
      ),
      body: GestureDetector(
        child: Hero(
          tag: 'imageHero',
          child: Image.network(
            'https://picsum.photos/250?image=9',
          ),
        ),
        onTap: () {
          Navigator.push(context, MaterialPageRoute(builder: (_) {
            return DetailScreen();
          }));
        },
      ),
    );
  }
}

class DetailScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: GestureDetector(
        child: Center(
          child: Hero(
            tag: 'imageHero',
            child: Image.network(
              'https://picsum.photos/250?image=9',
            ),
          ),
        ),
        onTap: () {
          Navigator.pop(context);
        },
      ),
    );
  }
}
```

![Hero Demo](/images/cookbook/hero.gif){:.site-mobile-screenshot}
