---
title: 레이아웃 튜토리얼
short-title: 튜토리얼
description: 레이아웃을 만드는 방법을 배웁니다.
diff2html: true
---

{% assign api = '{{site.api}}/flutter' -%}
{% capture examples -%} {{site.repo.this}}/tree/{{site.branch}}/examples {%- endcapture -%}
{% assign rawExFile = 'https://raw.githubusercontent.com/flutter/website/master/examples' -%}
{% capture demo -%} {{site.repo.flutter}}/tree/{{site.branch}}/examples/flutter_gallery/lib/demo {%- endcapture -%}

<style>dl, dd { margin-bottom: 0; }</style>

{{site.alert.secondary}}
  <h4 class="no_toc">다음을 배우게 됩니다.</h4>

  * Flutter의 레이아웃 메커니즘이 동작하는 방식.
  * 위젯을 수평과 수직으로 배치하는 방법.
  * Flutter 레이아웃을 만드는 방법.
{{site.alert.end}}

이 문서는 Flutter에서 레이아웃을 작성하는 방법을 소개합니다.
You'll build the layout for the following app.

{% include app-figure.md img-class="site-mobile-screenshot border"
    image="ui/layout/lakes.jpg" caption="The finished app" %}

Flutter의 레이아웃을 다루기 전에,
화면 위에 하나의 위젯을 배치하는 방법을 살펴보도록 하겠습니다.
위젯을 수평, 수직으로 배치하는 방법을 논의한 다음에는 가장 일반적인 레이아웃 위젯도 다뤄보겠습니다.

레이아웃 메커니즘을 이해하기 위한 "큰 그림"을 알고싶으신 경우라면,
[Flutter's approach to layout](/docs/development/ui/layout)을 찾아보세요.

## Step 0: 앱 기본 코드 만들기

[set up](/docs/get-started/install) 과정을 통해 개발 환경을 잘 구성하셨다면,
다음의 순서를 따라보세요.

 1. [Create a basic "Hello World" Flutter app][hello-world].
 2. 아래의 코드를 적용해서 app bar title과 app title을 바꿔보세요.

    <?code-excerpt "{codelabs/startup_namer/step1_base,layout/base}/lib/main.dart"?>
    ```diff
    --- codelabs/startup_namer/step1_base/lib/main.dart
    +++ layout/base/lib/main.dart
    @@ -6,10 +6,10 @@
       @override
       Widget build(BuildContext context) {
         return MaterialApp(
    -      title: 'Welcome to Flutter',
    +      title: 'Flutter layout demo',
           home: Scaffold(
             appBar: AppBar(
    -          title: Text('Welcome to Flutter'),
    +          title: Text('Flutter layout demo'),
             ),
             body: Center(
               child: Text('Hello World'),
    ```

[hello-world]: /docs/get-started/codelab#step-1-create-the-starter-flutter-app

## Step 1: 레이아웃 그리기

첫 단계로 레이아웃을 기본 요소들로 나눠보겠습니다.

* Row와 Column 레이아웃을 확인합니다.
* 레이아웃이 그리드를 포함하나요?
* 겹쳐친 요소가 있나요?
* UI에 탭이 필요한가요?
* 정렬, 패딩, 보더가 필요한 영역에 주의합니다.

먼저, 더 큰 요소들을 구분합니다. 이 예제에서는 4개의 요소를 가지고 있는데, 
이미지 하나, 두 개의 row 레이아웃, 텍스트 영역 하나입니다.

{% include app-figure.md img-class="site-mobile-screenshot border"
    image="ui/layout/lakes-column-elts.png" caption="Column elements (circled in red)" %}

다음에는 각 row를 그려봅니다. 첫 번째 row 타이틀 영역으로 
텍스트 column 하나, 별 아이콘 하나, 그리고 숫자를 가지고 있습니다.
첫 번째 자식 요소는 두 줄짜리 텍스트 column입니다.
이 column은 넓은 영역을 차지하고 있기 때문에, 
Expanded 위젯을 사용해야 합니다.

{% include app-figure.md image="ui/layout/title-section-parts.png" alt="Title section" %}

두 번째 row는 세 개의 자식 요소를 가지고 있고, 
각 요소는 아이콘과 텍스트를 가지고 있는 column 입니다.

{% include app-figure.md image="ui/layout/button-section-diagram.png" alt="Button section" %}

레이아웃을 다 그렸으면, 가장 하위 요소부터 구현합니다.
깊이 파묻힌 레이아웃 코드의 시각적 혼란을 
최소화 하기 위해서, 적당히 함수와 변수을 이용해서 구현합니다.

## Step 2: 타이틀 row 넣기

<?code-excerpt path-base="layout/lakes/step2"?>

먼저, 타이틀 영역의 좌측에 column을 만듭니다.
`MyApp` 클래스의 `build()` 메소드 가장 위에 다음의 코드를 추가합니다.

<?code-excerpt "lib/main.dart (titleSection)" title?>
```dart
Widget titleSection = Container(
  padding: const EdgeInsets.all(32),
  child: Row(
    children: [
      Expanded(
        /*1*/
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            /*2*/
            Container(
              padding: const EdgeInsets.only(bottom: 8),
              child: Text(
                'Oeschinen Lake Campground',
                style: TextStyle(
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
            Text(
              'Kandersteg, Switzerland',
              style: TextStyle(
                color: Colors.grey[500],
              ),
            ),
          ],
        ),
      ),
      /*3*/
      Icon(
        Icons.star,
        color: Colors.red[500],
      ),
      Text('41'),
    ],
  ),
);
```

{:.numbered-code-notes}
 1. Expanded widget안에 Column을 넣으면 row의 남은 공간을 모두 채우게 됩니다.
    `crossAxisAlignment` property를 `CrossAxisAlignment.start`로 
    설정하면 row의 시작 위치에 column을 배치합니다.
 2. Container안에 첫 번째 row의 텍스트를 넣으면 패딩을 설정할 수 있습니다.
    Column의 두 번째 자식은 글자색이 회색인 텍스트입니다.
 3. 타이틀 row의 남은 두 개의 아이템은 붉은색의 별모양 아이콘과 "41"이라는 텍스트입니다.
    전체 row는 Container안에 들어있고, 사방으로 32 pixel의 패딩이 들어갑니다.

타이틀 영역을 app body에 넣어보겠습니다.

<?code-excerpt path-base="layout/lakes"?>
<?code-excerpt "{../base,step2}/lib/main.dart" from="return MaterialApp"?>
```diff
--- ../base/lib/main.dart
+++ step2/lib/main.dart
@@ -8,11 +46,13 @@
     return MaterialApp(
       title: 'Flutter layout demo',
       home: Scaffold(
         appBar: AppBar(
           title: Text('Flutter layout demo'),
         ),
-        body: Center(
-          child: Text('Hello World'),
+        body: Column(
+          children: [
+            titleSection,
+          ],
         ),
       ),
     );
```

{{site.alert.tip}}
  - 코드를 복사해서 붙여넣기 하면 정렬이 어긋날 수 있습니다. 그럴 경우 
    Flutter 편집기에서 [automatic reformatting support](/docs/development/tools/formatting)를 사용해보세요.
  - 개발 내용을 즉시 반영하고 싶으시다면, Flutter'의 [hot reload][]가 있습니다.
  - 문제가 발생한다면, [lib/main.dart][]와 여러분의 코드를 비교해보세요.

  [hot reload]: /docs/development/tools/hot-reload
  [lib/main.dart]: {{examples}}/layout/lakes/step2/lib/main.dart
{{site.alert.end}}

## Step 3: 버튼 row 넣기

<?code-excerpt path-base="layout/lakes/step3"?>

버튼 영역은 모두 동일하게 텍스트 위에 아이콘을 배치한 column 레이아웃 3개를 담고 있습니다.
이 row의 column은 모두 동일한 영역을 차지합니다. 그리고, 텍스트와 아이콘은 기본 색으로 
그려집니다.

각 컬럼의 코드는 거의 동일하기 때문에, 색과, 아이콘, 그리고 텍스트를 인수로 받아 주어진 색으로 
칠한 위젯을 반환하는 `buildButtonColumn()` 이라는 private 헬퍼 메소드를 만듭니다.

<?code-excerpt "lib/main.dart (_buildButtonColumn)" title?>
```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // ···
  }

  Column _buildButtonColumn(Color color, IconData icon, String label) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Icon(icon, color: color),
        Container(
          margin: const EdgeInsets.only(top: 8),
          child: Text(
            label,
            style: TextStyle(
              fontSize: 12,
              fontWeight: FontWeight.w400,
              color: color,
            ),
          ),
        ),
      ],
    );
  }
}
```

함수는 column에 아이콘을 바로 넣습니다. 텍스트는 아이콘과 구분하는 상단 마진을 포함해서 컨테이너에 들어갑니다.

이 함수에 해당 column을 위한 색, `Icon`과 텍스트와 함께 호출해서 row를 작성해보세요.
main axis의 정렬은 column들이 모두 동일한 영역을 차지하도록 `MainAxisAlignment.spaceEvenly`를 사용합니다.
다음의 코드를 `build()` 메소드 안에 있는 `titleSection` 다음에 추가합니다.

<?code-excerpt "lib/main.dart (buttonSection)" title?>
```dart
Color color = Theme.of(context).primaryColor;

Widget buttonSection = Container(
  child: Row(
    mainAxisAlignment: MainAxisAlignment.spaceEvenly,
    children: [
      _buildButtonColumn(color, Icons.call, 'CALL'),
      _buildButtonColumn(color, Icons.near_me, 'ROUTE'),
      _buildButtonColumn(color, Icons.share, 'SHARE'),
    ],
  ),
);
```

버튼 영역을 body에 넣습니다.

<?code-excerpt path-base="layout/lakes"?>
<?code-excerpt "{step2,step3}/lib/main.dart" from="return MaterialApp" to="}"?>
```diff
--- step2/lib/main.dart
+++ step3/lib/main.dart
@@ -46,3 +59,3 @@
     return MaterialApp(
       title: 'Flutter layout demo',
       home: Scaffold(
@@ -52,8 +65,9 @@
         body: Column(
           children: [
             titleSection,
+            buttonSection,
           ],
         ),
       ),
     );
   }
```

## Step 4: 텍스트 영역 넣기

<?code-excerpt path-base="layout/lakes/step4"?>

텍스트 영역을 변수로 정의합니다. Container안에 텍스트를 넣고, 사방에 패딩을 넣습니다.
다음의 코드를 앞에서 정의한 `buttonSection` 바로 밑에 추가합니다.

<?code-excerpt "lib/main.dart (textSection)" title?>
```dart
Widget textSection = Container(
  padding: const EdgeInsets.all(32),
  child: Text(
    'Lake Oeschinen lies at the foot of the Blüemlisalp in the Bernese '
        'Alps. Situated 1,578 meters above sea level, it is one of the '
        'larger Alpine Lakes. A gondola ride from Kandersteg, followed by a '
        'half-hour walk through pastures and pine forest, leads you to the '
        'lake, which warms to 20 degrees Celsius in the summer. Activities '
        'enjoyed here include rowing, and riding the summer toboggan run.',
    softWrap: true,
  ),
);
```

`softwrap`을 true로 설정하면, 단어로 줄바꿈 하기 전에 column의 너비를 채웁니다.

텍스트 영역을 body에 추가합니다.

<?code-excerpt path-base="layout/lakes"?>
<?code-excerpt "{step3,step4}/lib/main.dart" from="return MaterialApp"?>
```diff
--- step3/lib/main.dart
+++ step4/lib/main.dart
@@ -59,3 +72,3 @@
     return MaterialApp(
       title: 'Flutter layout demo',
       home: Scaffold(
@@ -66,6 +79,7 @@
           children: [
             titleSection,
             buttonSection,
+            textSection,
           ],
         ),
       ),
```

## Step 5: 이미지 영역 넣기

4개의 column 요소들 중에서 이제 3개를 완성하고 이미지만 남았습니다.
예제에 이미지 파일을 주가합니다.

* `images` 디렉토리를 프로젝트의 최상위 폴더에 만듭니다.
* [`lake.jpg`]({{rawExFile}}/layout/lakes/step5/images/lake.jpg)를 추가합니다.

  {{site.alert.info}}
    이 바이너리 파일을 다운로드 할 때, `wget`은 동작하지 않습니다. 원본 이미지는 
    Creative Commons 라이센스로 [온라인][]에서 볼 수 있지만, 크고 느립니다.

    [온라인]: https://images.unsplash.com/photo-1471115853179-bb1d604434e0?dpr=1&amp;auto=format&amp;fit=crop&amp;w=767&amp;h=583&amp;q=80&amp;cs=tinysrgb&amp;crop=
  {{site.alert.end}}

* `assets` 태그를 넣기 위해 `pubspec.yaml`을 업데이트 합니다. 그러면 여러분의 
  코드에서 이미지를 사용할 수 있습니다.

  <?code-excerpt "{step4,step5}/pubspec.yaml"?>
  ```diff
  --- step4/pubspec.yaml
  +++ step5/pubspec.yaml
  @@ -17,3 +17,5 @@

   flutter:
     uses-material-design: true
  +  assets:
  +    - images/lake.jpg
  ```

이제 코드에서 이미지를 참조할 수 있습니다.

<?code-excerpt "{step4,step5}/lib/main.dart"?>
```diff
--- step4/lib/main.dart
+++ step5/lib/main.dart
@@ -77,6 +77,12 @@
         ),
         body: Column(
           children: [
+            Image.asset(
+              'images/lake.jpg',
+              width: 600,
+              height: 240,
+              fit: BoxFit.cover,
+            ),
             titleSection,
             buttonSection,
             textSection,
```

`BoxFit.cover` 는 가능한 작게 하지만 렌더 박스 전체를 덮을 정도로만 작아야 한다는 것을
프레임워크에 알려줍니다.

## Step 6: 마지막 작업

마지막 단계로 `Column`을 보다 작은 디바이스에서 스크롤이 가능하도록 `ListView`로 바꿔봅니다.

<?code-excerpt "{step5,step6}/lib/main.dart" diff-u="6" from="return MaterialApp"?>
```diff
--- step5/lib/main.dart
+++ step6/lib/main.dart
@@ -72,13 +77,13 @@
     return MaterialApp(
       title: 'Flutter layout demo',
       home: Scaffold(
         appBar: AppBar(
           title: Text('Flutter layout demo'),
         ),
-        body: Column(
+        body: ListView(
           children: [
             Image.asset(
               'images/lake.jpg',
               width: 600,
               height: 240,
               fit: BoxFit.cover,
```

**Dart code:** [main.dart]({{examples}}/layout/lakes/step6/lib/main.dart)<br>
**Image:** [images]({{examples}}/layout/lakes/step6/images)<br>
**Pubspec:** [pubspec.yaml]({{examples}}/layout/lakes/step6/pubspec.yaml)

다 됐습니다! 앱을 hot reload해보면 이 문서의 상단에 있는 스크린샷과 같은 앱 레이아웃을 보실수 있습니다.

[Adding Interactivity to Your Flutter App](/docs/development/ui/interactive)을 통해 레이아웃에
상호작용을 추가할 수 있습니다..
