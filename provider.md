

# Flutter Provider 상태관리 가이드라인

이 문서는 Flutter Provider 패키지 6.1.5 이상 버전을 기준으로 작성된 상태관리 필수 가이드라인입니다.
본 문서는 선택적이거나 권장하는 내용을 포함하지 않습니다. 대신, 반드시 따라야 할 내용을 포함하고 있으며 상태관리 작업 시 반드시 본 문서에 있는 지침대로만 코딩해야 합니다.


## 개요

Provider는 Flutter에서 가장 널리 사용되는 상태관리 솔루션 중 하나입니다. InheritedWidget을 기반으로 하여 간편하게 상태를 공유하고 관리할 수 있습니다.

## 핵심 개념

### ChangeNotifier

`ChangeNotifier`는 상태 변경을 구독자(listener)들에게 알릴 수 있는 클래스입니다. 상태가 변경되면 `notifyListeners()`를 호출하여 모든 구독자에게 알립니다.

```dart
class AppState extends ChangeNotifier {
  int _counter = 0;

  int get counter => _counter;

  // 상태 변경 후 notifyListeners() 호출하여 구독자들에게 알림
  void incrementCounter() {
    _counter++;
    notifyListeners();
  }
}
```

> **참고**: `notifyListeners()`는 내부적으로 Flutter의 `setState()`를 사용하여 상태를 업데이트합니다.

### ChangeNotifierProvider

`ChangeNotifierProvider`는 `ChangeNotifier`를 위젯 트리에 제공하고, 자동으로 리소스를 관리(dispose)합니다.

```dart
ChangeNotifierProvider(
  create: (context) => AppState(),
  child: const MyApp(),
)
```

> **중요**: `ChangeNotifier`와 `ChangeNotifierProvider`는 항상 쌍으로 사용해야 합니다.

## 설정 방법

### MultiProvider 설정

여러 개의 Provider를 사용할 때는 `MultiProvider`를 사용합니다. `main.dart`의 `runApp()`에서 설정합니다.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (context) => AppState()),
        ChangeNotifierProvider(create: (context) => RouterState()),
        ChangeNotifierProvider(create: (context) => UserState()),
      ],
      child: const MyApp(),
    ),
  );
}
```

## 상태 접근 방법

### context.read<T>()

상태를 **읽기만** 할 때 사용합니다. 상태 변경을 구독하지 않으므로 rebuild가 발생하지 않습니다.

```dart
// 이벤트 핸들러에서 상태 접근 (rebuild 불필요)
ElevatedButton(
  onPressed: () {
    context.read<AppState>().incrementCounter();
  },
  child: Text('증가'),
)
```


## Selector 위젯

### 개념 및 의도

`Selector`는 상태의 특정 부분만 구독하여 **불필요한 rebuild를 방지**하는 위젯입니다. `Consumer`보다 약간 번거롭지만, 성능 최적화와 UI 깜빡임 감소에 효과적입니다.

가능한 모든 곳에서 `Selector`를 사용하여 필요한 값만 구독합니다.

### 기본 형식

```dart
Selector<모델클래스, 데이터타입>(
  selector: (context, model) => model.값,  // 구독할 값 선택
  builder: (context, 값, child) => Text('$값'),  // 선택한 값으로 UI 빌드
)
```

- `selector`: 모델에서 구독할 특정 값을 반환합니다. 이 값이 이전 값과 다르면 `builder`가 다시 실행됩니다.
- `builder`: 선택한 값을 사용하여 위젯을 빌드합니다.
- `child`: rebuild되지 않는 자식 위젯 (선택사항, 성능 최적화용)

### 사용 예제

```dart
class CounterWidget extends StatelessWidget {
  const CounterWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Selector<AppState, int>(
      // counter 값만 선택하여 구독
      selector: (_, state) => state.counter,
      builder: (context, count, child) => Column(
        children: [
          Text('카운터: $count'),
          child!, // rebuild되지 않는 자식 위젯
        ],
      ),
      // 이 버튼은 count가 변경되어도 rebuild되지 않음
      child: ElevatedButton(
        onPressed: () {
          context.read<AppState>().incrementCounter();
        },
        child: const Text('증가'),
      ),
    );
  }
}
```

### 여러 값 선택하기

여러 값을 선택해야 할 때는 `Record`나 `Tuple`을 사용합니다.

```dart
Selector<AppState, (int, String)>(
  selector: (_, state) => (state.counter, state.name),
  builder: (context, data, child) {
    final (count, name) = data;
    return Text('$name: $count');
  },
)
```

## 모범 사례

### 1. Selector 우선 사용

가능한 모든 곳에서 `Selector`를 사용하여 불필요한 rebuild를 방지하세요.

```dart
// ❌ 비권장: 전체 상태 구독
Consumer<AppState>(
  builder: (context, state, child) => Text('${state.counter}'),
)

// ✅ 권장: 필요한 값만 선택적 구독
Selector<AppState, int>(
  selector: (_, state) => state.counter,
  builder: (context, count, child) => Text('$count'),
)
```

### 2. 이벤트 핸들러에서는 read() 사용

이벤트 핸들러에서 상태를 변경할 때는 `read()`를 사용하세요. `watch()`는 build 메서드 내에서만 사용해야 합니다.

```dart
// ❌ 비권장: 이벤트 핸들러에서 watch 사용
onPressed: () {
  context.watch<AppState>().incrementCounter(); // 에러 발생!
}

// ✅ 권장: 이벤트 핸들러에서 read 사용
onPressed: () {
  context.read<AppState>().incrementCounter();
}
```

### 3. child 파라미터 활용

rebuild가 필요 없는 자식 위젯은 `child` 파라미터로 전달하여 성능을 최적화하세요.

```dart
Selector<AppState, int>(
  selector: (_, state) => state.counter,
  builder: (context, count, child) => Column(
    children: [
      Text('$count'), // count 변경 시 rebuild
      child!, // rebuild되지 않음
    ],
  ),
  child: const ExpensiveWidget(), // 비용이 큰 위젯을 child로 전달
)
```

### 4. 상태 클래스는 단일 책임 원칙 준수

하나의 상태 클래스가 너무 많은 책임을 갖지 않도록 분리하세요.

```dart
// ❌ 비권장: 하나의 클래스에 모든 상태
class AppState extends ChangeNotifier {
  User? user;
  List<Post> posts;
  ThemeMode theme;
  // ... 너무 많은 책임
}

// ✅ 권장: 책임별로 분리
class UserState extends ChangeNotifier { /* 사용자 관련 */ }
class PostState extends ChangeNotifier { /* 게시물 관련 */ }
class ThemeState extends ChangeNotifier { /* 테마 관련 */ }
```

## 참고 링크

- [ChangeNotifierProvider 클래스](https://pub.dev/documentation/provider/latest/provider/ChangeNotifierProvider-class.html)
- [Provider 클래스](https://pub.dev/documentation/provider/latest/provider/Provider-class.html)
- [Selector 클래스](https://pub.dev/documentation/provider/latest/provider/Selector-class.html)
