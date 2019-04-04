앱 빌드가 완료되면, 기기에서 초기 앱을 확인할 수 있습니다. 

{% include app-figure.md img-class="site-mobile-screenshot border"
    path-prefix="get-started" platform="iOS" image="starter-app.png"
    caption="Starter app" %}

## Hot reload

Flutter는 _hot reload_ 를 통해 빠른 개발 사이클을 지원합니다. 
재시작하거나 앱 상태 잃어버리지 않고 실행 중인 앱의 코드를 다시 로드합니다. 
앱 소스를 변경하고, hot reload가 필요하다는 신호를 IDE나 커멘드라인 도구에 보낸 후, 
시뮬레이터나 에뮬레이터 혹은 기기에서 변화를 확인하세요. 

 1. `lib/main.dart` 여세요.
 1. 문자열을 변경하세요

    {% prettify dart %}
      'You have [[strike]]pushed[[/strike]] the button this many times'
    {% endprettify %}

    에서

    {% prettify dart %}
      'You have [!clicked!] the button this many times'
    {% endprettify %}

    로 

    {{site.alert.important}}
      앱을 중단하면 _안 됩니다._ 앱이 실행 상태를 유지해야 합니다.
    {{site.alert.end}}

 1. 변경 내용을 저장하세요. {{include.save_changes}}

실행 중인 앱에 변경된 문자열이 거의 즉시 나타납니다.
