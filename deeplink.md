# Flutter 딥링크 (Deep Link)

## 목차

1. [딥링크 개요](#딥링크-개요)
2. [GoRouter를 활용한 딥링크 처리](#gorouter를-활용한-딥링크-처리)
3. [redirect에서 데이터 전달 방법](#redirect에서-데이터-전달-방법)
4. [Apple Universal Links 설정](#apple-universal-links-설정)
5. [Android App Links 설정](#android-app-links-설정)
6. [테스트 방법](#테스트-방법)

---

## 딥링크 개요

딥링크(Deep Link)는 앱의 특정 화면으로 직접 이동할 수 있는 URL입니다.

### 딥링크 유형

| 유형 | 설명 | 예시 |
|------|------|------|
| **URL Scheme** | 커스텀 스킴 사용 | `myapp://product/123` |
| **Universal Links (iOS)** | HTTPS URL 사용 | `https://example.com/product/123` |
| **App Links (Android)** | HTTPS URL 사용 | `https://example.com/product/123` |

### 딥링크 처리 흐름

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          딥링크 URL 수신                                  │
│                 예: https://example.com/product/123                      │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    GoRouter redirect 함수                                │
│                    - URL 파싱 및 유형 판별                                │
│                    - 상태 저장 (NavigationState)                         │
│                    - 적절한 경로로 리다이렉트                              │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    GoRoute builder                                       │
│                    - 상태에서 데이터 읽기                                  │
│                    - 화면 렌더링                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## GoRouter를 활용한 딥링크 처리

### 기본 구조

```dart
final router = GoRouter(
  redirect: (context, state) {
    // 1. URL 파싱
    final uri = state.uri;
    final path = uri.path;
    final queryParams = uri.queryParameters;

    // 2. URL 유형에 따른 처리
    if (path.contains('/product/')) {
      // 상태에 데이터 저장
      MyState.of(context).productId = queryParams['id'];
      return '/product-view';
    }

    // 3. 인증 체크 등 기타 처리
    return null;
  },
  routes: [
    GoRoute(
      path: '/product-view',
      builder: (context, state) {
        // 상태에서 데이터 읽기
        final productId = MyState.of(context).productId;
        return ProductViewScreen(productId: productId);
      },
    ),
  ],
);
```

### redirect 함수의 핵심 개념

```dart
redirect: (context, state) {
  // state.uri: 전체 URI 객체
  // state.uri.path: 경로 (예: /product/view)
  // state.uri.queryParameters: 쿼리 파라미터 Map
  // state.matchedLocation: 매칭된 경로
  // state.fullPath: 전체 경로

  // 리턴 값:
  // - null: 리다이렉트 없음, 원래 경로로 진행
  // - String: 해당 경로로 리다이렉트
}
```

---

## redirect에서 데이터 전달 방법

### 문제점

GoRouter의 `redirect`에서는 **경로 문자열만 반환** 가능합니다.
`state.extra`를 통한 데이터 전달은 **불가능**합니다.

```dart
// ❌ 불가능: redirect에서 extra 전달
redirect: (context, state) {
  // extra를 전달할 방법이 없음
  return '/product-view';  // 경로만 반환 가능
}
```

### 해결 방법

| 방법 | 설명 | 장단점 |
|------|------|--------|
| **Query Parameter** | URL에 파라미터 추가 | 간단, URL에 노출됨 |
| **Path Parameter** | 경로에 파라미터 포함 | 깔끔한 URL, route 정의 변경 필요 |
| **외부 상태 관리** | Provider/State 사용 | 유연함, 복잡한 객체 전달 가능 |

### 방법 1: Query Parameter

```dart
// redirect에서
redirect: (context, state) {
  final productId = state.uri.queryParameters['id'];
  return '/product-view?id=$productId';
}

// builder에서
builder: (context, state) {
  final id = state.uri.queryParameters['id'];
  return ProductViewScreen(productId: id);
}
```

### 방법 2: Path Parameter

```dart
// route 정의
GoRoute(
  path: '/product-view/:id',
  builder: (context, state) {
    final id = state.pathParameters['id'];
    return ProductViewScreen(productId: id);
  },
)

// redirect에서
return '/product-view/$productId';
```

### 방법 3: 외부 상태 관리 (권장)

```dart
// 상태 클래스
class NavigationState extends ChangeNotifier {
  Product? product;
  String? initialCategoryId;
  // ...
}

// redirect에서 상태 저장
redirect: (context, state) {
  if (isProductView) {
    NavigationState.of(context, listen: false).product = Product(
      id: queryParams['id'],
    );
    return '/product-view';
  }
  return null;
}

// builder에서 상태 읽기
builder: (context, state) {
  Product? product;

  // 1. 일반 네비게이션: state.extra에서 가져오기
  if (state.extra != null) {
    product = state.extra as Product;
  }

  // 2. 딥링크: 상태에서 가져오기
  if (product == null) {
    final navState = NavigationState.of(context, listen: false);
    if (navState.product != null) {
      product = navState.product;
      navState.product = null;  // 사용 후 초기화
    }
  }

  // 3. 데이터 없으면 홈으로
  if (product == null) {
    WidgetsBinding.instance.addPostFrameCallback((_) {
      context.go('/');
    });
    return const Scaffold(body: Center(child: CircularProgressIndicator()));
  }

  return ProductViewScreen(product: product);
}
```

---

## Apple Universal Links 설정

### 1. Apple App Site Association (AASA) 파일

서버의 `/.well-known/apple-app-site-association`에 JSON 파일 배치:

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appIDs": [
          "TEAM_ID.com.example.app"
        ],
        "paths": [
          "*"
        ],
        "components": [
          {
            "/": "/*"
          }
        ]
      }
    ]
  },
  "webcredentials": {
    "apps": [
      "TEAM_ID.com.example.app"
    ]
  }
}
```

### 2. Xcode 설정

1. **Signing & Capabilities** 탭에서 **Associated Domains** 추가
2. 도메인 추가: `applinks:example.com`

### 3. 설정 확인

```sh
curl "https://example.com/.well-known/apple-app-site-association"
```

---

## Android App Links 설정

### 1. assetlinks.json 파일

서버의 `/.well-known/assetlinks.json`에 JSON 파일 배치:

```json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.example.app",
      "sha256_cert_fingerprints": [
        "SHA256_FINGERPRINT"
      ]
    }
  }
]
```

### 2. AndroidManifest.xml 설정

```xml
<activity ...>
  <intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data
      android:scheme="https"
      android:host="example.com"
      android:pathPattern=".*" />
  </intent-filter>
</activity>
```

---

## 테스트 방법

### iOS 시뮬레이터

```sh
# 딥링크 테스트
xcrun simctl openurl booted "https://example.com/product/view?id=123"

# 여러 패턴 테스트
xcrun simctl openurl booted "https://example.com/category/list?type=electronics"
xcrun simctl openurl booted "https://example.com/chat/room?id=abc123"
```

### Android 에뮬레이터

```sh
# 딥링크 테스트
adb shell am start -a android.intent.action.VIEW \
  -d "https://example.com/product/view?id=123" \
  com.example.app
```

### 디버깅 팁

```dart
redirect: (context, state) {
  // 로그 출력으로 딥링크 데이터 확인
  developer.log(
    'Redirect: path=${state.fullPath}, '
    'matchedLocation=${state.matchedLocation}, '
    'uri=${state.uri}',
    name: 'Router',
  );

  // ...
}
```

---

## 일반적인 문제 해결

### 1. redirect가 여러 번 호출됨

redirect 후 새 경로에 대해 다시 redirect가 호출될 수 있음.
조건문을 명확히 하여 무한 루프 방지.

```dart
redirect: (context, state) {
  // 이미 처리된 경로면 null 반환
  if (state.fullPath == '/product-view') {
    return null;
  }
  // ...
}
```

### 2. 딥링크 후 홈으로 돌아감

builder에서 `state.extra`가 null일 때 홈으로 이동시키는 코드가 있으면,
딥링크에서는 extra가 없으므로 항상 홈으로 이동함.

**해결**: 외부 상태에서도 데이터를 확인하도록 수정.

### 3. 인증이 필요한 화면

딥링크로 접근 시 인증되지 않은 상태일 수 있음.

```dart
redirect: (context, state) {
  final user = FirebaseAuth.instance.currentUser;

  // 인증 필요한 경로인데 미인증 상태
  if (requiresAuth(state.fullPath) && user == null) {
    // 딥링크 정보 저장 후 로그인 화면으로
    NavigationState.of(context).pendingDeepLink = state.uri.toString();
    return '/login';
  }

  return null;
}
```
