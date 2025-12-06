# GoRouter Reference

GoRouter 라우팅 패키지를 사용한 Flutter 네비게이션 가이드라인입니다.

## Table of Contents

- [Overview](#overview)
- [Parameter Passing](#parameter-passing)
- [Navigation Methods](#navigation-methods)
- [Redirect and Guards](#redirect-and-guards)
- [Advanced Usage](#advanced-usage)
- [Best Practices](#best-practices)

---

## Overview

GoRouter는 웹에서도 사용 가능한 declarative routing을 지향하는 Flutter 라우팅 패키지입니다.

### 기본 설정

```dart
final GlobalKey<NavigatorState> globalNavigatorKey = GlobalKey();
BuildContext get globalContext => globalNavigatorKey.currentContext!;

final router = GoRouter(
  navigatorKey: globalNavigatorKey,
  redirect: (context, state) {
    if (state.fullPath == EntryScreen.routeName) {
      return null;
    }
    if (FirebaseAuth.instance.currentUser == null) {
      return EntryScreen.routeName;
    }
    return null;
  },
  routes: [
    GoRoute(
      path: HomeScreen.routeName,
      name: HomeScreen.routeName,
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: EntryScreen.routeName,
      name: EntryScreen.routeName,
      pageBuilder: (context, state) => NoTransitionPage<void>(
        key: state.pageKey,
        child: const EntryScreen(),
      ),
    ),
  ],
);
```

---

## Parameter Passing

GoRouter에서 파라미터를 전달하는 세 가지 방법입니다.

### 1. Path Parameters (pathParameters)

URL의 named parameter로 전달합니다.

**Route 정의:**

```dart
GoRoute(
  path: '/sample/:id1/:id2',
  name: 'sample',
  builder: (context, state) => SampleWidget(
    id1: state.pathParameters['id1'],
    id2: state.pathParameters['id2'],
  ),
)
```

**호출:**

```dart
ElevatedButton(
  onPressed: () {
    context.goNamed(
      'sample',
      pathParameters: {'id1': 'param1', 'id2': 'param2'},
    );
  },
  child: Text(T.navigate),
)
```

**위젯에서 받기:**

```dart
class SampleWidget extends StatelessWidget {
  final String? id1;
  final String? id2;

  const SampleWidget({super.key, this.id1, this.id2});

  @override
  Widget build(BuildContext context) {
    return Text('id1: $id1, id2: $id2');
  }
}
```

### 2. Query Parameters (queryParameters)

URL의 쿼리 스트링으로 전달합니다. **문자열만 가능**.

**Route 정의:**

```dart
GoRoute(
  name: 'sample',
  path: '/sample',
  builder: (context, state) => SampleWidget(
    id1: state.uri.queryParameters['id1'],
    id2: state.uri.queryParameters['id2'],
  ),
)
```

**호출:**

```dart
ElevatedButton(
  onPressed: () {
    context.goNamed(
      'sample',
      queryParameters: {'id1': 'param1', 'id2': 'param2'},
    );
  },
  child: Text(T.navigate),
)
```

### 3. Extra (객체 전달)

객체를 직접 전달합니다. **웹 URL에는 반영되지 않음**.

**Route 정의:**

```dart
GoRoute(
  path: '/family',
  builder: (context, state) => FamilyScreen(
    family: state.extra! as Family,
  ),
)
```

**호출:**

```dart
ElevatedButton(
  onPressed: () {
    context.go('/family', extra: myFamilyObject);
  },
  child: Text(T.navigate),
)
```

### 파라미터 전달 비교

| 방법 | 타입 | 웹 URL 반영 | 사용 사례 |
|------|------|------------|----------|
| `pathParameters` | String만 | O | ID, 슬러그 |
| `queryParameters` | String만 | O | 필터, 검색어 |
| `extra` | Any Object | X | 복잡한 객체 |

---

## Navigation Methods

### context.go()

현재 스택을 교체합니다 (웹의 location 변경과 유사).

```dart
context.go('/home');
context.go('/profile/123');
context.go('/search', extra: searchParams);
```

### context.goNamed()

이름으로 네비게이션합니다.

```dart
context.goNamed('profile', pathParameters: {'id': '123'});
context.goNamed('search', queryParameters: {'q': 'flutter'});
```

### context.push()

스택에 새 페이지를 추가합니다 (뒤로 가기 가능).

```dart
context.push('/detail/123');
context.pushNamed('detail', pathParameters: {'id': '123'});
```

### context.pop()

이전 페이지로 돌아갑니다.

```dart
// 단순 뒤로 가기
context.pop();

// 값 반환하며 뒤로 가기 (Navigator 사용)
Navigator.of(context).pop(resultValue);
```

### 값 반환받기

```dart
// push로 이동하고 결과 받기
final result = await context.push<String>('/select-item');
if (result != null) {
  // 결과 처리
}

// 결과 반환하기
context.pop('selected_value');
```

---

## Redirect and Guards

### 로그인 상태에 따른 redirect

```dart
GoRouter(
  redirect: (context, state) {
    final isLoggedIn = FirebaseAuth.instance.currentUser != null;
    final isLoginRoute = state.matchedLocation == '/login';

    // 로그인 안 된 상태에서 보호된 페이지 접근 시
    if (!isLoggedIn && !isLoginRoute) {
      return '/login';
    }

    // 로그인된 상태에서 로그인 페이지 접근 시
    if (isLoggedIn && isLoginRoute) {
      return '/home';
    }

    return null;  // redirect 없음
  },
  routes: [...],
)
```

### 특정 경로만 보호

```dart
GoRouter(
  redirect: (context, state) {
    final isLoggedIn = authState.isLoggedIn;
    final protectedPaths = ['/profile', '/settings', '/orders'];

    final isProtectedRoute = protectedPaths.any(
      (path) => state.matchedLocation.startsWith(path),
    );

    if (!isLoggedIn && isProtectedRoute) {
      return '/login?redirect=${state.matchedLocation}';
    }

    return null;
  },
)
```

### refreshListenable로 상태 변경 감지

```dart
GoRouter(
  refreshListenable: authState,  // ChangeNotifier
  redirect: (context, state) {
    // authState가 notifyListeners() 호출 시 redirect 재평가
    if (!authState.isLoggedIn) {
      return '/login';
    }
    return null;
  },
)
```

---

## Advanced Usage

### URL에서 Query Parameters 얻기

```dart
final uri = Uri.parse(GoRouterState.of(context).uri.toString());
print('queryParameters: ${uri.queryParameters}');
print('id: ${uri.queryParameters['id']}');
```

### Global Navigator Key 얻기

```dart
// 라우터에서 context 얻기
final context = router.routerDelegate.navigatorKey.currentContext!;

// 글로벌 서비스에서 사용
class _RootAppState extends State<RootApp> {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addPostFrameCallback((timeStamp) {
      AppService.instance.init(
        context: router.routerDelegate.navigatorKey.currentContext!,
      );
    });
  }
}
```

### Navigator Builder (모든 페이지 감싸기)

```dart
GoRouter(
  navigatorBuilder: (context, state, child) {
    return DynamicLinksHandler(child: child);
  },
  routes: [...],
)
```

모든 페이지를 특정 위젯으로 감쌀 때 사용합니다 (예: Dynamic Links 처리).

```dart
class DynamicLinksHandler extends StatefulWidget {
  const DynamicLinksHandler({super.key, required this.child});

  final Widget child;

  @override
  State<DynamicLinksHandler> createState() => _DynamicLinksHandlerState();
}

class _DynamicLinksHandlerState extends State<DynamicLinksHandler> {
  @override
  Widget build(BuildContext context) {
    return widget.child;
  }
}
```

### ShellRoute (공통 UI 감싸기)

```dart
GoRouter(
  routes: [
    ShellRoute(
      builder: (context, state, child) {
        return ScaffoldWithNavBar(child: child);
      },
      routes: [
        GoRoute(path: '/home', builder: ...),
        GoRoute(path: '/search', builder: ...),
        GoRoute(path: '/profile', builder: ...),
      ],
    ),
  ],
)
```

바텀 네비게이션 바 등 공통 UI를 유지하면서 자식 라우트만 변경할 때 사용합니다.

### NoTransitionPage (전환 애니메이션 없이)

```dart
GoRoute(
  path: '/entry',
  name: 'entry',
  pageBuilder: (context, state) => NoTransitionPage<void>(
    key: state.pageKey,
    child: const EntryScreen(),
  ),
)
```

---

## Screen Template

GoRouter와 함께 사용하는 페이지(Screen)의 기본 구조입니다.

### 기본 구조

각 Screen에 `routeName`, `push`, `go` static 멤버를 정의하여 라우팅을 직관적으로 만듭니다.

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

class NameScreen extends StatefulWidget {
  // 라우트 이름: Screen 이름에서 "/Name" 형태로 지정
  static const String routeName = '/Name';

  // push: 스택에 추가 (뒤로가기 가능)
  static Function(BuildContext ctx) push = (ctx) => ctx.push(routeName);

  // go: 스택 교체
  static Function(BuildContext ctx) go = (ctx) => ctx.go(routeName);

  const NameScreen({super.key});

  @override
  State<NameScreen> createState() => _NameScreenState();
}

class _NameScreenState extends State<NameScreen> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Name'),
      ),
      body: const Column(
        children: [
          Text("Name"),
        ],
      ),
    );
  }
}
```

### 사용법

Screen 클래스에서 직접 `.push()` 또는 `.go()`를 호출합니다.

```dart
// push로 이동 (뒤로가기 가능)
NameScreen.push(context);

// go로 이동 (스택 교체)
NameScreen.go(context);
```

### 장점

- **직관적**: `NameScreen.push(context)`로 해당 페이지로 이동하는 것이 명확함
- **타입 안전**: 컴파일 타임에 라우트 존재 여부 확인
- **유지보수 용이**: 라우트 변경 시 Screen 클래스만 수정

### 동적 파라미터가 있는 경우

```dart
class ProfileScreen extends StatefulWidget {
  // 동적 파라미터가 포함된 라우트
  static const String routeName = '/Profile/:id';

  // 파라미터를 받아서 라우트 생성
  static void push(BuildContext ctx, String id) =>
      ctx.push(routeName.replaceFirst(':id', id));

  static void go(BuildContext ctx, String id) =>
      ctx.go(routeName.replaceFirst(':id', id));

  final String id;
  const ProfileScreen({super.key, required this.id});

  @override
  State<ProfileScreen> createState() => _ProfileScreenState();
}

// 사용
ProfileScreen.push(context, '123');
ProfileScreen.go(context, 'user-456');
```

### GoRouter 설정에 등록

```dart
final router = GoRouter(
  routes: [
    GoRoute(
      path: NameScreen.routeName,
      name: NameScreen.routeName,
      builder: (context, state) => const NameScreen(),
    ),
    GoRoute(
      path: ProfileScreen.routeName,
      name: ProfileScreen.routeName,
      builder: (context, state) => ProfileScreen(
        id: state.pathParameters['id']!,
      ),
    ),
  ],
);
```

---

## Best Practices

### 1. Route 이름 상수화

```dart
class AppRoutes {
  static const home = 'home';
  static const profile = 'profile';
  static const settings = 'settings';

  static const homePath = '/';
  static const profilePath = '/profile/:id';
  static const settingsPath = '/settings';
}

// 사용
context.goNamed(AppRoutes.profile, pathParameters: {'id': userId});
```

### 2. Screen에서 Route 정보 정의

```dart
class ProfileScreen extends StatelessWidget {
  static const routeName = '/profile';

  static GoRoute route = GoRoute(
    path: routeName,
    name: routeName,
    builder: (context, state) => const ProfileScreen(),
  );

  const ProfileScreen({super.key});

  @override
  Widget build(BuildContext context) {...}
}

// 라우터에서 사용
GoRouter(
  routes: [
    ProfileScreen.route,
    SettingsScreen.route,
  ],
)
```

### 3. Type-safe 네비게이션

```dart
extension ProfileNavigation on BuildContext {
  void goToProfile(String userId) {
    goNamed('profile', pathParameters: {'id': userId});
  }

  void goToSearch({String? query, String? category}) {
    goNamed(
      'search',
      queryParameters: {
        if (query != null) 'q': query,
        if (category != null) 'category': category,
      },
    );
  }
}

// 사용
context.goToProfile('123');
context.goToSearch(query: 'flutter', category: 'tutorial');
```

### 4. 에러 페이지 처리

```dart
GoRouter(
  errorBuilder: (context, state) => ErrorScreen(
    error: state.error,
  ),
  routes: [...],
)
```

---

## Quick Reference

| 작업 | 코드 |
|------|------|
| 페이지 이동 (스택 교체) | `context.go('/path')` |
| 이름으로 이동 | `context.goNamed('name')` |
| 페이지 추가 (뒤로 가기 가능) | `context.push('/path')` |
| 뒤로 가기 | `context.pop()` |
| 값과 함께 뒤로 가기 | `Navigator.of(context).pop(value)` |
| Path 파라미터 | `pathParameters: {'id': '123'}` |
| Query 파라미터 | `queryParameters: {'q': 'search'}` |
| 객체 전달 | `extra: myObject` |
| 파라미터 읽기 | `state.pathParameters['id']` |
| 쿼리 읽기 | `state.uri.queryParameters['q']` |
| Extra 읽기 | `state.extra as MyType` |

## References

- [go_router package](https://pub.dev/packages/go_router)
- [GoRouter Documentation](https://pub.dev/documentation/go_router/latest/)
