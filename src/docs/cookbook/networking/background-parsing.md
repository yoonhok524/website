---
title: 백그라운드에서 JSON 파싱하기
prev:
  title: 인증된 요청 생성하기
  path: /docs/cookbook/networking/authenticated-requests
next:
  title: WebSockets으로 작업하기
  path: /docs/cookbook/networking/web-sockets
---

기본적으로, 다트 앱은 모든 작업을 단일 스레드에서 수행합니다. 대부분의 경우 이러한 모델은
코딩을 단순화시키며, 앱 성능이 떨어지거나 "jank"라고 불리는 뚝뚝 끊기는 애니메이션을 
야기하지 않을 만큼 충분히 빠릅니다.

하지만 아주 큰 JSON 문서를 파싱하는 것과 같은 값 비싼 연산을 해야하는 경우가 있습니다.
만약 이러한 작업이 16ms 넘게 걸린다면, 사용자는 jank를 경험하게 될 것 입니다.

Jank를 피하기 위해서는, 이러한 값 비싼 연산을 백그라운드에서 수행해야 합니다. 
Android에서는, 다른 스레드에 작업을 스케줄링하는 것을 의미합니다. Flutter에서는,
별도의 [Isolate]({{site.api}}/flutter/dart-isolate/Isolate-class.html)를 
사용할 수 있습니다.

## 진행 단계

  1. `http` 패키지 추가하기
  2. `http` 패키지를 사용하여 네트워크 요청 생성하기
  3. 응답 결과를 Photo 리스트로 변환하기
  4. 이 작업을 별도 isolte로 옮기기

## 1. `http` 패키지 추가하기

먼저 프로젝트에 [`http`]({{site.pub-pkg}}/http) 패키지를 추가하세요.
`http` 패키지를 통해 JSON 엔드 포인트로부터 데이터를 받아오는 것과 같은
네트워크 통신을 쉽게 수행할 수 있습니다.

```yaml
dependencies:
  http: <latest_version>
```

## 2. 네트워크 요청 생성하기

본 예제에서는 [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com)
로부터 5000개의 photo 객체가 포함된 아주 큰 사이즈의 JSON 문서를 
[http.get()]({{site.pub-api}}/http/latest/http/get.html) 메서드를 사용하여
받아올 것입니다.

<!-- skip -->
```dart
Future<http.Response> fetchPhotos(http.Client client) async {
  return client.get('https://jsonplaceholder.typicode.com/photos');
}
```

참고: 본 예제에서는 함수에 `http.Client`를 제공하고 있는데, 이로인해 해당 함수를
테스트하거나 다른 환경에서도 사용하기 쉬워집니다.

## 3. json을 Photo 리스트로 파싱하여 변환하기

다음으로, [Fetch data from the internet](/docs/cookbook/networking/fetch-data)의
가이드를 따라하세요. `http.Response`를 다트 객체의 리스트로 변환할 건데 그렇게 데이터를
벼환하고 나면 이후 작업을 더 쉽게 수행할 수 있습니다.

### `Photo` 클래스를 정의하세요

먼저 사진 관련 데이터를 담을 `Photo` 클래스를 정의합니다. json 객체를 통해 
`Photo`를 좀더 쉽게 생성하기 위해 `fromJson` 팩토리 메서드를 같이 정의하세요 

<!-- skip -->
```dart
class Photo {
  final int id;
  final String title;
  final String thumbnailUrl;

  Photo({this.id, this.title, this.thumbnailUrl});

  factory Photo.fromJson(Map<String, dynamic> json) {
    return Photo(
      id: json['id'] as int,
      title: json['title'] as String,
      thumbnailUrl: json['thumbnailUrl'] as String,
    );
  }
}
```

### 응답 결과를 Photo 리스트로 변환하기

이제 `Future<List<Photo>>`를 반환하도록 `fetchPhotos` 함수를 수정하겠습니다.
이 작업을 하기 위해 다음의 것들이 필요합니다:

  1. Create a `parsePhotos` that converts the response body into a `List<Photo>`
  1. 응답 결과를 `List<Photo>`로 변환할 `parsePhotos` 를 생성하세요
  2. `fetchPhotos` 함수에서 `parsePhotos` 함수를 사용하세요

<!-- skip -->
```dart
// 응답 결과를 List<Photo>로 변환하는 함수
List<Photo> parsePhotos(String responseBody) {
  final parsed = json.decode(responseBody).cast<Map<String, dynamic>>();

  return parsed.map<Photo>((json) => Photo.fromJson(json)).toList();
}

Future<List<Photo>> fetchPhotos(http.Client client) async {
  final response =
      await client.get('https://jsonplaceholder.typicode.com/photos');

  return parsePhotos(response.body);
}
```

## 4. 이 작업을 별도 isolte로 옮기기

만약 `fetchPhotos` 함수를 구형의 오래된 폰에서 돌려보면, 앱이 json을 파싱하고 변환하는
과정에서 버벅대는 것을 느낄 수도 있습니다. 이런 현상을 jank라고 부르는데 이 현상을
개선하겠습니다.

[`compute`]({{site.api}}/flutter/foundation/compute.html) 함수를 사용하여
파싱하고 변환하는 작업을 백그라운드 isolate으로 옮기면 됩니다. `compute` 함수는
오래 걸리는 함수를 백그라운드 isolate에서 돌리고 그 결과를 반환합니다. 본 예제에서는
`parsePhotos` 함수를 백그라운드에서 수행할 것입니다.

<!-- skip -->
```dart
Future<List<Photo>> fetchPhotos(http.Client client) async {
  final response =
      await client.get('https://jsonplaceholder.typicode.com/photos');

  // compute 함수를 사용하여 parsePhotos를 별도 isolate에서 수행합니다
  return compute(parsePhotos, response.body);
}
```

## Isolate 작업에 대한 참고 사항

Isolate는 메세지를 주고 받으며 통신합니다. 메세지는 `null`, `num`, `bool`, `double`,
`String` 과 같은 원시 타입이거나 본 예제에서 사용한 `List<Photo>`와 같은 간단한 객체일 수 
있습니다. 

Isolate 간에 `Future`나 `http.Response`와 같이 다소 복잡한 객체를 주고 받을 경우
에러가 발생할 수도 있습니다.

## 완성된 예제

```dart
import 'dart:async';
import 'dart:convert';

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

Future<List<Photo>> fetchPhotos(http.Client client) async {
  final response =
      await client.get('https://jsonplaceholder.typicode.com/photos');

  // compute 함수를 사용하여 parsePhotos를 별도 isolate에서 수행합니다
  return compute(parsePhotos, response.body);
}

// 응답 결과를 List<Photo>로 변환하는 함수
List<Photo> parsePhotos(String responseBody) {
  final parsed = json.decode(responseBody).cast<Map<String, dynamic>>();

  return parsed.map<Photo>((json) => Photo.fromJson(json)).toList();
}

class Photo {
  final int albumId;
  final int id;
  final String title;
  final String url;
  final String thumbnailUrl;

  Photo({this.albumId, this.id, this.title, this.url, this.thumbnailUrl});

  factory Photo.fromJson(Map<String, dynamic> json) {
    return Photo(
      albumId: json['albumId'] as int,
      id: json['id'] as int,
      title: json['title'] as String,
      url: json['url'] as String,
      thumbnailUrl: json['thumbnailUrl'] as String,
    );
  }
}

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final appTitle = 'Isolate Demo';

    return MaterialApp(
      title: appTitle,
      home: MyHomePage(title: appTitle),
    );
  }
}

class MyHomePage extends StatelessWidget {
  final String title;

  MyHomePage({Key key, this.title}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(title),
      ),
      body: FutureBuilder<List<Photo>>(
        future: fetchPhotos(http.Client()),
        builder: (context, snapshot) {
          if (snapshot.hasError) print(snapshot.error);

          return snapshot.hasData
              ? PhotosList(photos: snapshot.data)
              : Center(child: CircularProgressIndicator());
        },
      ),
    );
  }
}

class PhotosList extends StatelessWidget {
  final List<Photo> photos;

  PhotosList({Key key, this.photos}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
      ),
      itemCount: photos.length,
      itemBuilder: (context, index) {
        return Image.network(photos[index].thumbnailUrl);
      },
    );
  }
}
```

![Isolate Demo](/images/cookbook/isolate.gif){:.site-mobile-screenshot}
