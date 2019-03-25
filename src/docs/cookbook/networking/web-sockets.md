---
title: WebSockets으로 작업하기
prev:
  title: 백그라운드에서 JSON 파싱하기
  path: /docs/cookbook/networking/background-parsing
next:
  title: SQLite에 데이터 저장하기
  path: /docs/cookbook/persistence/sqlite
---

일반적인 HTTP 요청 외에도, WebSocket을 사용하여 서버에 연결할 수 있습니다.
WebSocket은 폴링 방식이 아닌 서버와의 양방향 통신을 지원합니다.

본 예제에서는, [websocket.org에서 제공하는 테스트 서버](http://www.websocket.org/echo.html)와
통신할 것입니다. 이 테스트 서버는 단순히 클라이언트가 보낸 메세지를 그대로 회신합니다.

## 진행 단계

  1. WebSocket 서버에 접속하기
  2. 서버 메세지 수신하기
  3. 서버에 데이터 보내기
  4. WebSocket 연결 종료하기

## 1. WebSocket 서버에 접속하기

[web_socket_channel]({{site.pub-pkg}}/web_socket_channel) 패키지는 WebSocket 
서버에 연결하는데 필요한 도구를 제공합니다.

패키지가 제공하는 `WebSocketChannel`을 통해 서버 메세지를 수신하거나 메세지를 서버에
보낼 수 있습니다.

Flutter에서는 아래 코드 한 줄로 서버에 연결하기 위한 `WebSocketChannel`을 생성합니다:

<!-- skip -->
```dart
final channel = IOWebSocketChannel.connect('ws://echo.websocket.org');
```

## 2. 서버 메세지 수신하기

이제 서버와 연결이 됐으므로, 서버로부터 메세지를 수신할 수 있습니다.

테스트 서버에 메세지를 보내면, 서버는 동일한 메세지를 회신합니다.

어떻게 메세지를 수신하고 화면에 보여줄 수 있을까요? 본 예제에서는 새로운 메세지를
수신하기 위해
[`StreamBuilder`]({{site.api}}/flutter/widgets/StreamBuilder-class.html) 위젯을
사용하고, 그것들을 화면에 보여주기 위한 목적으로 
[`Text`]({{site.api}}/flutter/widgets/Text-class.html) 위젯을 사용할 것입니다.

<!-- skip -->
```dart
StreamBuilder(
  stream: widget.channel.stream,
  builder: (context, snapshot) {
    return Text(snapshot.hasData ? '${snapshot.data}' : '');
  },
);
```

### 이 작업이 어떻게 이뤄질까요?

`WebSocketChannel`은 서버로부터 온 메세지의
[`Stream`]({{site.api}}/flutter/dart-async/Stream-class.html)을
제공합니다.

`Stream` 클래스는 `dart:async` 패키지의 핵심적인 부분으로 데이터 소스로부터
비동기 이벤트를 수신하는 방법을 제공합니다. 하나의 비동기 응답을 반환하는 `Future`와 
다르게, `Stream` 클래스는 시간 흐름에 따라 여러 이벤트들을 전달할 수 있습닏.

[`StreamBuilder`]({{site.api}}/flutter/widgets/StreamBuilder-class.html) 위젯은
`Stream`와 연결하여 이벤트를 받을 때마다 `builder` 함수를 통해 Flutter에게 다시 
빌드하도록 요청합니다.

## 3. 서버에 데이터 보내기

서버에 데이터를 보내기 위해, `WebSocketChannel`가 제공하는 `sink`에 `add` 메서드를
사용하여 메세지를 추가하세요.

<!-- skip -->
```dart
channel.sink.add('Hello!');
```

### 이 작업이 어떻게 이뤄질까요?

`WebSocketChannel`은 서버에 메세지를 보낼 수 있도록
[`StreamSink`]({{site.api}}/flutter/dart-async/StreamSink-class.html)를
제공합니다.

`StreamSink` 클래스는 동기나 비동기 이벤트를 데이터 소스에 추가할 수 있는 일반적인 
방법을 제공합니다.

## 4. WebSocket 연결 종료하기

WebSocket을 다 사용했다면, 연결을 끊어야 합니다.
`sink`를 닫으면, 연결이 끊깁니다.

<!-- skip -->
```dart
channel.sink.close();
```

## 완성된 예제

```dart
import 'package:flutter/foundation.dart';
import 'package:web_socket_channel/io.dart';
import 'package:flutter/material.dart';
import 'package:web_socket_channel/web_socket_channel.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final title = 'WebSocket Demo';
    return MaterialApp(
      title: title,
      home: MyHomePage(
        title: title,
        channel: IOWebSocketChannel.connect('ws://echo.websocket.org'),
      ),
    );
  }
}

class MyHomePage extends StatefulWidget {
  final String title;
  final WebSocketChannel channel;

  MyHomePage({Key key, @required this.title, @required this.channel})
      : super(key: key);

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  TextEditingController _controller = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            Form(
              child: TextFormField(
                controller: _controller,
                decoration: InputDecoration(labelText: 'Send a message'),
              ),
            ),
            StreamBuilder(
              stream: widget.channel.stream,
              builder: (context, snapshot) {
                return Padding(
                  padding: const EdgeInsets.symmetric(vertical: 24.0),
                  child: Text(snapshot.hasData ? '${snapshot.data}' : ''),
                );
              },
            )
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _sendMessage,
        tooltip: 'Send message',
        child: Icon(Icons.send),
      ), // This trailing comma makes auto-formatting nicer for build methods.
    );
  }

  void _sendMessage() {
    if (_controller.text.isNotEmpty) {
      widget.channel.sink.add(_controller.text);
    }
  }

  @override
  void dispose() {
    widget.channel.sink.close();
    super.dispose();
  }
}
```

![Web Sockets Demo](/images/cookbook/web-sockets.gif){:.site-mobile-screenshot}
