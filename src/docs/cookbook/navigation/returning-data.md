---
title: 이전 화면에 데이터 반환하기
prev:
  title: 명명된 경로로의 화면 전환
  path: /docs/cookbook/navigation/named-routes
next:
  title: 새로운 화면으로 데이터 전달하기
  path: /docs/cookbook/navigation/passing-data
---

새로운 화면으로부터 이전 화면으로 데이터를 반환해야하는 경우가 있습니다. 예를 들어,
사용자에게 두 가지 옵션을 보여주는 화면이 있다고 합시다. 사용자가 한 옵션을 선택했을 때
그것을 첫 번째 화면에 알려주면 그에 맞는 동작을 할 수 있을 것입니다!

어떻게 구현할 수있을 까요?
[`Navigator.pop`]({{site.api}}/flutter/widgets/Navigator/pop.html)을 사용하세요!

## 진행 단계

  1. 홈 화면을 정의합니다.
  2. 선택 창을 띄우는 버튼을 추가합니다.
  3. 두 개의 버튼을 가진 선택 창을 보여줍니다.
  4. 하나의 버튼을 클릭하면 선택 창을 닫습니다.
  5. 선택된 정보를 홈 화면의 snackbar에 보여줍니다.

## 1. 홈 화면을 정의합니다.

홈 화면에서는 버튼 하나를 보여줄 것입니다. 버튼을 클릭하면 선택 창을 띄울 것입니다!

<!-- skip -->
```dart
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Returning Data Demo'),
      ),
      // 다음 단계에서 SelectionButton 위젯을 만들 것입니다.
      body: Center(child: SelectionButton()),
    );
  }
}
```

## 2. 선택 창을 띄우는 버튼을 추가합니다.

Now, we'll create our SelectionButton. Our selection button will:
이제 SelectionButton을 만들 차례입니다. 선택 버튼은:

  1. 사용자가 클릭했을 때, SelectionScreen을 띄울 것입니다.
  2. SelectionScreen이 결과를 반환할 때까지 대기할 것입니다.

<!-- skip -->
```dart
class SelectionButton extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return RaisedButton(
      onPressed: () {
        _navigateAndDisplaySelection(context);
      },
      child: Text('Pick an option, any option!'),
    );
  }

  // SelectionScreen을 띄우고 navigator.pop으로부터 결과를 기다리는 메서드
  _navigateAndDisplaySelection(BuildContext context) async {
    // Navigator.push는 Future를 반환합니다. Future는 선택 창에서 
    // Navigator.pop이 호출된 이후 완료될 것입니다.
    final result = await Navigator.push(
      context,
      // 다음 단계에서 SelectionScreen를 만들 것입니다.
      MaterialPageRoute(builder: (context) => SelectionScreen()),
    );
  }
}
```

## 3. 두 개의 버튼을 가진 선택 창을 보여줍니다.

이제 선택 창을 만들 차례입니다. 선택 창은 두 개의 버튼을 갖고 있으며, 사용자가 하나의
버튼을 클릭하면 선택 창을 닫고 그 결과를 홈 화면에 알려줄 것입니다.

이번 단계에서는 UI를 정의하고, 다음 단계에서 데이터를 반환하는 방법에 대해 설명하겠습니다.

```dart
class SelectionScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Pick an option'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: RaisedButton(
                onPressed: () {
                  // "Yep" 문자열과 함께 이전 화면으로 돌아갑니다.
                },
                child: Text('Yep!'),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: RaisedButton(
                onPressed: () {
                  // "Nope" 문자열과 함께 이전 화면으로 돌아갑니다.
                },
                child: Text('Nope.'),
              ),
            )
          ],
        ),
      ),
    );
  }
}
```

## 4. 하나의 버튼을 클릭하면 선택 창을 닫습니다.

이제 앞서 만든 두 개 버튼의 `onPressed` 콜백을 작성할 차례입니다. 첫 번째 화면으로
데이터를 반환하기 위해, [`Navigator.pop`]({{site.api}}/flutter/widgets/Navigator/pop.html)
메서드를 사용할 것입니다.

`Navigator.pop`는 `result`라고 명시된 두 번째 인자를 선택적으로 받습니다. 만약
결과 값을 인자로 제공한다면, SelectionButton의 `Future`에 실려 반환될 것입니다.

### Yep 버튼

<!-- skip -->
```dart
RaisedButton(
  onPressed: () {
    // Yep 버튼은 결과로 "Yep!"을 반환합니다.
    Navigator.pop(context, 'Yep!');
  },
  child: Text('Yep!'),
);
```

### Nope 버튼

<!-- skip -->
```dart
RaisedButton(
  onPressed: () {
    // Nope 버튼은 결과로 "Nope!"을 반환합니다.
    Navigator.pop(context, 'Nope!');
  },
  child: Text('Nope!'),
);
```

## 5. 선택된 정보를 홈 화면의 snackbar에 보여줍니다.

선택 창을 띄우고 결과를 기다리고 있습니다. 이제 결과 값을 갖고 무언가 할 차례입니다.

이 예제에서는 결과 값을 보여줄 수 있도록 Snackbar를 띄우겠습니다. 이 작업을 하기 위해
`SelectionButton`의 `_navigateAndDisplaySelection` 메서드를 수정할 것입니다.

<!-- skip -->
```dart
_navigateAndDisplaySelection(BuildContext context) async {
  final result = await Navigator.push(
    context,
    MaterialPageRoute(builder: (context) => SelectionScreen()),
  );

  // 선택 창으로부터 결과 값을 받은 후, 이전에 있던 snackbar는 숨기고 새로운 결과 값을
  // 보여줍니다.
  Scaffold.of(context)
    ..removeCurrentSnackBar()
    ..showSnackBar(SnackBar(content: Text("$result")));
}
```

## 완성된 예제

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(
    title: 'Returning Data',
    home: HomeScreen(),
  ));
}

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Returning Data Demo'),
      ),
      body: Center(child: SelectionButton()),
    );
  }
}

class SelectionButton extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return RaisedButton(
      onPressed: () {
        _navigateAndDisplaySelection(context);
      },
      child: Text('Pick an option, any option!'),
    );
  }

  // SelectionScreen을 띄우고 navigator.pop으로부터 결과를 기다리는 메서드
  _navigateAndDisplaySelection(BuildContext context) async {
    // Navigator.push는 Future를 반환합니다. Future는 선택 창에서 
    // Navigator.pop이 호출된 이후 완료될 것입니다.
    final result = await Navigator.push(
      context,
      MaterialPageRoute(builder: (context) => SelectionScreen()),
    );

    // 선택 창으로부터 결과 값을 받은 후, 이전에 있던 snackbar는 숨기고 새로운 결과 값을
    // 보여줍니다.
    Scaffold.of(context)
      ..removeCurrentSnackBar()
      ..showSnackBar(SnackBar(content: Text("$result")));
  }
}

class SelectionScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Pick an option'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: RaisedButton(
                onPressed: () {
                  // 창을 닫고 결과로 "Yep!"을 반환합니다.
                  Navigator.pop(context, 'Yep!');
                },
                child: Text('Yep!'),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: RaisedButton(
                onPressed: () {
                  // 창을 닫고 결과로 "Nope!"을 반환합니다.
                  Navigator.pop(context, 'Nope.');
                },
                child: Text('Nope.'),
              ),
            )
          ],
        ),
      ),
    );
  }
}
```

![Returning Data Demo](/images/cookbook/returning-data.gif){:.site-mobile-screenshot}
