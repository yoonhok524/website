---
title: SQLite에 데이터 저장하기
prev:
  title: WebSocket 사용하기
  path: /docs/cookbook/networking/web-sockets
next:
  title: File 읽고 쓰기
  path: /docs/cookbook/persistence/reading-writing-files
---

로컬 디바이스에 많은 데이터를 저장하고 쿼리를 요청해야 한다면, 로컬 파일이나 키-값 저장소 대신 데이터베이스를
사용해보세요. 일반적으로 데이터베이스는 다른 로컬 솔루션보다 더 빠른 쓰기, 수정, 읽기 성능을 제공합니다.

Flutter 앱은 pub에서 이용가능한 
[`sqflite`](https://pub.dartlang.org/packages/sqflite) 플러그인을 통해 SQLite 데이터베이스를
사용할 수 있습니다. 본 예제에서는 다양한 강아지 데이터를 가지고 추가, 읽기, 수정, 삭제 등과 같은 기본적인 
`sqflite` 사용법을 소개할 것입니다.

If you are new to SQLite and SQL statements, please review the [SQLite Tutorial
site](http://www.sqlitetutorial.net/) to learn the basics before completing
this recipe.
만약 SQLite와 SQL문에 대한 경험이 없다면, 본 예제를 완료하기 전에 
[SQLite Tutorial site](http://www.sqlitetutorial.net/)에서 기본적인 내용을 익히도록 하세요.

## 진행 단계

  1. 의존성 추가
  2. `Dog` 데이터 모델 정의
  3. 데이터베이스 열기
  4. `dogs` 테이블 생성
  5. 데이터베이스에 `Dog` 추가
  6. Dog 리스트 조회
  7. 데이터베이스에 있는 `Dog` 수정
  8. 데이터베이스에서 `Dog` 삭제

## 1. 의존성 추가

SQLite 데이터베이스를 사용하기 위해 `sqflite`와 `path` 패키지를 추가하세요.

  - `sqflite` 패키지는 SQLite 데이터베이스를 사용할 수 있도록 여러 클래스와 함수를 제공합니다.
  - `path` 패키지는 디스크에 저장될 데이터베이스의 올바른 위치를 정의할 수 있는 함수를 제공합니다.

```yaml
dependencies:
  flutter:
    sdk: flutter
  sqflite:
  path:
```

## 2. Dog 데이터 모델 정의

Before you create the table to store information on Dogs, take a few moments to
define the data that needs to be stored. For this example, define a Dog class
that contains three pieces of data: A unique `id`, the `name`, and the `age` of
each dog.
Dog 정보를 저장할 테이블을 생성하기 전에, 저장할 데이터 구조를 정의하겠습니다. 본 예제에서는 세 개의
데이터를 갖는 Dog 클래스를 정의할 것입니다: 각 강아지에 대해 유일한 `id`, `name`, `age`

<!-- skip -->
```dart
class Dog {
  final int id;
  final String name;
  final int age;

  Dog({this.id, this.name, this.age});
}
``` 

## 3. 데이터베이스 열기

데이터베이스에서 데이터를 읽고 쓰기 전에, 데이터베이스에 대한 연결을 활성화해야 합니다. 이는 아래 두 단계를
통해 수행됩니다:

  1. `path` 패키지의 `path` 함수와 `sqflite` 패키지의 `getDatabasesPath`를 조합하여 
  데이터베이스 파일을 위한 경로를 정의하세요. 
  2. `sqflite`의 `openDatabase` 함수를 사용하여 데이터베이스를 여세요.

<!-- skip -->
```dart
// 데이터베이스를 열고 참조 값을 얻습니다.
final Future<Database> database = openDatabase(
  // 데이터베이스 경로를 지정합니다. 참고: `path` 패키지의 `join` 함수를 사용하는 것이
  // 각 플랫폼 별로 경로가 제대로 생성됐는지 보장할 수 있는 가장 좋은 방법입니다.
  join(await getDatabasesPath(), 'doggie_database.db'),
);
```

## 4. `dogs` 테이블 생성

다음으로, 다양한 강아지 정보를 저장할 테이블을 만들겠습니다. `dogs` 테이블을 생성하세요. 이 경우,
각 `Dog`는 `id`, `name`, `age`를 갖고 있습니다. 그리고 이것들은 `dogs` 테이블에서 세 개의 
컬럼으로 표현될 것입니다.

  1. `id`는 다트의 `int` 타입이며, SQLite에는 `INTEGER` 타입으로 저장됩니다. `id`를 
  테이블의 기본키로 사용하는 것은 쿼리, 수정 성능을 높이기 위한 좋은 접근법입니다.
  2. `name`은 다트의 `String` 타입이며, SQLite에는 `TEXT` 타입으로 저장됩니다.
  3. `age` 역시 다트의 `int` 타입이므로, SQLite에는 `INTEGER` 타입으로 저장됩니다.

SQLite 데이터베이스에 저장될 수 있는 사용 가능한 자료형들에 대해 더 자세한 정보를 원한다면 
[공식 SQLite 자료형 문서](https://www.sqlite.org/datatype3.html)를
살펴보시기 바랍니다.

<!-- skip -->
```dart
final Future<Database> database = openDatabase(
  // 데이터베이스 경로를 지정합니다.
  join(await getDatabasesPath(), 'doggie_database.db'),
  // 데이터베이스가 처음 생성될 때, dog를 저장하기 위한 테이블을 생성합니다.
  onCreate: (db, version) {
    // 데이터베이스에 CREATE TABLE 수행
    return db.execute(
      "CREATE TABLE dogs(id INTEGER PRIMARY KEY, name TEXT, age INTEGER)",
    );
  },
  // 버전을 설정하세요. onCreate 함수에서 수행되며 데이터베이스 업그레이드와 다운그레이드를 
  // 수행하기 위한 경로를 제공합니다.
  version: 1,
);
``` 

## 5. 데이터베이스에 Dog 추가

이제 다양한 강아지 정보를 저장할 수 있는 테이블과 함께 데이터베이스가 준비되었습니다. 데이터를 읽고
쓸 차례입니다!

먼저, `dogs` 테이블에 `Dog`를 추가해보겠습니다. 다음 두 단계를 수행하세요:

  1. `Dog`를 `Map`으로 변환하세요
  2. `dogs` 테이블에 `Map`을 저장하기 위해
  [`insert`](https://pub.dartlang.org/documentation/sqflite/latest/sqlite_api/DatabaseExecutor/insert.html)
  를 사용하세요

<!-- skip -->
```dart
// Dog 클래스에 `toMap` 메서드를 추가하세요
class Dog {
  final int id;
  final String name;
  final int age;

  Dog({this.id, this.name, this.age});

  // dog를 Map으로 변환합니다. key는 데이터베이스 컬럼 명과 동일해야 합니다.
  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'name': name,
      'age': age,
    };
  }
}

// 다음으로 데이터베이스에 dog를 추가하는 함수를 정의합니다.
Future<void> insertDog(Dog dog) async {
  // 데이터베이스 reference를 얻습니다.
  final Database db = await database;

  // Dog를 올바른 테이블에 추가합니다. 동일한 dog가 두번 추가되는 경우를 처리하기 위해 
  // `conflictAlgorithm`을 명시할 수 있습니다.
  // 
  // 본 예제에서는, 이전 데이터를 갱신하도록 하겠습니다.
  await db.insert(
    'dogs',
    dog.toMap(),
    conflictAlgorithm: ConflictAlgorithm.replace,
  );
}

// 이제 Dog를 생성하고 dogs 테이블에 추가할 수 있습니다!
final fido = Dog(
  id: 0, 
  name: 'Fido', 
  age: 35,
);

await insertDog(fido);
```

## 6. Dog 리스트 조회

이제 데이터베이스에 하나의 `Dog` 정보가 저장되어 있으므로, 특정 dog를 조회하거나 모든 dog 리스트를
조회할 수 있습니다! 다음 두 단계로 수행할 수 있습니다:

  1. `dogs` 테이블에 `query`를 수행하세요. `List<Map>`를 반환할 것입니다.
  2. `List<Map>`을 `List<Dog>`로 변환하세요.
  
<!-- skip -->
```dart
// dogs 테이블의 모든 dog를 얻는 메서드
Future<List<Dog>> dogs() async {
  // 데이터베이스 reference를 얻습니다.
  final Database db = await database;

  // 모든 Dog를 얻기 위해 테이블에 질의합니다.
  final List<Map<String, dynamic>> maps = await db.query('dogs');

  // List<Map<String, dynamic>를 List<Dog>으로 변환합니다.
  return List.generate(maps.length, (i) {
    return Dog(
      id: maps[i]['id'],
      name: maps[i]['name'],
      age: maps[i]['age'],
    );
  });
}

// 이제, 모든 dog를 얻을 수 있는 위 메서드를 사용할 수 있습니다!
print(await dogs()); // Fido를 포함한 리스트를 출력합니다
```

## 7. 데이터베이스에 있는 `Dog` 수정

데이터베이스에 어떤 정보를 추가하고 이후에 해당 정보를 수정하고 싶은 경우가 있습니다. 이를 위해
`sqflite` 라이브러리의 
[`update`](https://pub.dartlang.org/documentation/sqflite/latest/sqlite_api/DatabaseExecutor/update.html)
메서드를 사용하세요.

다음 두 단계로 수행할 수 있습니다:

  1. Dog를 Map으로 변환하세요
  2. 올바른 Dog를 수정하기 위해 `where` 절을 사용하세요

<!-- skip -->
```dart
Future<void> updateDog(Dog dog) async {
  // 데이터베이스 reference를 얻습니다.
  final db = await database;

  // 주어진 Dog를 수정합니다.
  await db.update(
    'dogs',
    dog.toMap(),
    // id가 일치하는 Dog만 수정하도록 확인합니다.
    where: "id = ?",
    // Dog의 id를 where의 인자로 넘겨 SQL injection을 방지합니다.
    whereArgs: [dog.id],
  );
}

// 이제 Fido의 나이를 수정할 수 있습니다!
await updateDog(Dog(
  id: 0, 
  name: 'Fido', 
  age: 42,
));

// 그리고 수정된 결과를 출력할 수 있습니다.
print(await dogs()); // Fido의 정보를 나이 42와 함께 출력합니다.
```

{{site.alert.warning}}
`where` 절에 인자를 전달할 때에는 항상 `whereArgs`를 사용하세요. SQL injection 
공격으로부터 보호할 수 있습니다.

`where: "id = ${dog.id}"`와 같은 String interpolation 문법을 사용하지 마세요!
{{site.alert.end}}


## 8. 데이터베이스에서 `Dog` 삭제

Dog에 대한 정보를 추가, 수정하는 것외에 데이터베이스에서 dog를 제거할 수도 있습니다. 데이터를 제거하기 위해
`sqflite` 라이브러리의 
[`delete`](https://pub.dartlang.org/documentation/sqflite/latest/sqlite_api/DatabaseExecutor/delete.html)
메서드를 사용하세요

여기에서는 하나의 id 값을 인자로 받아 데이터베이스에서 id 값이 일치하는 dog를 삭제하는 함수를 만들 것입니다.
삭제될 항목을 제한하기 위해 `where` 절을 제공해야 합니다.

<!-- skip -->
```dart
Future<void> deleteDog(int id) async {
  // 데이터베이스 reference를 얻습니다.
  final db = await database;

  // 데이터베이스에서 Dog를 삭제합니다.
  await db.delete(
    'dogs',
    // 특정 dog를 제거하기 위해 `where` 절을 사용하세요
    where: "id = ?",
    // Dog의 id를 where의 인자로 넘겨 SQL injection을 방지합니다.
    whereArgs: [id],
  );
}
```

## 예제

예제를 실행하기 위해:

  1. 새로운 Flutter 프로젝트를 생성하세요
  2. `pubspec.yaml`에 `sqflite`와 `path` 패키지를 추가하세요
  3. 새로운 파일 `lib/db_test.dart`에 아래 코드를 붙여 넣으세요
  4. `flutter run lib/db_test.dart` 코드를 실행하세요

```dart
import 'dart:async';

import 'package:path/path.dart';
import 'package:sqflite/sqflite.dart';

void main() async {
  final database = openDatabase(
    // 데이터베이스 경로를 지정합니다. 참고: `path` 패키지의 `join` 함수를 사용하는 것이
    // 각 플랫폼 별로 경로가 제대로 생성됐는지 보장할 수 있는 가장 좋은 방법입니다.
    join(await getDatabasesPath(), 'doggie_database.db'),
    // 데이터베이스가 처음 생성될 때, dog를 저장하기 위한 테이블을 생성합니다.
    onCreate: (db, version) {
      return db.execute(
        "CREATE TABLE dogs(id INTEGER PRIMARY KEY, name TEXT, age INTEGER)",
      );
    },
    // 버전을 설정하세요. onCreate 함수에서 수행되며 데이터베이스 업그레이드와 다운그레이드를 
    // 수행하기 위한 경로를 제공합니다.
    version: 1,
  );

  Future<void> insertDog(Dog dog) async {
    // 데이터베이스 reference를 얻습니다.
    final Database db = await database;

    // Dog를 올바른 테이블에 추가하세요. 본 예제에서는 `conflictAlgorithm`을 명시할 것입니다.
    // 만약 동일한 dog가 여러번 추가되면, 이전 데이터를 덮어쓸 것입니다.
    await db.insert(
      'dogs',
      dog.toMap(),
      conflictAlgorithm: ConflictAlgorithm.replace,
    );
  }

  Future<List<Dog>> dogs() async {
    // 데이터베이스 reference를 얻습니다.
    final Database db = await database;

    // 모든 Dog를 얻기 위해 테이블에 질의합니다.
    final List<Map<String, dynamic>> maps = await db.query('dogs');

    // List<Map<String, dynamic>를 List<Dog>으로 변환합니다.
    return List.generate(maps.length, (i) {
      return Dog(
        id: maps[i]['id'],
        name: maps[i]['name'],
        age: maps[i]['age'],
      );
    });
  }

  Future<void> updateDog(Dog dog) async {
    // 데이터베이스 reference를 얻습니다.
    final db = await database;

    // 주어진 Dog를 수정합니다.
    await db.update(
      'dogs',
      dog.toMap(),
      // id가 일치하는 Dog만 수정하도록 확인합니다.
      where: "id = ?",
      // Dog의 id를 where의 인자로 넘겨 SQL injection을 방지합니다.
      whereArgs: [dog.id],
    );
  }

  Future<void> deleteDog(int id) async {
    // 데이터베이스 reference를 얻습니다.
    final db = await database;

    // 데이터베이스에서 Dog를 삭제합니다.
    await db.delete(
      'dogs',
      // 특정 dog를 제거하기 위해 `where` 절을 사용하세요
      where: "id = ?",
      // Dog의 id를 where의 인자로 넘겨 SQL injection을 방지합니다.
      whereArgs: [id],
    );
  }

  var fido = Dog(
    id: 0,
    name: 'Fido',
    age: 35,
  );

  // 데이터베이스에 dog를 추가합니다.
  await insertDog(fido);

  // dog 목록을 출력합니다. (지금은 Fido만 존재합니다.)
  print(await dogs());

  // Fido의 나이를 수정한 뒤 데이터베이스에 저장합니다.
  fido = Dog(
    id: fido.id,
    name: fido.name,
    age: fido.age + 7,
  );
  await updateDog(fido);

  // Fido의 수정된 정보를 출력합니다.
  print(await dogs());

  // Fido를 데이터베이스에서 제거합니다.
  await deleteDog(fido.id);

  // dog 목록을 출력합니다. (비어있습니다.)
  print(await dogs());
}

class Dog {
  final int id;
  final String name;
  final int age;

  Dog({this.id, this.name, this.age});

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'name': name,
      'age': age,
    };
  }

  // 각 dog 정보를 보기 쉽도록 print 문을 사용하여 toString을 구현하세요
  @override
  String toString() {
    return 'Dog{id: $id, name: $name, age: $age}';
  }
}
```

