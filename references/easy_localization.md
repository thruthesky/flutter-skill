# Easy Localization Reference

Flutter 앱 다국어(i18n) 지원을 위한 `easy_localization` 패키지 가이드입니다.

## 목차

- [현재 프로젝트 적용 방식 (필수)](#현재-프로젝트-적용-방식-필수)
- [설치 및 설정](#설치-및-설정)
- [번역 파일 구조](#번역-파일-구조)
- [기본 사용법](#기본-사용법)
- [인자 전달](#인자-전달)
- [복수형 처리](#복수형-처리)
- [성별 처리](#성별-처리)
- [연결 번역](#연결-번역)
- [로케일 관리](#로케일-관리)
- [코드 생성](#코드-생성)
- [Asset Loaders](#asset-loaders)
- [iOS 설정](#ios-설정)
- [로거 설정](#로거-설정)
- [유틸리티](#유틸리티)

---

## 현재 프로젝트 적용 방식 (필수)

> **본 프로젝트는 JSON 파일을 사용하지 않고, Dart Map 기반 커스텀 AssetLoader를 사용합니다.**
> **번역 키는 한국어 원문 텍스트 자체를 사용합니다. (영어 키 사용 금지)**

### 아키텍처 개요

```
lib/l10n/
├── code_asset_loader.dart   ← 커스텀 AssetLoader (AssetLoader 구현)
└── translations.dart        ← 번역 Map 데이터 (ko, en)
```

- **JSON/CSV/YAML 파일 불필요** — 번역 텍스트를 Dart 코드에서 직접 관리
- **키 = 한국어 원문** — `'로그인'.tr()` 처럼 한국어 텍스트가 곧 번역 키
- **플랫 구조** — 중첩 Map 사용하지 않음 (모든 키가 최상위)

### 파일 구조

#### `lib/l10n/code_asset_loader.dart`

```dart
import 'package:flutter/material.dart';
import 'package:easy_localization/easy_localization.dart';
import 'translations.dart';

class CodeAssetLoader extends AssetLoader {
  const CodeAssetLoader();

  @override
  Future<Map<String, dynamic>> load(String path, Locale locale) async {
    switch (locale.languageCode) {
      case 'ko':
        return Translations.ko;
      case 'en':
        return Translations.en;
      default:
        return Translations.en;
    }
  }
}
```

#### `lib/l10n/translations.dart`

```dart
class Translations {
  Translations._();

  static const Map<String, dynamic> ko = {
    // 공통
    '로그인': '로그인',
    '로그아웃': '로그아웃',

    // 하단 네비게이션
    '홈': '홈',
    '게시판': '게시판',
    '채팅': '채팅',

    // 인자가 있는 번역
    '다음 레벨까지 {}%': '다음 레벨까지 {}%',
    '{}개': '{}개',
    '{}개 게시물': '{}개 게시물',
  };

  static const Map<String, dynamic> en = {
    // 공통
    '로그인': 'Login',
    '로그아웃': 'Logout',

    // 하단 네비게이션
    '홈': 'Home',
    '게시판': 'Forum',
    '채팅': 'Chat',

    // 인자가 있는 번역
    '다음 레벨까지 {}%': '{}% to next level',
    '{}개': '{}',
    '{}개 게시물': '{} posts',
  };
}
```

### main.dart 설정

```dart
import 'package:easy_localization/easy_localization.dart';
import 'package:philgo/l10n/code_asset_loader.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await EasyLocalization.ensureInitialized();

  runApp(
    EasyLocalization(
      supportedLocales: const [Locale('ko'), Locale('en')],
      path: 'unused',  // CodeAssetLoader에서 path를 사용하지 않지만 필수 파라미터
      assetLoader: const CodeAssetLoader(),
      fallbackLocale: const Locale('ko'),
      child: MultiProvider(
        providers: [...],
        child: const MyApp(),
      ),
    ),
  );
}
```

```dart
// MaterialApp에 localization 연결
MaterialApp.router(
  localizationsDelegates: context.localizationDelegates,
  supportedLocales: context.supportedLocales,
  locale: context.locale,
  routerConfig: router,
)
```

### 위젯에서 사용

```dart
import 'package:easy_localization/easy_localization.dart';

// 기본 번역 — 한국어 원문이 곧 키
Text('로그인'.tr())
Text('로그아웃'.tr())
Text('홈'.tr())

// 인자가 있는 번역
Text('다음 레벨까지 {}%'.tr(args: ['${user.levelProgress}']))
Text('{}개'.tr(args: ['${user.noOfPost}']))
Text('{}개 게시물'.tr(args: ['${user.noOfPost}']))

// 조건부 번역
Text(user.gender == 'M' ? '남성'.tr() : '여성'.tr())
```

### 새 번역 추가 방법

1. `lib/l10n/translations.dart`의 `ko` Map과 `en` Map에 동일한 한국어 키를 추가
2. `ko` Map에는 한국어 값, `en` Map에는 영어 값 설정
3. 위젯에서 `'한국어키'.tr()` 호출

```dart
// 1. translations.dart에 추가
static const Map<String, dynamic> ko = {
  '새 번역': '새 번역',
  '{}님 환영합니다': '{}님 환영합니다',
};
static const Map<String, dynamic> en = {
  '새 번역': 'New Translation',
  '{}님 환영합니다': 'Welcome, {}',
};

// 2. 위젯에서 사용
Text('새 번역'.tr())
Text('{}님 환영합니다'.tr(args: [userName]))
```

### 핵심 규칙

| 규칙 | 설명 |
|------|------|
| 키는 한국어 | `'login'.tr()` (X) → `'로그인'.tr()` (O) |
| 플랫 구조 | `'menu.login'.tr()` (X) → `'로그인'.tr()` (O) |
| ko/en 양쪽 추가 | 번역 누락 방지를 위해 반드시 두 Map 모두에 키 추가 |
| JSON 파일 불필요 | Dart 코드에서 직접 관리, 별도 에셋 파일 없음 |

---

## 설치 및 설정

### pubspec.yaml

```yaml
dependencies:
  easy_localization: ^3.0.8
  # 커스텀 로더 사용 시 (CSV, YAML, XML, HTTP 등)
  # easy_localization_loader: <latest_version>

flutter:
  assets:
    - assets/translations/
```

### 번역 파일 디렉토리 구조

```
assets/
└── translations/
    ├── en.json          # useOnlyLangCode: true 인 경우
    ├── ko.json
    ├── en-US.json       # useOnlyLangCode: false (기본값)
    └── ko-KR.json
```

### main.dart 초기화

```dart
import 'package:easy_localization/easy_localization.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await EasyLocalization.ensureInitialized();

  runApp(
    EasyLocalization(
      supportedLocales: [Locale('en', 'US'), Locale('ko', 'KR')],
      path: 'assets/translations',
      fallbackLocale: Locale('en', 'US'),
      child: MyApp(),
    ),
  );
}
```

### MaterialApp 연결

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      localizationsDelegates: context.localizationDelegates,
      supportedLocales: context.supportedLocales,
      locale: context.locale,
      home: MyHomePage(),
    );
  }
}
```

### EasyLocalization 위젯 속성

| 속성 | 필수 | 기본값 | 설명 |
|------|------|--------|------|
| `child` | O | — | 메인 앱 위젯 |
| `supportedLocales` | O | — | 지원 로케일 목록 |
| `path` | O | — | 번역 파일 경로 |
| `assetLoader` | X | `RootBundleAssetLoader()` | 커스텀 에셋 로더 |
| `fallbackLocale` | X | — | 로케일 미지원 시 대체 로케일 |
| `startLocale` | X | — | 디바이스 로케일 무시하고 시작 로케일 강제 지정 |
| `saveLocale` | X | `true` | 로케일 디바이스 저장 여부 |
| `useFallbackTranslations` | X | `false` | 누락 키에 대해 fallback 번역 사용 |
| `useFallbackTranslationsForEmptyResources` | X | `false` | 빈 값에 대해 fallback 번역 사용 |
| `useOnlyLangCode` | X | `false` | 언어 코드만 사용 (en.json vs en-US.json) |
| `ignorePluralRules` | X | `true` | few/many 복수형 무시 |

## 번역 파일 구조

### JSON 예제 (en-US.json)

```json
{
  "title": "PhilGo App",
  "welcome": "Welcome to PhilGo",
  "msg": "{} are written in the {} language",
  "msg_named": "{app} is written in the {lang} language",
  "gender": {
    "male": "Hi man ;) {}",
    "female": "Hello girl :) {}",
    "other": "Hello {}"
  },
  "clicked": {
    "zero": "Click the button!",
    "one": "Clicked {} time",
    "other": "Clicked {} times"
  },
  "example": {
    "hello": "Hello",
    "world": "World!",
    "helloWorld": "@:example.hello @:example.world"
  }
}
```

## 기본 사용법

### `tr()` 번역 호출 방식

```dart
// 1. context extension (권장)
Text(context.tr('title'))

// 2. Text 위젯 extension
Text('title').tr()

// 3. String extension
print('title'.tr())

// 4. 정적 함수
var title = tr('title')
```

### 코드 생성된 키 사용 (권장)

```dart
import 'generated/locale_keys.g.dart';

Text(LocaleKeys.title).tr()
print(LocaleKeys.title.tr())
```

## 인자 전달

### 위치 인자 (Positional Arguments)

```json
{ "msg": "{} are written in the {} language" }
```

```dart
Text('msg').tr(args: ['Easy localization', 'Dart'])
// 결과: "Easy localization are written in the Dart language"
```

### 이름 인자 (Named Arguments)

```json
{ "msg_named": "{app} is written in the {lang} language" }
```

```dart
Text('msg_named').tr(namedArgs: {'app': 'PhilGo', 'lang': 'Dart'})
// 결과: "PhilGo is written in the Dart language"
```

## 복수형 처리

### 번역 파일

```json
{
  "day": {
    "zero": "{} 일",
    "one": "{} 일",
    "other": "{} 일"
  },
  "money": {
    "zero": "돈이 없습니다",
    "one": "{} 달러 있습니다",
    "other": "{} 달러 있습니다"
  }
}
```

> **필수**: `"other"` 키는 반드시 포함해야 합니다.

### 사용법

```dart
// 기본 복수형
Text('clicked').plural(counter)

// 숫자 포맷 적용
Text('money').plural(1000000,
  format: NumberFormat.compact(locale: context.locale.toString()))

// 인자 함께 사용
plural('money_args', 10.23, args: ['John', '10.23'])
plural('money_named_args', 10.23, namedArgs: {'name': 'Jane', 'money': '10.23'})
```

## 성별 처리

### 번역 파일

```json
{
  "gender": {
    "male": "안녕하세요 형님 {}",
    "female": "안녕하세요 언니 {}",
    "other": "안녕하세요 {}"
  }
}
```

### 사용법

```dart
Text('gender').tr(gender: isFemale ? 'female' : 'male')

// 인자와 함께
Text('gender').tr(args: ['홍길동'], gender: isFemale ? 'female' : 'male')
```

## 연결 번역 (Linked Translations)

다른 번역 키를 참조하여 재사용할 수 있습니다.

### 번역 파일

```json
{
  "example": {
    "hello": "Hello",
    "world": "World!",
    "helloWorld": "@:example.hello @:example.world",
    "fullName": "Full Name",
    "emptyNameError": "Please fill in your @.lower:example.fullName"
  }
}
```

### 포맷 수정자

| 수정자 | 설명 | 예시 |
|--------|------|------|
| `@.upper:key` | 전체 대문자 | `FULL NAME` |
| `@.lower:key` | 전체 소문자 | `full name` |
| `@.capitalize:key` | 첫 글자 대문자 | `Full name` |

## 로케일 관리

```dart
// 로케일 변경
context.setLocale(Locale('ko', 'KR'));

// 현재 로케일 확인
print(context.locale.toString());  // ko_KR

// 디바이스 기본 로케일로 초기화
context.resetLocale();

// 디바이스 로케일 확인
print(context.deviceLocale.toString());

// 저장된 로케일 삭제
context.deleteSaveLocale();

// 지원 로케일 목록 확인
print(context.supportedLocales);   // [en_US, ko_KR]

// fallback 로케일 확인
print(context.fallbackLocale);
```

## 코드 생성

코드 생성을 사용하면 타입 안전한 키를 사용할 수 있어 오타를 방지합니다.

### 로더 생성

```bash
flutter pub run easy_localization:generate \
  -S assets/translations \
  -O lib/generated \
  -o codegen_loader.g.dart
```

### 키 상수 클래스 생성

```bash
flutter pub run easy_localization:generate \
  -S assets/translations \
  -O lib/generated \
  -o locale_keys.g.dart \
  -f keys
```

### 생성 명령 옵션

| 옵션 | 기본값 | 설명 |
|------|--------|------|
| `-S`, `--source-dir` | `resources/langs` | 번역 파일 소스 디렉토리 |
| `-s`, `--source-file` | — | 특정 소스 파일 |
| `-O`, `--output-dir` | `lib/generated` | 출력 디렉토리 |
| `-o`, `--output-file` | `codegen_loader.g.dart` | 출력 파일명 |
| `-f`, `--format` | `json` | 출력 형식 (`json` 또는 `keys`) |
| `-u`, `--skip-unnecessary-keys` | — | 불필요한 키 건너뛰기 |

### 생성된 코드 사용

```dart
import 'generated/codegen_loader.g.dart';
import 'generated/locale_keys.g.dart';

// EasyLocalization에 CodegenLoader 지정
EasyLocalization(
  assetLoader: CodegenLoader(),
  // ...
)

// 타입 안전한 키 사용
Text(LocaleKeys.title).tr();
print(LocaleKeys.title.tr());
Text(LocaleKeys.clicked).plural(counter);
```

## Asset Loaders

기본 `RootBundleAssetLoader` 외에 `easy_localization_loader` 패키지로 다양한 로더를 사용할 수 있습니다.

### 설치

```yaml
dependencies:
  easy_localization_loader: <latest_version>
```

### 지원 로더 종류

| 로더 | 설명 |
|------|------|
| `RootBundleAssetLoader()` | 기본 Flutter 에셋 로더 (JSON) |
| `CodegenLoader()` | 코드 생성 기반 로더 |
| `JsonAssetLoader()` | JSON 파일 로더 |
| `CsvAssetLoader()` | CSV 파일 로더 |
| `YamlAssetLoader()` | YAML 다중 파일 로더 |
| `YamlSingleAssetLoader()` | YAML 단일 파일 로더 |
| `XmlAssetLoader()` | XML 다중 파일 로더 |
| `XmlSingleAssetLoader()` | XML 단일 파일 로더 |
| `HttpAssetLoader()` | HTTP 원격 로더 |
| `FileAssetLoader()` | 파일 시스템 로더 |

### 로더 설정 예제

```dart
EasyLocalization(
  supportedLocales: [Locale('en', 'US'), Locale('ko', 'KR')],
  path: 'resources/langs/langs.csv',  // CSV 파일 경로
  assetLoader: CsvAssetLoader(),
  child: MyApp(),
)
```

## 멀티 모듈 지원

여러 패키지의 번역을 병합할 수 있습니다.

```dart
EasyLocalization(
  supportedLocales: [Locale('en', 'US')],
  path: 'resources/langs',
  assetLoader: CodegenLoader(),
  extraAssetLoaders: [
    TranslationsLoader(packageName: 'package_example_1'),
    TranslationsLoader(packageName: 'package_example_2'),
  ],
  child: MyApp(),
)
```

## iOS 설정

iOS에서 다국어를 지원하려면 `ios/Runner/Info.plist`에 로케일을 추가해야 합니다.

```xml
<key>CFBundleLocalizations</key>
<array>
  <string>en</string>
  <string>ko</string>
</array>
```

## 로거 설정

### 경고/에러 메시지만 표시

```dart
EasyLocalization.logger.enableLevels = [
  LevelMessages.error,
  LevelMessages.warning,
];
```

### 로거 비활성화

```dart
EasyLocalization.logger.enableBuildModes = [];
```

### 커스텀 로그 프린터

```dart
EasyLocalization.logger.printer = (
  Object object, {
  String? name,
  StackTrace? stackTrace,
  LevelMessages? level,
}) {
  debugPrint('$name: ${object.toString()}');
};
```

## 유틸리티

### String → Locale 변환

```dart
'en_US'.toLocale()                  // Locale('en', 'US')
'ko_KR'.toLocale()                  // Locale('ko', 'KR')
'en|US'.toLocale(separator: '|')    // Locale('en', 'US')
```

### Locale → String 변환

```dart
Locale('en', 'US').toStringWithSeparator(separator: '|')  // "en|US"
```

## 전체 예제

```dart
import 'package:easy_localization/easy_localization.dart';
import 'package:flutter/material.dart';

import 'generated/locale_keys.g.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await EasyLocalization.ensureInitialized();

  runApp(EasyLocalization(
    child: MyApp(),
    supportedLocales: [
      Locale('en', 'US'),
      Locale('ko', 'KR'),
      Locale('de', 'DE'),
    ],
    path: 'assets/translations',
    fallbackLocale: Locale('en', 'US'),
  ));
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      localizationsDelegates: context.localizationDelegates,
      supportedLocales: context.supportedLocales,
      locale: context.locale,
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int counter = 0;
  bool _gender = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(LocaleKeys.title).tr(),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 성별 처리
            Text(LocaleKeys.gender).tr(
              gender: _gender ? 'female' : 'male',
            ),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(Icons.male),
                Switch(value: _gender, onChanged: (val) {
                  setState(() => _gender = val);
                }),
                Icon(Icons.female),
              ],
            ),
            // 인자 전달
            Text(LocaleKeys.msg).tr(args: ['PhilGo', 'Flutter']),
            // 이름 인자
            Text(LocaleKeys.msg_named).tr(namedArgs: {'lang': 'Dart'}),
            // 복수형
            Text(LocaleKeys.clicked).plural(counter),
            TextButton(
              onPressed: () => setState(() => counter++),
              child: Text(LocaleKeys.clickMe).tr(),
            ),
            // 로케일 초기화 버튼
            ElevatedButton(
              onPressed: () => context.resetLocale(),
              child: Text(LocaleKeys.reset_locale).tr(),
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() => counter++),
        child: Text('+1'),
      ),
    );
  }
}
```
