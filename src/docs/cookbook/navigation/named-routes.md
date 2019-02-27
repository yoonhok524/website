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

## Directions

  1. Create two screens
  2. Define the routes
  3. Navigate to the second screen using `Navigator.pushNamed`
  4. Return to the first screen using `Navigator.pop`

## 1. Create two screens

First, we'll need two screens to work with. The first screen will contain a
button that navigates to the second screen. The second screen will contain a
button that navigates back to the first.

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
            // Navigate to second screen when tapped!
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
            // Navigate back to first screen when tapped!
          },
          child: Text('Go back!'),
        ),
      ),
    );
  }
}
```

## 2. Define the routes

Next, we'll need to define our routes by providing additional properties to the
[`MaterialApp`]({{site.api}}/flutter/material/MaterialApp-class.html)
constructor: the `initialRoute` and the `routes` themselves.

The `initialRoute` property defines which route our app should start with. The
`routes` property defines the available named routes and the Widgets that should
be built when we navigate to those routes.

<!-- skip -->
```dart
MaterialApp(
  // Start the app with the "/" named route. In our case, the app will start
  // on the FirstScreen Widget
  initialRoute: '/',
  routes: {
    // When we navigate to the "/" route, build the FirstScreen Widget
    '/': (context) => FirstScreen(),
    // When we navigate to the "/second" route, build the SecondScreen Widget
    '/second': (context) => SecondScreen(),
  },
);
```

참고: When using `initialRoute`, be sure you do not define a `home` property.

## 3. Navigate to the second screen

With our Widgets and routes in place, we can begin navigating! In this case,
we'll use the
[`Navigator.pushNamed`]({{site.api}}/flutter/widgets/Navigator/pushNamed.html)
function. This tells Flutter to build the Widget defined in our `routes` table
and launch the screen.

In the `build` method of our `FirstScreen` Widget, we'll update the `onPressed`
callback:

<!-- skip -->
```dart
// Within the `FirstScreen` Widget
onPressed: () {
  // Navigate to the second screen using a named route
  Navigator.pushNamed(context, '/second');
}
```

## 4. Return to the first screen

In order to navigate back to the first page, we can use the
[`Navigator.pop`]({{site.api}}/flutter/widgets/Navigator/pop.html)
function.

<!-- skip -->
```dart
// Within the SecondScreen Widget
onPressed: () {
  // Navigate back to the first screen by popping the current route
  // off the stack
  Navigator.pop(context);
}
```

## Complete example

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(
    title: 'Named Routes Demo',
    // Start the app with the "/" named route. In our case, the app will start
    // on the FirstScreen Widget
    initialRoute: '/',
    routes: {
      // When we navigate to the "/" route, build the FirstScreen Widget
      '/': (context) => FirstScreen(),
      // When we navigate to the "/second" route, build the SecondScreen Widget
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
            // Navigate to the second screen using a named route
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
            // Navigate back to the first screen by popping the current route
            // off the stack
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
