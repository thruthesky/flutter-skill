---
name: flutter-skill
description: "본 스킬은 Flutter 프레임워크로 앱을 개발하는 데 반드시 따라야 하는 UI/UX 디자인, 상태관리, 네트워킹, API 연동에 관한 가이드라인 제공합니다. 본 스킬은 선택적인 정보를 제공하는 것이 아니라 Flutter 앱 개발에 필수적인 지침을 제공하며 반드시 준수해야 할 사항들을 제공합니다. 개발자가 디자인, UI, UX, 디자인 효과, 상태관리, 라우팅, 네트워킹, API 연동에 관한 요청, 코믹디자인, Comic 관련 요청, 채팅, FCM, 푸시 알림, 메시지, 알림에 관한 요청이 있을 때 반드시 본 스킬을 사용해서 본 스킬이 제공하는 대로 작업을 수행해야 합니다. 각 스킬 문서에의 상단에는 반드시 따라야 할 Workflow 가 있습니다. 반드시 그 Workflow 를 따라야 합니다. 추가 트리거 키워드 - 딥링크, deep link, 캐싱, cache, 아이콘, FontAwesome, Isolate, 동시성, concurrency, 카카오톡, KakaoTalk, Crashlytics, 크래시 리포팅, 공유, share, share_plus, 공유 버튼, 바코드, QR코드, 스캔, scanner, mobile_scanner, 카메라 스캔, barcode, qr, 전화번호 인증, Phone Auth, SMS 인증, verifyPhoneNumber, APNs, AppDelegate, notification-not-forwarded (project)"
---

# Flutter Skill

본 스킬은 Flutter 프레임워크로 앱을 개발하는 데 반드시 따라야 하는 **범용적인** UI/UX 디자인, 상태관리, 네트워킹, API 연동에 관한 가이드라인을 제공합니다. 즉, 본 스킬은 특정 앱에 종속되는 코딩 가이드라인을 제공하는 것이 아니라, 다양한 종류의 플러터 앱 개발에 사용 될 수 있는 범용적인 지침을 제공합니다.

## Table of Contents

- [Workflow](#workflow)
- [Core Principles](#core-principles)
- [Reference Documents](#reference-documents)
- [Quick Reference](#quick-reference)

## Workflow

개발자 요청에 따라 아래 워크플로우를 따릅니다:

1. **디자인/UI/UX 요청 시**: [Comic Design 문서](./references/comic-design.md) 참조
   - 키워드: 디자인, UI, UX, 버튼, 카드, 레이아웃, 애니메이션, Comic, 코믹

2. **레이아웃 요청 시**: [Flutter Layout 문서](./references/flutter-layout.md) 참조
   - 키워드: 레이아웃, 스크롤, CustomScrollView, ListView, 위젯 배치

3. **상태관리 요청 시**: [Provider 문서](./references/provider.md) 참조
   - 키워드: 상태관리, Provider, Selector, ChangeNotifier

4. **라우팅 요청 시**: [Go Route 문서](./references/go_route.md) 참조
   - 키워드: 라우팅, 네비게이션, GoRouter, 페이지 이동

5. **다국어 요청 시**: [Easy Localization 문서](./references/easy_localization.md) 참조
   - 키워드: 다국어, 번역, localization, i18n, easy_localization, tr(), plural()

6. **푸시 알림/FCM 요청 시**: [Firebase FCM 문서](./references/firebase/firebase-fcm.md) 참조
   - 키워드: 푸시 알림, FCM, Firebase Messaging, 알림, notification, 토큰, 구독

7. **Firebase 인증 요청 시**: [Firebase Auth 문서](./references/firebase/firebase-auth.md) 참조
   - 키워드: Firebase Auth, 로그인, 회원가입, 인증, 에러 처리, invalid-credential, user-not-found, wrong-password, Email enumeration protection, 전화번호 인증, Phone Auth, SMS, verifyPhoneNumber, notification-not-forwarded, missing-client-identifier, APNs, AppDelegate, easy_phone_sign_in

8. **크래시 리포팅 요청 시**: [Firebase Crashlytics 문서](./references/firebase/firebase-crashlytics.md) 참조
   - 키워드: Crashlytics, 크래시 리포팅, 에러 추적, 사용자 식별

9. **딥링크 요청 시**: [Deeplink 문서](./references/deeplink.md) 참조
   - 키워드: 딥링크, deep link, Universal Links, App Links

10. **캐싱 요청 시**: [File Cache 문서](./references/file-cache.md) 또는 [Memory Cache 문서](./references/memory-cache.md) 참조
    - 키워드: 캐싱, cache, TTL, 파일 캐시, 메모리 캐시, LRU

11. **아이콘 요청 시**: [Font Awesome 문서](./references/fontawesome.md) 참조
    - 키워드: 아이콘, FontAwesome, FaIcon

12. **동시성/Isolate 요청 시**: [Concurrency 문서](./references/concurrency-and-isolates.md) 참조
    - 키워드: Isolate, 동시성, concurrency, UI Jank, compute

13. **카카오톡 연동 요청 시**: [KakaoTalk 문서](./references/kakaotalk-friend-add.md) 참조
    - 키워드: 카카오톡, KakaoTalk, 오픈채팅, 1:1 채팅 링크

14. **공유 기능 요청 시**: [Share Plus 문서](./references/share-plus.md) 참조
    - 키워드: 공유, share, 공유 버튼, share_plus, SharePlus, 외부 앱 공유, Share Sheet

15. **바코드/QR코드 스캔 요청 시**: [Mobile Scanner 문서](./references/mobile-scanner.md) 참조
    - 키워드: 바코드, QR코드, 스캔, scanner, mobile_scanner, 카메라 스캔, barcode, qr

## Core Principles

### 필수 규칙

```dart
// ❌ 절대 금지
color: Colors.blue              // 하드코딩된 색상
fontSize: 16                    // 하드코딩된 크기
Text('Hello')                   // 하드코딩된 텍스트
elevation: 4                    // 0이 아닌 elevation

// ✅ 반드시 사용
color: Theme.of(context).colorScheme.primary  // 테마 색상
style: Theme.of(context).textTheme.bodyLarge  // 테마 텍스트 스타일
Text(T.hello)                                  // i18n 번역
elevation: 0                                   // 플랫 디자인
```

### 테마 기반 스타일링

모든 색상, 폰트, 스타일은 반드시 `Theme.of(context)`를 사용합니다:

| 용도 | 사용 방법 |
|------|----------|
| Primary 색상 | `Theme.of(context).colorScheme.primary` |
| Surface 색상 | `Theme.of(context).colorScheme.surface` |
| 테두리 색상 | `Theme.of(context).colorScheme.outline` |
| 본문 텍스트 | `Theme.of(context).textTheme.bodyLarge` |
| 제목 텍스트 | `Theme.of(context).textTheme.titleMedium` |

### Comic 디자인 규칙 요약

| 속성 | 값 | 설명 |
|------|-----|------|
| Border Width | `2.0` (표준), `1.0` (목록) | Comic 스타일 테두리 |
| Border Radius | `12` | 둥근 모서리 |
| Elevation | `0` | 그림자 없음 |
| 간격 | 8의 배수 | 8, 16, 24, 32... |

## Reference Documents

각 문서는 해당 주제에 대한 상세한 가이드라인과 코드 예제를 제공합니다:

### 디자인 & UI

| 문서 | 내용 |
|------|------|
| [comic-design.md](./references/comic-design.md) | Comic UI 디자인 시스템, 버튼, 카드, 폼, SnackBar 등 |
| [flutter-layout.md](./references/flutter-layout.md) | 스크롤 화면, CustomScrollView, ListView 패턴 |
| [fontawesome.md](./references/fontawesome.md) | Font Awesome Pro 아이콘 사용법, 라이센스 정보 |

### 상태관리 & 라우팅

| 문서 | 내용 |
|------|------|
| [provider.md](./references/provider.md) | Provider 상태관리, Selector, ChangeNotifier |
| [go_route.md](./references/go_route.md) | GoRouter 라우팅, 파라미터 전달, redirect |
| [deeplink.md](./references/deeplink.md) | 딥링크 구현, Universal Links, App Links 설정 |

### 다국어 & 캐싱

| 문서 | 내용 |
|------|------|
| [easy_localization.md](./references/easy_localization.md) | easy_localization 패키지 기반 다국어 지원 (tr(), plural(), 성별, 코드 생성) |
| [file-cache.md](./references/file-cache.md) | file_cache_flutter 패키지, 메모리+파일 이중 캐싱, TTL |
| [memory-cache.md](./references/memory-cache.md) | LRU 메모리 캐시 서비스 (MemoryCache 클래스) |

### Firebase

| 문서 | 내용 |
|------|------|
| [firebase-fcm.md](./references/firebase/firebase-fcm.md) | FCM 푸시 알림, 토큰 관리, 메시지 핸들링 |
| [firebase-auth.md](./references/firebase/firebase-auth.md) | Firebase 인증, Email enumeration protection, 에러 코드 처리 (`invalid-credential`, `user-not-found`, `wrong-password` 등) |
| [firebase-crashlytics.md](./references/firebase/firebase-crashlytics.md) | Crashlytics 설정, 크래시 리포팅, 사용자 추적 |

### 기타

| 문서 | 내용 |
|------|------|
| [concurrency-and-isolates.md](./references/concurrency-and-isolates.md) | Dart Isolate, UI Jank 방지, 비동기 처리 |
| [kakaotalk-friend-add.md](./references/kakaotalk-friend-add.md) | 카카오톡 오픈채팅 연동, 1:1 채팅 링크 처리 |
| [share-plus.md](./references/share-plus.md) | share_plus 패키지, 텍스트/파일/URI 공유, iPad 호환, 게시글 공유 패턴 |
| [mobile-scanner.md](./references/mobile-scanner.md) | mobile_scanner 패키지, 바코드/QR코드 실시간 스캔, 이미지 분석, 카메라 제어 |

## 필수 pub.dev 패키지

### file_cache_flutter

Flutter 애플리케이션용 파일 캐시 라이브러리로, 메모리 + 파일 이중 캐싱과 TTL(Time-To-Live) 기반 자동 만료를 지원합니다.

- **pub.dev**: [file_cache_flutter](https://pub.dev/packages/file_cache_flutter)
- **상세 문서**: [file-cache.md](./references/file-cache.md)

#### 주요 기능

| 기능 | 설명 |
|------|------|
| 이중 캐싱 | 메모리와 파일 캐시 동시 활용으로 빠른 접근 |
| TTL 지원 | 캐시 만료 시간 설정 가능 (기본값 30분) |
| 제네릭 타입 | 모든 데이터 타입 캐싱 가능 |
| 자동 정리 | 만료된 캐시 자동 삭제 |

#### 설치

```yaml
dependencies:
  file_cache_flutter: ^0.0.3
```

#### 사용 예제

```dart
import 'package:file_cache_flutter/file_cache_flutter.dart';

// 캐시 인스턴스 생성
final cache = FileCache<UserData>(
  cacheName: 'user_data',           // 캐시 이름 (파일 저장 경로에 사용)
  fromJson: UserData.fromJson,      // JSON → 객체 변환 함수
  toJson: (data) => data.toJson(),  // 객체 → JSON 변환 함수
  defaultTtl: Duration(minutes: 30), // 기본 TTL 설정
);

// 데이터 저장
await cache.set('user_123', UserData(name: '홍길동', age: 25));

// 데이터 조회 (만료되지 않은 경우에만 반환)
final user = await cache.get('user_123');

// 캐시 존재 여부 확인
final exists = await cache.has('user_123');

// 특정 키 삭제
await cache.remove('user_123');

// 전체 캐시 삭제
await cache.clear();

// 만료된 캐시만 정리
await cache.cleanup();
```

#### 주요 메서드

| 메서드 | 설명 |
|--------|------|
| `get(key)` | 캐시에서 데이터 조회 (만료 시 null 반환) |
| `set(key, data, [ttl])` | 캐시에 데이터 저장 (선택적 TTL 지정) |
| `has(key)` | 캐시 존재 여부 확인 |
| `remove(key)` | 특정 키의 캐시 삭제 |
| `clear()` | 전체 캐시 삭제 |
| `cleanup()` | 만료된 캐시 정리 |

## Quick Reference

### Import 문

```dart
import 'package:flutter/material.dart';
import 'package:font_awesome_flutter/font_awesome_flutter.dart';
import 'package:flutter_animate/flutter_animate.dart';
import 'package:provider/provider.dart';
import 'package:go_router/go_router.dart';
```

### 애니메이션 (flutter_animate 패키지 사용)

```dart
Container()
  .animate()
  .fadeIn(duration: 300.ms)
  .slideX(begin: -0.2, end: 0)
```

### 아이콘 우선순위

Font Awesome Pro 아이콘 사용 시 우선순위: **Light > Regular > Solid**

```dart
FaIcon(FontAwesomeIcons.lightCamera)  // Light 우선
```
