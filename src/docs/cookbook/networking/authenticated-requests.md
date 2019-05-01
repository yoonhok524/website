---
title: 인증된 요청 생성하기
prev:
  title: 인터넷에서 데이터 가져오기
  path: /docs/cookbook/networking/fetch-data
next:
  title: 백그라운드에서 JSON 파싱하기
  path: /docs/cookbook/networking/background-parsing
---

대부분의 웹 서비스에서 데이터를 가져오기 위해서는 인증 정보를 제공해야 합니다.
여러 방법들이 있지만, 가장 일반적으로 사용하는 방법은 `Authorization` HTTP 헤더를
사용하는 것입니다.

## Authorization 헤더를 추가하세요

[`http`]({{site.pub-pkg}}/http) 패키지를 통해 요청 객체에 헤더를 쉽게 추가할 수 
있습니다. 또한 일반적인 `HttpHeaders`를 위해 `dart:io` 패키지를 사용할 수도 있습니다.

<!-- skip -->
```dart
Future<http.Response> fetchPost() {
  return http.get(
    'https://jsonplaceholder.typicode.com/posts/1',
    // 백엔드에 Authorization 헤더를 보냅니다
    headers: {HttpHeaders.authorizationHeader: "Basic your_api_token_here"},
  );
}
```

## 완성된 예제

이 예제는 [인터넷에서 데이터 가져오기](/docs/cookbook/networking/fetch-data/) 예제를
기반으로 합니다.

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:http/http.dart' as http;

Future<Post> fetchPost() async {
  final response = await http.get(
    'https://jsonplaceholder.typicode.com/posts/1',
    headers: {HttpHeaders.authorizationHeader: "Basic your_api_token_here"},
  );
  final responseJson = json.decode(response.body);

  return Post.fromJson(responseJson);
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
```
