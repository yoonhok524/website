---
title: 첫 번째 Flutter 앱 작성하기 - 1부
short-title: 첫 번째 Flutter 앱 작성하기
prev:
  title: Test drive
  path: /docs/get-started/test-drive
next:
  title: 더 알아보기
  path: /docs/get-started/learn-more
diff2html: true
---

{% assign code-url = 'https://raw.githubusercontent.com/flutter/codelabs/master' -%}

{% asset get-started/startup-namer-part-1 alt="The app that you'll be building" class='site-image-right' %}

{%- comment %}
  Code highlights in this page are green, to match diff additions.
{% endcomment -%}
<style>pre .highlight { background-color: #dfd; }</style>

첫 번째 Flutter 앱을 만드는 방법을 안내합니다.
변수, 반복문, 조건문 등 기본 프로그래밍 개념과 객체지향에 친숙하다면, 이 튜토리얼을 완료할 수 있습니다.
다트 경험이나 모바일 프로그래밍 경험이 없어도 상관없습니다.

이 가이드는 코드랩 2부 중 1부입니다.
[2부](https://codelabs.developers.google.com/codelabs/first-flutter-app-pt2)는
[Google Developers](https://codelabs.developers.google.com)에서 보실 수 있습니다.
[1부](https://codelabs.developers.google.com/codelabs/first-flutter-app-pt1) 또한
[Google Developers](https://codelabs.developers.google.com)에서도 볼 수 있습니다.

## 1부에서 무엇을 만드는가
{:.no_toc}

스타트업 회사를 위해 이름을 생성하여 제안하는 간단한 모바일 앱을 구현할 것입니다.
사용자는 이름을 선택하거나 선택을 취소할 수 있으며, 가장 좋은 이름을 저장할 수 있습니다.
코드는 lazy하게 이름을 생성합니다.
사용자가 스크롤하면 더 많은 이름이 생성됩니다.
무한하게 스크롤 할 수 있습니다.

GIF 애니메이션은 1부를 완료하면 앱이 어떻게 동작하는지를 보여줍니다.

{{site.alert.secondary}}
  <h4 class="no_toc">1부에서 무엇을 배우는가</h4>

  * iOS와 Android 모두에서 자연스럽게 보이는 Flutter 앱을 작성하는 방법.
  * Flutter 앱의 기본 구조.
  * 패키지를 찾고 패키지를 사용하여 기능을 확장하기.
  * 더 빠른 개발 사이클을 위한 hot reload 사용.
  * 상태를 저장하는 위젯을 구현하는 방법.
  * Lazy하게 로드하는 무한 리스트를 만드는 방법.

  코드랩 [2부](https://codelabs.developers.google.com/codelabs/first-flutter-app-pt2)에서는
  상호작용을 추가하고, 앱 테마를 수정하며, 새 화면으로 이동하는 기능(Flutter에서 _route_ 라고 불립니다)을 추가합니다.
{{site.alert.end}}

{{site.alert.secondary}}
  <h4 class="no_toc">무엇을 사용하는가</h4>

  이 코드랩을 완료하기 위해 
  [Flutter SDK](/docs/get-started/install)와 [에디터](/docs/get-started/editor)가 필요합니다.
  이 코드랩은 안드로이드 스튜디오를 기반으로 되어있지만, 원한다면 에디터를 이용할 수도 있습니다. 

  이 코드랩을 아래 기기에서 실행할 수 있습니다:

  * 컴퓨터에 연결되어 있고 개발자 모드로 설정되어 있는 
    실제 기기 ([안드로이드](install/macos#set-up-your-android-device)
    또는 [iOS](install/macos#deploy-to-ios-devices)).
  * [iOS 시뮬레이터](install/macos#set-up-the-ios-simulator).
  * [Android 에뮬레이터](install/macos#set-up-the-android-emulator).
{{site.alert.end}}

## Step 1: Starter Flutter app 만들기 

<?code-excerpt path-base="codelabs/startup_namer/step1_base"?>

[첫 번째 Flutter 앱 시작하기](/docs/get-started/test-drive#create-app)에 있는 지침을 따라 
간단한 템플릿 기반 Flutter 앱을 만듭니다.
이름을 **startup_namer**로 지정합니다 (_myapp_ 대신).

{{site.alert.tip}}
  IDE에서 "New Flutter Project"가 보이지 않는다면,
  [Flutter와 다트 플러그인 설치](/docs/get-started/editor)가 설치되어 있는지 확인하세요.
{{site.alert.end}}

코드랩에서는 주로 다트 코드가 있는 **lib/main.dart**를 수정할 것입니다.

 1. `lib/main.dart`의 내용을 바꿉니다.<br>
    **lib/main.dart**의 모든 코드를 삭제합니다.
    화면 중앙에 "Hello World"를 표시하는 아래 코드로 대체합니다.

    <?code-excerpt "lib/main.dart" title?>
    ```dart
    import 'package:flutter/material.dart';

    void main() => runApp(MyApp());

    class MyApp extends StatelessWidget {
      @override
      Widget build(BuildContext context) {
        return MaterialApp(
          title: 'Welcome to Flutter',
          home: Scaffold(
            appBar: AppBar(
              title: Text('Welcome to Flutter'),
            ),
            body: Center(
              child: Text('Hello World'),
            ),
          ),
        );
      }
    }
    ```

    {{site.alert.tip}}
      코드를 앱에 붙여 넣을 때 들여쓰기가 깨질 수 있습니다.
      Flutter 도구를 활용하여 자동으로 문제를 해결할 수 있습니다.

      * 안드로이드 스튜디오 / IntelliJ IDEA: 코드에서 오른쪽 버튼을 클릭한 후 
      **Reformat Code with dartfmt**를 선택하세요.
      * VS Code: 오른쪽 버튼을 클릭한 후 **Format Document**를 선택하세요.
      * 터미널: `flutter format <파일명>`을 실행하세요.
    {{site.alert.end}}

 2. IDE에서 녹색 화살표를 클릭하여 [앱을 실행](/docs/get-started/test-drive#androidstudio)하세요.  
    기기에 따라, 안드로이드 또는 iOS 화면에 글자가 표시되어야 합니다. 

    {% indent %}
      {% include android-ios-figure-pair.md image="hello-world.png" alt="Hello world app" %}
    {% endindent %}

    {{site.alert.tip}}
      실제 기기에서 처음 실행할 때는, 로딩에 시간이 걸릴 수 있습니다.
      이후, 빠른 업데이트를 위해 hot reload를 사용할 수 있습니다.
      앱이 실행 중일 때 **저장**을 하면 hot reload를 수행합니다.
    {{site.alert.end}}

### 관찰
{:.no_toc}

* 이 예제는 머터리얼 앱을 만듭니다. 
  [머터리얼](https://material.io/guidelines/)은
  모바일 및 웹에서 표준으로 사용되는 시각 디자인 언어입니다.
  Flutter는 다양한 머터리얼 위젯을 제공합니다. 
* `main()` 메서드는 화살표(`=>`) 표기법을 사용합니다.
  한 줄 함수 또는 메서드에 화살표 표기법을 사용하세요.
* 앱은 `StatelessWidget`을 상속받아 앱 자체를 위젯으로 만듭니다.
  Flutter에서는 정렬, 여백, 레이아웃 등 거의 모든것이 위젯입니다.
* 머터리얼 라이브러리의 `Scaffold` 위젯은
  홈 스크린의 위젯 트리를 구성하는
  app bar, title, body 속성을 기본으로 제공합니다.  
  위젯 하위 트리는 상당히 복잡할 수 있습니다. 
* 위젯의 주된 임무는 
  다른 하위 위젯을 어떻게 표현할 지를 설명하는
  `build()` 메서드를 제공하는 것입니다.  
* 이 예제는 자식 위젯으로 `Text`을 포함하는 `Center` 위젯으로 구성됩니다. 
  Center 위젯은 하위 위젯을 화면 중앙에 정렬합니다. 

## Step 2: Use an external package

In this step, you’ll start using an open-source package named
[english_words](https://pub.dartlang.org/packages/english_words),
which contains a few thousand of the most used
English words plus some utility functions.

You can find the `english_words` package, as well as many other open source
packages, on [the Package site](https://pub.dartlang.org/flutter).

 1. The pubspec file manages the assets and dependencies for a Flutter app. In
    `pubspec.yaml`, add `english_words` (3.1.0 or higher) to the dependencies
    list:

    <?code-excerpt path-base="codelabs/startup_namer"?>
    <?code-excerpt "{step1_base,step2_use_package}/pubspec.yaml" diff-u="4" from="dependencies" to="english"?>
    ```diff
    --- step1_base/pubspec.yaml
    +++ step2_use_package/pubspec.yaml
    @@ -5,4 +5,5 @@
     dependencies:
       flutter:
         sdk: flutter
       cupertino_icons: ^0.1.2
    +  english_words: ^3.1.0
    ```

 2. While viewing the pubspec in Android Studio's editor view,
    click **Packages get**. This pulls the package into
    your project. You should see the following in the console:

    ```terminal
    $ flutter packages get
    Running "flutter packages get" in startup_namer...
    Process finished with exit code 0
    ```

    Performing `Packages get` also auto-generates the `pubspec.lock`
    file with a list of all packages pulled into the project and
    their version numbers.

 3. In `lib/main.dart`, import the new package:

    <?code-excerpt path-base="codelabs/startup_namer/step2_use_package"?>
    <?code-excerpt "lib/main.dart" title retain="/^import/" replace="/import.*?english.*/[!$&!]/g" indent-by="2"?>
    ```dart
      import 'package:flutter/material.dart';
      [!import 'package:english_words/english_words.dart';!]
    ```

    As you type, Android Studio gives you suggestions for libraries to import.
    It then renders the import string in gray, letting you know that the
    imported library is unused (so far).

 4. Use the English words package to generate the text instead of
    using the string "Hello World":

    <?code-excerpt path-base="codelabs/startup_namer"?>
    <?code-excerpt "{step1_base,step2_use_package}/lib/main.dart" from="class"?>
    ```diff
    --- step1_base/lib/main.dart
    +++ step2_use_package/lib/main.dart
    @@ -5,6 +6,7 @@
     class MyApp extends StatelessWidget {
       @override
       Widget build(BuildContext context) {
    +    final wordPair = WordPair.random();
         return MaterialApp(
           title: 'Welcome to Flutter',
           home: Scaffold(
    @@ -12,7 +14,7 @@
               title: Text('Welcome to Flutter'),
             ),
             body: Center(
    -          child: Text('Hello World'),
    +          child: Text(wordPair.asPascalCase),
             ),
           ),
         );
    ```

    {{site.alert.note}}
      "Pascal case" (also known as "upper camel case"),
      means that each word in the string, including the first one,
      begins with an uppercase letter. So, "uppercamelcase" becomes
      "UpperCamelCase".
    {{site.alert.end}}

 5. If the app is running, use the hot reload button
    <i class="material-icons align-bottom">offline_bolt</i>
    to update the running app. Each time you click hot reload,
    or save the project, you should see a different word pair,
    chosen at random, in the running app.
    This is because the word pairing is generated inside the build
    method, which is run each time the MaterialApp requires rendering,
    or when toggling the Platform in Flutter Inspector.

    {% indent %}
      {% include android-ios-figure-pair.md image="step2.png" alt="App at completion of second step" %}
    {% endindent %}

### Problems?
{:.no_toc}

If your app is not running correctly, look for typos. If needed,
use the code at the following links to get back on track.

* [pubspec.yaml]({{code-url}}/startup_namer/step2_use_package/pubspec.yaml)
* [lib/main.dart]({{code-url}}/startup_namer/step2_use_package/lib/main.dart)

## Step 3: Add a Stateful widget

<?code-excerpt path-base="codelabs/startup_namer/step3_stateful_widget"?>

State<em>less</em> widgets are immutable, meaning that their
properties can’t change&mdash;all values are final.

State<em>ful</em> widgets maintain state that might change
during the lifetime of the widget. Implementing a stateful
widget requires at least two classes: 1) a StatefulWidget class
that creates an instance of 2) a State class. The StatefulWidget
class is, itself, immutable, but the State class persists over the
lifetime of the widget.

In this step, you’ll add a stateful widget, `RandomWords`, which creates
its `State` class, `RandomWordsState`. You'll then use `RandomWords` as
a child inside the existing `MyApp` stateless widget.

 1. Create a minimal state class. Add the following to the bottom
    of `main.dart`:

    <?code-excerpt "lib/main.dart (RandomWordsState)" title region="RWS-class-only" plaster="// TODO Add build() method" indent-by="2"?>
    ```dart
      class RandomWordsState extends State<RandomWords> {
        // TODO Add build() method
      }
    ```

    Notice the declaration `State<RandomWords>`. This indicates that we're
    using the generic
    [State](https://docs.flutter.io/flutter/widgets/State-class.html)
    class specialized for use with `RandomWords`. Most of the app's logic
    and state resides here&mdash;it maintains the state for the `RandomWords`
    widget. This class saves the generated word pairs, which grows infinitely
    as the user scrolls, and favorite word pairs (in
    [part 2](https://codelabs.developers.google.com/codelabs/first-flutter-app-pt2)),
    as the user adds or removes them from the list by toggling the heart icon.

    `RandomWordsState` depends on the `RandomWords` class. You'll add that next.

 2. Add the stateful `RandomWords` widget to `main.dart`.
    The `RandomWords` widget does little else beside creating its State class:

    <?code-excerpt "lib/main.dart (RandomWords)" title indent-by="2"?>
    ```dart
      class RandomWords extends StatefulWidget {
        @override
        RandomWordsState createState() => new RandomWordsState();
      }
    ```

    After adding the state class, the IDE complains that
    the class is missing a build method. Next, you'll add a basic
    build method that generates the word pairs by moving the
    word generation code from `MyApp` to `RandomWordsState`.

 3. Add the `build()` method to `RandomWordsState`:

    <?code-excerpt "lib/main.dart (RandomWordsState)" title indent-by="2" replace="/(\n  )(.*)/$1[!$2!]/g"?>
    ```dart
      class RandomWordsState extends State<RandomWords> {
        [!@override!]
        [!Widget build(BuildContext context) {!]
        [!  final wordPair = WordPair.random();!]
        [!  return Text(wordPair.asPascalCase);!]
        [!}!]
      }
    ```

 4. Remove the word generation code from `MyApp` by making the changes shown in
    the following diff:

    <?code-excerpt path-base="codelabs/startup_namer"?>
    <?code-excerpt "{step2_use_package,step3_stateful_widget}/lib/main.dart" to="}"?>
    ```diff
    --- step2_use_package/lib/main.dart
    +++ step3_stateful_widget/lib/main.dart
    @@ -6,7 +6,6 @@
     class MyApp extends StatelessWidget {
       @override
       Widget build(BuildContext context) {
    -    final wordPair = WordPair.random();
         return MaterialApp(
           title: 'Welcome to Flutter',
           home: Scaffold(
    @@ -14,8 +13,8 @@
               title: Text('Welcome to Flutter'),
             ),
             body: Center(
    -          child: Text(wordPair.asPascalCase),
    +          child: RandomWords(),
             ),
           ),
         );
       }
    ```

 5. Restart the app.
    The app should behave as before, displaying a word
    pairing each time you hot reload or save the app.

{{site.alert.tip}}
  If you see the following warning on a hot reload, consider restarting
  the app:

  **Reloading...<br>
  Some program elements were changed during reload but did not run when
  the view was reassembled; you may need to restart the app (by pressing "R")
  for the changes to have an effect.**

  It may be a false positive, but restarting ensures that your changes
  are reflected in the app's UI.
{{site.alert.end}}


### Problems?
{:.no_toc}

If your app is not running correctly, you can use the code
at the following link to get back on track.

* [lib/main.dart]({{code-url}}/startup_namer/step3_stateful_widget/lib/main.dart)

## Step 4: Create an infinite scrolling ListView

<?code-excerpt path-base="codelabs/startup_namer/step4_infinite_list"?>

In this step, you'll expand `RandomWordsState` to generate
and display a list of word pairings. As the user scrolls, the list
displayed in a `ListView` widget, grows infinitely. `ListView`'s
`builder` factory constructor allows you to build a list view
lazily, on demand.

 1. Add a `_suggestions` list to the `RandomWordsState`
    class for saving suggested word pairings.
    Also, add a `_biggerFont` variable for making the font size larger.

    <?code-excerpt "lib/main.dart" title region="RWS-var" indent-by="2" replace="/final .*/[!$&!]/g"?>
    ```dart
      class RandomWordsState extends State<RandomWords> {
        [!final _suggestions = <WordPair>[];!]
        [!final _biggerFont = const TextStyle(fontSize: 18.0);!]
        // ···
      }
    ```

    {{site.alert.note}}
      Prefixing an identifier with an underscore [enforces
      privacy](https://www.dartlang.org/guides/language/language-tour)
      in the Dart
      language.
    {{site.alert.end}}

    Next, you'll add a `_buildSuggestions()` function to the `RandomWordsState`
    class. This method builds the `ListView` that displays the suggested
    word pairing.

    The `ListView` class provides a builder property, `itemBuilder`, that's a
    factory builder and callback function specified as an anonymous function.
    Two parameters are passed to the function&mdash;the `BuildContext`,
    and the row iterator, `i`. The iterator begins at 0 and increments
    each time the function is called, once for every suggested word pairing.
    This model allows the suggested list to grow infinitely as the user scrolls.

 2. Add a `_buildSuggestions()` function to the `RandomWordsState` class:

    <?code-excerpt "lib/main.dart (_buildSuggestions)" title indent-by="2"?>
    ```dart
      Widget _buildSuggestions() {
        return ListView.builder(
            padding: const EdgeInsets.all(16.0),
            itemBuilder: /*1*/ (context, i) {
              if (i.isOdd) return Divider(); /*2*/

              final index = i ~/ 2; /*3*/
              if (index >= _suggestions.length) {
                _suggestions.addAll(generateWordPairs().take(10)); /*4*/
              }
              return _buildRow(_suggestions[index]);
            });
      }
    ```

    {:.numbered-code-notes}
     1. The `itemBuilder` callback is called once per suggested word pairing,
        and places each suggestion into a `ListTile` row. For even rows, the
        function adds a `ListTile` row for the word pairing. For odd rows, the
        function adds a `Divider` widget to visually separate the entries. Note
        that the divider may be difficult to see on smaller devices.
     2. Add a one-pixel-high divider widget before each row in the `ListView`.
     3. The expression `i ~/ 2` divides `i` by 2 and returns an integer result.
        For example: 1, 2, 3, 4, 5 becomes 0, 1, 1, 2, 2. This calculates the
        actual number of word pairings in the `ListView`, minus the divider
        widgets.
     4. If you've reached the end of the available word pairings, then generate
        10 more and add them to the suggestions list.

    The `_buildSuggestions()` function calls `_buildRow()` once per
    word pair. This function displays each new pair in a `ListTile`,
    which allows you to make the rows more attractive in the next step.

 3. Add a `_buildRow()` function to `RandomWordsState`:

    <?code-excerpt "lib/main.dart (_buildRow)" title indent-by="2"?>
    ```dart
      Widget _buildRow(WordPair pair) {
        return ListTile(
          title: Text(
            pair.asPascalCase,
            style: _biggerFont,
          ),
        );
      }
    ```

 4. In the `RandomWordsState` class, update the `build()` method to use
    `_buildSuggestions()`, rather than directly calling the word
    generation library.
    ([Scaffold](https://docs.flutter.io/flutter/material/Scaffold-class.html)
    implements the basic Material Design visual layout.)
    Replace the method body with the highlighted code:

    <?code-excerpt "lib/main.dart (build)" title region="RWS-build" replace="/(\n  )(return.*|  .*|\);)/$1[!$2!]/g" indent-by="2"?>
    ```dart
      @override
      Widget build(BuildContext context) {
        [!return Scaffold(!]
        [!  appBar: AppBar(!]
        [!    title: Text('Startup Name Generator'),!]
        [!  ),!]
        [!  body: _buildSuggestions(),!]
        [!);!]
      }
    ```

 5. In the `MyApp` class, update the `build()` method by changing the title, and
    changing the home to be a `RandomWords` widget:

    <?code-excerpt path-base="codelabs/startup_namer"?>
    <?code-excerpt "{step3_stateful_widget,step4_infinite_list}/lib/main.dart" diff-u="4" from="class MyApp" to="}"?>
    ```diff
    --- step3_stateful_widget/lib/main.dart
    +++ step4_infinite_list/lib/main.dart
    @@ -6,15 +6,8 @@
     class MyApp extends StatelessWidget {
       @override
       Widget build(BuildContext context) {
         return MaterialApp(
    -      title: 'Welcome to Flutter',
    -      home: Scaffold(
    -        appBar: AppBar(
    -          title: Text('Welcome to Flutter'),
    -        ),
    -        body: Center(
    -          child: RandomWords(),
    -        ),
    -      ),
    +      title: 'Startup Name Generator',
    +      home: RandomWords(),
         );
       }
    ```

 6. Restart the app. You should see a list of word pairings no matter how far
    you scroll.

    {% indent %}
      {% include android-ios-figure-pair.md image="step4-infinite-list.png" alt="App at completion of fourth step" %}
    {% endindent %}

### Problems?
{:.no_toc}

If your app is not running correctly, you can use the code at the following link
to get back on track.

* [lib/main.dart]({{code-url}}/startup_namer/step4_infinite_list/lib/main.dart)

## Next steps
{:.no_toc}

{% include app-figure.md class="site-image-right" img-class="border"
    image="get-started/startup-namer.gif" caption="The app from part 2" %}

Congratulations!

You've written an interactive Flutter app that runs on both iOS and Android.
In this codelab, you've:

* Created a Flutter app from the ground up.
* Written Dart code.
* Leveraged an external, third-party library.
* Used hot reload for a faster development cycle.
* Implemented a stateful widget.
* Created a lazily loaded, infinite scrolling list.

If you would like to extend this app, proceed to
[part 2](https://codelabs.developers.google.com/codelabs/first-flutter-app-pt2)
on the
[Google Developers Codelabs](https://codelabs.developers.google.com) site,
where you add the following functionality:

* Implement interactivity by adding a clickable heart icon to save
  favorite pairings.
* Implement navigation to a new route by adding a new screen
  containing the saved favorites.
* Modify the theme color, making an all-white app.
