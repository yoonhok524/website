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

First, add the [`http`]({{site.pub-pkg}}/http) package to your project.
The `http` package makes it easier to perform network
requests, such as fetching data from a JSON endpoint.

```yaml
dependencies:
  http: <latest_version>
```

## 2. 네트워크 요청 생성하ㅣ

In this example, you'll fetch a JSON large document that contains a list of
5000 photo objects from the [JSONPlaceholder REST
API](https://jsonplaceholder.typicode.com)
using the [http.get()]({{site.pub-api}}/http/latest/http/get.html) method.

<!-- skip -->
```dart
Future<http.Response> fetchPhotos(http.Client client) async {
  return client.get('https://jsonplaceholder.typicode.com/photos');
}
```

참고: You're providing an `http.Client` to the function in this example.
This makes the function easier to test and use in different environments.

## 3. json을 Photo 리스트로 파싱하여 변환하기

Next, following the guidance from the [Fetch data from the
internet](/docs/cookbook/networking/fetch-data)
recipe, you'll want to convert the `http.Response` into a list of Dart objects.
This makes the data easier to work with in the future.

### `Photo` 클래스를 정의하세요

First, create a `Photo` class that contains data about a photo.
You will include a `fromJson` factory method to make it easy to create a
`Photo` starting with a json object.

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

Now, update the `fetchPhotos` function so it can return a
`Future<List<Photo>>`. To do so, you'll need to:

  1. Create a `parsePhotos` that converts the response body into a `List<Photo>`
  2. Use the `parsePhotos` function in the `fetchPhotos` function

<!-- skip -->
```dart
// A function that converts a response body into a List<Photo>
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

If you run the `fetchPhotos` function on a slower phone, you may notice the app
freezes for a brief moment as it parses and converts the json. This is jank,
and we want to be rid of it.

So how can we do that? By moving the parsing and conversion to a background
isolate using the [`compute`]({{site.api}}/flutter/foundation/compute.html)
function provided by Flutter. The `compute` function runs expensive
functions in a background isolate and returns the result. In this case,
we want to run the `parsePhotos` function in the background.

<!-- skip -->
```dart
Future<List<Photo>> fetchPhotos(http.Client client) async {
  final response =
      await client.get('https://jsonplaceholder.typicode.com/photos');

  // Use the compute function to run parsePhotos in a separate isolate
  return compute(parsePhotos, response.body);
}
```

## Notes on working with Isolates

Isolates communicate by passing messages back and forth. These messages can
be primitive values, such as `null`, `num`, `bool`, `double`, or `String`, or
simple objects such as the `List<Photo>` in this example.

You may experience errors if you try to pass more complex objects, such as
a `Future` or `http.Response` between isolates.

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

  // Use the compute function to run parsePhotos in a separate isolate
  return compute(parsePhotos, response.body);
}

// A function that converts a response body into a List<Photo>
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
