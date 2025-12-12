# Font Awesome Flutter 사용 가이드

이 프로젝트에서는 **Font Awesome 7.1 Pro Icons**를 사용합니다.

## 라이센스 정보

- **라이센스 만료일**: 2026년 12월 12일
- ⚠️ 이 날짜 전에 최신 버전을 한 번 다운로드해야 합니다.

---

## 설치 방법

### 1. pubspec.yaml에 패키지 추가

```yaml
dependencies:
  font_awesome_flutter: ^10.12.0
```

### 2. dependency_overrides 설정

Pro 아이콘을 사용하기 위해 로컬 패키지로 오버라이드합니다:

```yaml
dependency_overrides:
  font_awesome_flutter:
    path: packages/font_awesome_flutter
```

### 3. Git Submodule 추가

Font Awesome Pro 아이콘이 포함된 커스텀 패키지를 submodule로 추가합니다:

```bash
git submodule add https://github.com/thruthesky/font_awesome_flutter ./packages/font_awesome_flutter
```

기존 프로젝트를 클론한 경우 submodule 초기화:

```bash
git submodule update --init --recursive
```

---

## 사용 방법

### Import

```dart
import 'package:font_awesome_flutter/font_awesome_flutter.dart';
```

### FaIcon 위젯 사용

```dart
// Light 스타일 (권장)
FaIcon(FontAwesomeIcons.lightUser)

// Regular 스타일
FaIcon(FontAwesomeIcons.user)

// Solid 스타일
FaIcon(FontAwesomeIcons.solidUser)

// 크기 및 색상 지정
FaIcon(
  FontAwesomeIcons.lightCamera,
  size: 24,
  color: Theme.of(context).colorScheme.primary,
)
```

### 아이콘 스타일 우선순위

디자인 가이드라인에 따라 다음 우선순위로 아이콘 스타일을 선택합니다:

1. **Light** (최우선) - `FontAwesomeIcons.light*`
2. **Regular** - `FontAwesomeIcons.*` (접두사 없음)
3. **Solid** - `FontAwesomeIcons.solid*`

---

## 아이콘 검색

- 공식 사이트: https://fontawesome.com/icons
- Flutter 패키지: https://pub.dev/packages/font_awesome_flutter

---

## Pro vs Free 아이콘

| 구분 | Free | Pro |
|------|------|-----|
| Solid | ✅ | ✅ |
| Regular | ✅ | ✅ |
| Light | ❌ | ✅ |
| Thin | ❌ | ✅ |
| Duotone | ❌ | ✅ |
| Sharp | ❌ | ✅ |
| Brands | ✅ | ✅ |

이 프로젝트는 **Pro 라이센스**를 사용하므로 모든 스타일의 아이콘을 사용할 수 있습니다.

---

## 주의사항

- `Icon` 위젯 대신 반드시 `FaIcon` 위젯을 사용하세요.
- Pro 아이콘은 `packages/font_awesome_flutter` 경로의 로컬 패키지에서 제공됩니다.
- submodule이 초기화되지 않으면 아이콘이 표시되지 않습니다.
