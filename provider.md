# Flutter Provider State Management Reference

Flutter Provider 패키지 6.1.5 이상 버전을 기준으로 작성된 상태관리 필수 가이드라인입니다.

## Table of Contents

- [Overview](#overview)
- [Core Concepts](#core-concepts)
- [Setup](#setup)
- [State Access Methods](#state-access-methods)
- [Selector Widget](#selector-widget)
- [Best Practices](#best-practices)

---

## Overview

Provider는 Flutter에서 가장 널리 사용되는 상태관리 솔루션입니다. InheritedWidget을 기반으로 하여 간편하게 상태를 공유하고 관리할 수 있습니다.

### 필수 규칙

| 규칙 | 설명 |
|------|------|
| Selector 우선 사용 | Consumer 대신 Selector 사용 |
| read() 사용 위치 | 이벤트 핸들러에서만 사용 |
| watch() 사용 금지 | build 메서드 외에서 사용 금지 |

---

## Core Concepts

### ChangeNotifier

상태 변경을 구독자들에게 알릴 수 있는 클래스입니다.

```dart
class AppState extends ChangeNotifier {
  int _counter = 0;

  int get counter => _counter;

  // 상태 변경 후 notifyListeners() 호출
  void incrementCounter() {
    _counter++;
    notifyListeners();  // 내부적으로 setState() 사용
  }
}
```

### ChangeNotifierProvider

ChangeNotifier를 위젯 트리에 제공하고, 자동으로 리소스를 관리(dispose)합니다.

```dart
ChangeNotifierProvider(
  create: (context) => AppState(),
  child: const MyApp(),
)
```

> **중요**: `ChangeNotifier`와 `ChangeNotifierProvider`는 항상 쌍으로 사용합니다.

---

## Setup

### MultiProvider 설정

여러 Provider를 사용할 때는 `main.dart`의 `runApp()`에서 설정합니다.

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

---

## State Access Methods

### context.read<T>()

상태를 **읽기만** 할 때 사용합니다. 상태 변경을 구독하지 않으므로 rebuild가 발생하지 않습니다.

```dart
// 이벤트 핸들러에서 상태 접근 (rebuild 불필요)
ElevatedButton(
  onPressed: () {
    context.read<AppState>().incrementCounter();
  },
  child: Text(T.increase),
)
```

### context.watch<T>() - 사용 제한

build 메서드 내에서만 사용해야 합니다. **이벤트 핸들러에서 사용 금지**.

```dart
// ❌ 절대 금지
onPressed: () {
  context.watch<AppState>().incrementCounter();  // 에러 발생!
}

// ✅ build 메서드에서만 사용 (권장하지 않음 - Selector 사용)
@override
Widget build(BuildContext context) {
  final state = context.watch<AppState>();
  return Text('${state.counter}');
}
```

---

## Selector Widget

### 개념 및 의도

`Selector`는 상태의 특정 부분만 구독하여 **불필요한 rebuild를 방지**하는 위젯입니다.

**필수 규칙**: 가능한 모든 곳에서 `Selector`를 사용합니다.

### 기본 형식

```dart
Selector<모델클래스, 데이터타입>(
  selector: (context, model) => model.값,  // 구독할 값 선택
  builder: (context, 값, child) => Text('$값'),  // 선택한 값으로 UI 빌드
)
```

| 매개변수 | 설명 |
|---------|------|
| `selector` | 모델에서 구독할 특정 값을 반환 (이 값이 변경되면 builder 재실행) |
| `builder` | 선택한 값을 사용하여 위젯을 빌드 |
| `child` | rebuild되지 않는 자식 위젯 (선택사항, 성능 최적화용) |

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
          child!,  // rebuild되지 않는 자식 위젯
        ],
      ),
      // 이 버튼은 count가 변경되어도 rebuild되지 않음
      child: ElevatedButton(
        onPressed: () {
          context.read<AppState>().incrementCounter();
        },
        child: Text(T.increase),
      ),
    );
  }
}
```

### selector 함수의 값 가공

`selector`는 단순히 값을 뽑는 것 외에 **표현식(계산, 변환, 조합)** 결과를 반환할 수 있습니다.
반환된 값이 그대로 `builder`의 두 번째 인자로 전달됩니다.

```dart
/// 문자열 조합 예시
Selector<UserState, String>(
  selector: (_, state) => '${state.firstName} ${state.lastName}',
  builder: (_, fullName, __) => Text(fullName),
)

/// 컬렉션 필터링/계산 예시
Selector<TodoState, int>(
  selector: (_, state) => state.items.where((e) => e.done).length,
  builder: (_, doneCount, __) => Text('완료: $doneCount'),
)
```

### 여러 값 선택하기

Record를 사용하여 여러 값을 묶어 전달합니다.

```dart
Selector<AppState, (int, String)>(
  selector: (_, state) => (state.counter, state.name),
  builder: (context, data, child) {
    final (count, name) = data;
    return Text('$name: $count');
  },
)

/// 조건부 UI 렌더링 예시
Selector<AppState, (int, bool)>(
  selector: (_, state) => (state.count, state.isLoading),
  builder: (_, value, __) {
    final (count, isLoading) = value;
    return isLoading
        ? const CircularProgressIndicator()
        : Text('count: $count');
  },
)
```

### Selector 사용 시 주의사항

Selector는 이전 결과와 새 결과를 `==`로 비교하여 **달라졌을 때만** builder를 재호출합니다.

| 주의 항목 | 설명 | 해결 방법 |
|----------|------|----------|
| 순수 함수 | selector는 부작용 없이 값 계산만 수행 | side effect 코드 제거 |
| 새 인스턴스 생성 | 내용이 같아도 `==`가 false면 리빌드 | primitive 값 반환, `==`/`hashCode` 구현 |
| 무거운 계산 | selector는 자주 호출됨 | 모델에서 미리 계산/캐시 |

```dart
// ❌ 비권장: 매번 새 리스트 생성 → 불필요한 리빌드
Selector<TodoState, List<Todo>>(
  selector: (_, state) => state.items.where((e) => e.done).toList(),
  builder: (_, doneItems, __) => Text('${doneItems.length}'),
)

// ✅ 권장: primitive 값 반환
Selector<TodoState, int>(
  selector: (_, state) => state.items.where((e) => e.done).length,
  builder: (_, doneCount, __) => Text('$doneCount'),
)

// ✅ 권장: 모델에서 미리 계산된 값 사용
class TodoState extends ChangeNotifier {
  List<Todo> _items = [];

  // 캐시된 계산 결과 제공
  int get doneCount => _items.where((e) => e.done).length;
}

Selector<TodoState, int>(
  selector: (_, state) => state.doneCount,  // 이미 계산된 값 사용
  builder: (_, count, __) => Text('$count'),
)
```

### Selector vs Consumer 비교

```dart
// ❌ 비권장: 전체 상태 구독 (불필요한 rebuild 발생)
Consumer<AppState>(
  builder: (context, state, child) => Text('${state.counter}'),
)

// ✅ 권장: 필요한 값만 선택적 구독
Selector<AppState, int>(
  selector: (_, state) => state.counter,
  builder: (context, count, child) => Text('$count'),
)
```

---

## Best Practices

### 1. Selector 우선 사용

```dart
// ✅ 권장
Selector<AppState, int>(
  selector: (_, state) => state.counter,
  builder: (context, count, child) => Text('$count'),
)
```

### 2. 이벤트 핸들러에서는 read() 사용

```dart
// ✅ 권장
onPressed: () {
  context.read<AppState>().incrementCounter();
}
```

### 3. child 파라미터 활용

rebuild가 필요 없는 자식 위젯은 `child` 파라미터로 전달합니다.

```dart
Selector<AppState, int>(
  selector: (_, state) => state.counter,
  builder: (context, count, child) => Column(
    children: [
      Text('$count'),  // count 변경 시 rebuild
      child!,  // rebuild되지 않음
    ],
  ),
  child: const ExpensiveWidget(),  // 비용이 큰 위젯
)
```

### 4. 상태 클래스는 단일 책임 원칙 준수

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

---

## Quick Reference

| 상황 | 사용 방법 |
|------|----------|
| UI에서 상태 표시 | `Selector<T, V>` 사용 |
| 버튼 클릭 시 상태 변경 | `context.read<T>()` 사용 |
| 여러 값 구독 | `Selector<T, (V1, V2)>` Record 사용 |
| 비용 큰 위젯 최적화 | `child` 파라미터 활용 |

## References

- [ChangeNotifierProvider](https://pub.dev/documentation/provider/latest/provider/ChangeNotifierProvider-class.html)
- [Provider](https://pub.dev/documentation/provider/latest/provider/Provider-class.html)
- [Selector](https://pub.dev/documentation/provider/latest/provider/Selector-class.html)
