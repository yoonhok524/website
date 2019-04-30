---
title: 인터넷에서 데이터 가져오기
prev:
  title: 새로운 화면으로 데이터 보내기
  path: /docs/cookbook/navigation/passing-data
next:
  title: 인증된 요청 생성하기
  path: /docs/cookbook/networking/authenticated-requests
---

인터넷을 통해 데이터를 가져오는 것은 대부분의 앱에서 필수적입니다. 다행스럽게도, Dart와 Flutter는
이러한 유형의 작업을 위해 도구를 제공합니다.

## 진행 단계

  1. `http` 패키지 추가하기
  2. `http` 패키지를 사용하여 네트워크 요청 생성하기
  3. 응답 정보를 커스텀 Dart 객체로 변환하기
  4. 가져온 데이터를 Flutter에 보여주기

## 1. `http` 패키지 추가하기

[`http`]({{site.pub-pkg}}/http) 패키지를 사용하면 인터넷으로부터
데이터를 손쉽게 가져올 수 있습니다.

`http`패키지를 설치하기 위해서, `pubspec.yaml`의 의존성 부분에 추가해줘야 합니다.
[최신 버전의 http 패키지는 Pub 사이트]({{site.pub}}/packages/http#-installing-tab-)
에서 확인할 수 있습니다.

```yaml
dependencies:
  http: <latest_version>
```

## 2. `http` 패키지를 사용하여 네트워크 요청 생성하기

본 예제에서는, [http.get()]({{site.pub-api}}/http/latest/http/get.html) 메서드를
사용하여 [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/)
으로부터 샘플 Post를 가져올 것입니다.

<!-- skip -->
```dart
Future<http.Response> fetchPost() {
  return http.get('https://jsonplaceholder.typicode.com/posts/1');
}
```

`http.get()` 메서드는 `Response`를 포함하고 있는 `Future`를 반환합니다.

  * [`Future`]({{site.api}}/flutter/dart-async/Future-class.html)는
    비동기 연산에 사용되는 Dart의 핵심 클래스입니다.
    미래의 특정 시점에 사용가능한 잠재적인 값이나 에러를 나타내기 위해
    사용됩니다.
  * `http.Response` 클래스는 http 요청이 성공했을 때 응답으로 받은 데이터를 
    갖고 있습니다.

## 3. 응답 정보를 커스텀 Dart 객체로 변환하기

네트워크 요청 생성은 쉽지만 `Future<http.Response>`을 사용하는 것은 편리하지 
않습니다. 이를 좀 더 쉽게 하기 위해 `http.Response`를 Dart 객체로 변환하겠습니다.

### `Post` 클래스를 생성하세요

우선 네트워크 요청으로부터 받은 데이터를 지니고 있을 `Post` 클래스를 
생성하세요. json으로부터 `Post`를 생성하는 factory 생성자를 포함할 
것입니다.

JSON을 직접 변환하는 것도 가능합니다. 더 자세한 정보를 원한다면,
[JSON과 직렬화](/docs/development/data-and-backend/json) 
문서를 참고하세요.

<!-- skip -->
```dart
class Post {
  final int userId;
  final int id;
  final String title;
  final String body;

  Post({this.userId, this.id, this.title, this.body});

  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      userId: json['userId'],
      id: json['id'],
      title: json['title'],
      body: json['body'],
    );
  }
}
```

### `http.Response`를 `Post`로 변환하세요

이제 `fetchPost` 함수가 `Future<Post>`를 반환하도록 수정하겠습니다. 이 작업을 위해
다음의 것들이 필요합니다:

  1. `dart:convert` 패키지를 통해 응답 내용을 json `Map`으로 변환하세요.
  2. 만약 서버가 상태 코드 200과 함께 "OK" 응답을 반환하면, factory 메서드 
     `fromJson`를 사용하여 json `Map`을 `Post`로 변환하세요.
  3. 만약 서버가 예상치 못한 응답을 반환하면, 에러를 던지세요.

<!-- skip -->
```dart
Future<Post> fetchPost() async {
  final response =
      await http.get('https://jsonplaceholder.typicode.com/posts/1');

  if (response.statusCode == 200) {
    // 만약 서버가 OK 응답을 반환하면, JSON을 파싱합니다.
    return Post.fromJson(json.decode(response.body));
  } else {
    // 만약 응답이 OK가 아니면, 에러를 던집니다.
    throw Exception('Failed to load post');
  }
}
```

와우! 이제 인터넷에서 Post를 가져올 수 있는 함수가 생겼습니다.

## 4. 가져온 데이터를 Flutter에 보여주기

데이터를 가져오고 화면에 보여주기 위한 목적으로, [`FutureBuilder`]({{site.api}}/flutter/widgets/FutureBuilder-class.html)
위젯을 사용할 수 있습니다. `FutureBuilder` 위젯은 Flutter에 기본적으로 
제공되는 위젯으로 비동기 데이터 처리를 쉽게 해줍니다.

두 가지 파라미터를 제공해야 합니다:

  1. `Future` 객체. 여기서는 `fetchPost()` 함수를 그대로 넘겨줍니다.
  2. `Futer`의 상태(로딩, 성공 혹은 에러)에 따라 Flutter에게 무엇을 랜더링해야 할지 
     알려줄 `builder` 함수

<!-- skip -->
```dart
FutureBuilder<Post>(
  future: fetchPost(),
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      return Text(snapshot.data.title);
    } else if (snapshot.hasError) {
      return Text("${snapshot.error}");
    }

    // 기본적으로 로딩 Spinner를 보여줍니다.
    return CircularProgressIndicator();
  },
);
```

## 5. 네트워크 요청 호출을 `build()` 바깥으로 옮기세요

편리하겠지만, API 요청 코드를 `build()` 메서드 안에 위치시키는 것은 바람직하지
않습니다.

Flutter는 무언가 변경될 때마다 `build()` 메서드를 호출하는데, 이 호출은 놀랄 
만큼 자주 일어납니다. 만약 네트워크 요청 코드를 `build()` 메서드에 그대로 남겨둔다면,
불필요한 API 요청이 아주 많이 발생하고 앱이 느려질 수 있습니다.

페이지가 최초 로드될 때 한번 API를 호출하게 하는 좀 더 나은 방법이 있습니다.

### `StatelessWidget`에 넘기세요

이 방법을 사용하면, 부모 위젯은 fetch 메서드를 호출하고 그 결과를 저장하고 위젯에 
전달하게 됩니다.

<!-- skip -->
```dart
class MyApp extends StatelessWidget {
  final Future<Post> post;

  MyApp({Key key, this.post}) : super(key: key);
```

아래 완성된 예제에서 이에 대한 작동 예제를 확인할 수 있습니다.

### `StatefulWidget`의 state의 생명주기 안에서 호출하세요

만약 위젯이 stateful이라면, fetch 메서드를 [`initState`]({{site.api}}/flutter/widgets/State/initState.html) 혹은
[`didChangeDependencies`]({{site.api}}/flutter/widgets/State/didChangeDependencies.html) 
메서드에서 호출할 수 있습니다.

`initState`는 정확히 한번만 호출됩니다.
만약 
[`InheritedWidget`]({{site.api}}/flutter/widgets/InheritedWidget-class.html) 변경 시
API를 다시 호출해야 하는 상황이라면 `didChangeDependencies` 메서드 안에서 호출하세요.
더 자세한 내용을 위해 
[`State`]({{site.api}}/flutter/widgets/State-class.html) 문서를 참고하세요.

<!-- skip -->
```dart
class _MyAppState extends State<MyApp> {
  Future<Post> post;

  @override
  void initState() {
    super.initState();
    post = fetchPost();
  }
```

## 테스팅

이 기능을 테스트하는 방법에 대해 더 자세히 알고 싶다면,
아래 문서를 확인하세요:

  * [단위 테스트 소개](/docs/cookbook/testing/unit/introduction)
  * [Mockito를 사용한 Mock 의존성](/docs/cookbook/testing/unit/mocking)

## 완성된 예제

```dart
import 'dart:async';
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

Future<Post> fetchPost() async {
  final response =
      await http.get('https://jsonplaceholder.typicode.com/posts/1');

  if (response.statusCode == 200) {
    // 만약 서버로의 요청이 성공하면, JSON을 파싱합니다.
    return Post.fromJson(json.decode(response.body));
  } else {
    // 만약 요청이 실패하면, 에러를 던집니다.
    throw Exception('Failed to load post');
  }
}

class Post {
  final int userId;
  final int id;
  final String title;
  final String body;

  Post({this.userId, this.id, this.title, this.body});

  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      userId: json['userId'],
      id: json['id'],
      title: json['title'],
      body: json['body'],
    );
  }
}

void main() => runApp(MyApp(post: fetchPost()));

class MyApp extends StatelessWidget {
  final Future<Post> post;

  MyApp({Key key, this.post}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Fetch Data Example',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        appBar: AppBar(
          title: Text('Fetch Data Example'),
        ),
        body: Center(
          child: FutureBuilder<Post>(
            future: post,
            builder: (context, snapshot) {
              if (snapshot.hasData) {
                return Text(snapshot.data.title);
              } else if (snapshot.hasError) {
                return Text("${snapshot.error}");
              }

              // 기본적으로 로딩 Spinner를 보여줍니다.
              return CircularProgressIndicator();
            },
          ),
        ),
      ),
    );
  }
}
```
