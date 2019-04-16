---
title: Mockito를 사용하여 의존성들에 대해 mock 객체 생성하기
short-title: Mocking
prev:
  title: 단위 테스트 소개
  path: /docs/cookbook/testing/unit/introduction
next:
  title: Widget 테스트 소개
  path: /docs/cookbook/testing/widget/introduction
---

어떠한 경우에는 단위 테스트가 웹 서비스나 데이터베이스로부터 데이터를 가져오는 역할을
수행하는 특정 클래스에 의존하는 경우가 있습니다.

  * 상용 서비스나 데이터베이스를 호출하는 것은 테스트 수행 성능을 저하시킵니다.
  * 성공해야 하는 테스트일지라도 만약 웹 서비스나 데이터베이스가 예기치 못한 결과를 
    반환하면 실패할 수 있습니다. 이러한 경우를 다른 말로 "flaky test"라고 합니다.
  * 상용 서비스나 데이터베이스를 사용하여 모든 가능한 성공 & 실패 시나리오를 테스트하는
    것은 어렵습니다.

그러므로, 상용 웹 서비스나 데이터베이스에 의존하기 보다는, 이러한 의존성을 
"mock" 으로 이용할 수 있습니다. Mock은 상용 서비스나 데이터베이스를 가상으로
돌릴 수 있게 하며, 상황에 따라 특정 결과 값을 반환하도록 할 수 있습니다.

일반적으로 클래스의 모의 구현체를 만들어 mock 의존성을 주입할 수 있습니다.
이러한 모의 구현체는 직접 구현할 수도 있지만, [Mockito 패키지]({{site.pub-pkg}}/mockito)르
사용하여 만들 수도 있습니다.

본 에제에서는 Mockito 패키지를 사용하여 Mock 객체를 생성하는 기본적인 방법을 
소개할 것입니다. 더 자세한 내용은 [Mockito 패키지 문서]({{site.pub-pkg}}/mockito)를
참고하시기 바랍니다.

## 진행 단계

  1. `mockito` & `test` 의존성을 추가하기
  2. 테스트할 함수 생성하기
  3. `http.Client` Mock 객체와 함께 테스트 파일 생성하기
  4. 각 조건마다 테스트 작성하기
  5. 테스트 수행하기

## 1. `mockito` 의존성 추가하기

`mockito` 패키지를 사용하기 위해, `pubspec.yaml` 파일의 `dev_dependencies` 영역에
`flutter_test` 의존성과 함께 추가하세요.

본 예제에서는 `http` 패키지도 필요하므로, `dependencies` 영역에 추가해주세요.

```yaml
dependencies:
  http: <newest_version>
dev_dependencies:
  test: <newest_version>
  mockito: <newest_version>
```

## 2. 테스트할 함수 생성하기

본 예제에서는 [인터넷에서 데이터 가져오기](/docs/cookbook/networking/fetch-data/) 
예제에서의 `fetchPost` 함수를 단위 테스트할 것입니다. 이 함수를 테스트하기 위해 
두 가지 변경이 필요합니다:

  1. 함수에 `http.Client`를 제공하세요. 상황에 따라 적당한 `http.Client`를 제공해줄 
     것입니다. Flutter와 서버-사이드 프로젝트에서는 `http.IOClient`를 제공할 수 있습니다.
     브라우져 앱의 경우는 `http.BrowserClient`를 제공할 수 있습니다.
     테스트 경우에는 `http.Client`의 mock 객체를 제공할 수 있습니다.
  2. mock 객체로 만들기 어려운 static `http.get` 메서드가 아닌, 
     제공된 `client`를 사용하여 인터넷으로부터 데이터를 가져오세요

변경된 함수는 아래와 같습니다:

<!-- skip -->
```dart
Future<Post> fetchPost(http.Client client) async {
  final response =
      await client.get('https://jsonplaceholder.typicode.com/posts/1');

  if (response.statusCode == 200) {
    // 만약 서버로의 요청이 성공했다면, JSON으로 파싱합니다.
    return Post.fromJson(json.decode(response.body));
  } else {
    // 만약 요청이 실패하게 되면, 에러를 던집니다.
    throw Exception('Failed to load post');
  }
}
```

## 3. `http.Client` Mock 객체와 함께 테스트 파일 생성하기

이제 `MockClient` 클래스와 함께 테스트 파일을 만들 차례입니다.
[단위 테스트 소개](/docs/cookbook/testing/unit/introduction)
예제를 따라, 최상위 `test` 폴더 안에 `fetch_post_test.dart` 파일을 만드세요.

`MockClient` 클래스는 `http.Client` 클래스를 구현할 것입니다. 이를 통해 
`MockClient`를 `fetchPost` 함수에 전달할 수 있게 되며, 각 테스트마다
다른 http 응답을 반환할 수 있게 해줍니다.

<!-- skip -->
```dart
// Mockito 패키지에서 제공되는 Mock 클래스를 사용하여 MockClient를 생성합니다.
// 각 테스트 마다 이 클래스의 새로운 인스턴스를 생성하세요.
class MockClient extends Mock implements http.Client {}

main() {
  // 테스트 코드는 여기에 위치합니다.
}
```

## 4. 각 조건마다 테스트 작성하기

`fetchPost` 함수는 두 가지 일 중 하나를 수행하게 됩니다:

  1. http 요청이 성공하면, `Post` 를 반환합니다.
  2. http 요청이 실패하면, `Exception`을 던집니다.

그러므로, 이 두 가지 경우에 대해 테스트 코드를 작성해야 합니다.
`MockClient` 클래스를 사용하여 성공 케이스를 위해 "Ok" 
응답을 반환할 수 있고, 실패 케이스를 위해 error 응답을 반환할 수 있습니다.

이를 위해 Mockito가 제공하는 `when` 함수를 사용하겠습니다.

<!-- skip -->
```dart
// Mockito 패키지에서 제공되는 Mock 클래스를 사용하여 MockClient를 생성합니다.
// 각 테스트 마다 이 클래스의 새로운 인스턴스를 생성하세요.
class MockClient extends Mock implements http.Client {}ㄴ

main() {
  group('fetchPost', () {
    test('returns a Post if the http call completes successfully', () async {
      final client = MockClient();

      // 제공된 http.Client를 호출했을 때, 성공적인 응답을 반환하기 위해 
      // Mockito를 사용합니다.
      when(client.get('https://jsonplaceholder.typicode.com/posts/1'))
          .thenAnswer((_) async => http.Response('{"title": "Test"}', 200));

      expect(await fetchPost(client), isInstanceOf<Post>());
    });

    test('throws an exception if the http call completes with an error', () {
      final client = MockClient();

      // 제공된 http.Client를 호출했을 때, 실패 응답을 반환하기 위해 
      // Mockito를 사용합니다.
      when(client.get('https://jsonplaceholder.typicode.com/posts/1'))
          .thenAnswer((_) async => http.Response('Not Found', 404));

      expect(fetchPost(client), throwsException);
    });
  });
}
```

### 5. 테스트 수행하기

이제 테스트가 있는 `fetchPost` 함수가 준비되었으니 실행해봅시다.

```terminal
$ dart test/fetch_post_test.dart
```

[단위 테스트 소개](/docs/cookbook/testing/unit/introduction#run-tests-using-intellij-or-vscode)
예제에 있는 가이드에 따라 선호하는 에디터에서도 테스트를 수행할 수 있습니다.

### 요약

본 예제에서는 웹 서비스나 데이터베이스에 의존하는 함수나 클래스를 테스트하기 위해 
Mockito를 사용하는 방법을 배웠습니다. 이 예제는 Mockito 라이브러리와 mock 컨셉에 
대한 간단한 소개에 불과합니다. 더 자세한 정보를 원한다면 
[Mockito 패키지]({{site.pub-pkg}}/mockito)에서 제공하는 문서를 참고하세요.
