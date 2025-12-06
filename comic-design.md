# Comic Design Reference

Comic 스타일 UI 디자인 가이드라인과 코드 예제를 제공합니다.

## Table of Contents

- [Design Principles](#design-principles)
- [Theme Guidelines](#theme-guidelines)
- [ComicButton System](#comicbutton-system)
- [ComicTextFormField](#comictextformfield)
- [List-Based Widgets](#list-based-widgets)
- [ComicModalBottomSheet](#comicmodalbottomsheet)
- [Comic AppBar](#comic-appbar)
- [Comic SnackBar](#comic-snackbar)

---

## Design Principles

### 핵심 디자인 규칙

| 속성 | 값 | 설명 |
|------|-----|------|
| Border Width | `2.0` (표준), `1.5` (목록) | Comic 스타일 테두리 |
| Border Radius | `12` (큰 요소), `8` (작은 요소) | 둥근 모서리 |
| Elevation | `0` | 항상 그림자 없음 |
| 간격 | 8의 배수 | 8, 16, 24, 32... |

### 색상 사용 규칙

| 용도 | Theme 색상 |
|------|------------|
| 주요 요소 | `colorScheme.primary` |
| 보조 요소 | `colorScheme.secondary` |
| 카드/컨테이너 | `colorScheme.surface` |
| 테두리 | `colorScheme.outline` |
| 텍스트/아이콘 | `colorScheme.onSurface` |

### 타이포그래피 규칙

| 용도 | Text Style |
|------|------------|
| 일반 텍스트 | `textTheme.bodyLarge` |
| 제목 | `textTheme.titleMedium` |
| 버튼 | `textTheme.labelLarge` |

### 재사용 가능한 위젯

모든 Comic 위젯은 `./lib/widgets/theme/comic_<component_name>.dart`에 저장합니다.

---

## Theme Guidelines

### 필수 규칙

```dart
// ❌ 절대 금지
Text('Login')                              // 하드코딩된 텍스트
color: Colors.blue                         // 하드코딩된 색상
fontSize: 16                               // 하드코딩된 크기
border: Border.all()                       // 테두리 허용 안됨 (outline 사용)
elevation: 4                               // 0이어야 함
ElevatedButton(
  child: Text('Click', style: TextStyle(...))  // 인라인 스타일이 테마 덮어씀
)

// ✅ 반드시 사용
Text(T.login)                              // i18n 번역
color: Theme.of(context).colorScheme.primary   // 테마 색상
style: Theme.of(context).textTheme.bodyLarge   // 테마 텍스트 스타일
elevation: 0                               // 플랫 디자인
ElevatedButton(child: Text(T.click))       // 테마가 스타일링 처리
```

---

## ComicButton System

통합된 재사용 가능한 버튼 컴포넌트 라이브러리입니다.

### 핵심 디자인 원칙

- **테두리**: outline 색상으로 2.0px 두께
- **Elevation**: 항상 0 (플랫 디자인)
- **모서리**: borderRadius 12 (normal) 또는 완전히 둥근 형태 (full/pill)
- **색상**: 테마 기반 (surface, primary, secondary)

### Design Options

#### ComicButtonRounded

| 값 | 설명 | 사용 사례 |
|-----|------|----------|
| `full` | 알약/스타디움 형태 | CTA 버튼, 로그인 버튼 |
| `normal` | 둥근 모서리 (borderRadius: 12) | 표준 버튼 |

#### ComicButtonPadding

| 값 | 패딩 | 사용 사례 |
|-----|------|----------|
| `large` | 32×20 | 중요한 액션 버튼 |
| `medium` | 24×16 | 표준 버튼 (기본값) |
| `small` | 16×12 | 컴팩트 버튼 |

#### ComicButtonTextSize

| 값 | 텍스트 스타일 | 사용 사례 |
|-----|-------------|----------|
| `large` | titleMedium | 중요한 액션 버튼 |
| `medium` | bodyLarge | 표준 버튼 (기본값) |
| `small` | bodyMedium | 컴팩트 버튼 |

### 버튼 변형

#### ComicButton (기본)

```dart
// 표준 버튼
ComicButton(
  onPressed: () => doSomething(),
  child: Text(Lo.of(context)!.action),
)

// 큰 로그인 버튼 (알약 형태)
ComicButton(
  onPressed: () => login(),
  rounded: ComicButtonRounded.full,
  padding: ComicButtonPadding.large,
  textSize: ComicButtonTextSize.large,
  child: Text(Lo.of(context)!.login),
)
```

#### ComicPrimaryButton

```dart
ComicPrimaryButton(
  onPressed: () => submit(),
  rounded: ComicButtonRounded.full,
  padding: ComicButtonPadding.large,
  textSize: ComicButtonTextSize.large,
  child: Text(Lo.of(context)!.submit),
)
```

#### ComicSecondaryButton

```dart
ComicSecondaryButton(
  onPressed: () => cancel(),
  padding: ComicButtonPadding.small,
  textSize: ComicButtonTextSize.small,
  child: Text(Lo.of(context)!.cancel),
)
```

### Quick Reference

| 버튼 스타일 | rounded | padding | textSize |
|-----------|---------|---------|----------|
| 큰 CTA | `full` | `large` | `large` |
| 표준 | `normal` | `medium` | `medium` |
| 컴팩트 | `normal` | `small` | `small` |

**파일 위치**: `./lib/widgets/theme/comic_button.dart`

---

## ComicTextFormField

Comic 디자인 언어를 구현하는 재사용 가능한 텍스트 입력 컴포넌트입니다.

### 디자인 원칙

- **테두리**: outline 색상으로 1.0px 두께
- **Elevation**: 항상 0
- **모서리**: borderRadius 12
- **색상**: surface 배경, outline 테두리, primary 포커스

### 사용 예제

#### 기본 텍스트 필드

```dart
ComicTextFormField(
  controller: _nameController,
  labelText: T.fullName,
  hintText: T.fullNameHint,
)
```

#### 비밀번호 필드

```dart
ComicTextFormField(
  controller: _passwordController,
  labelText: T.password,
  hintText: T.enterPassword,
  obscureText: true,
  prefixIcon: Icon(Icons.lock),
)
```

#### 유효성 검사 포함

```dart
ComicTextFormField(
  controller: _emailController,
  labelText: T.email,
  hintText: T.enterEmail,
  keyboardType: TextInputType.emailAddress,
  validator: (value) {
    if (value == null || value.isEmpty) {
      return T.emailRequired;
    }
    return null;
  },
)
```

#### 여러 줄 텍스트 영역

```dart
ComicTextFormField(
  controller: _bioController,
  labelText: T.bio,
  hintText: T.enterBio,
  maxLines: 5,
  minLines: 3,
  maxLength: 500,
)
```

### 테두리 상태

| 상태 | 색상 | 두께 |
|------|------|------|
| Enabled | outline | 1.0px |
| Focused | primary | 1.0px |
| Error | error | 1.0px |
| Disabled | outline 50% | 1.0px |

### 주요 매개변수

| 매개변수 | 타입 | 설명 |
|---------|------|------|
| controller | TextEditingController? | 텍스트 필드 컨트롤러 |
| labelText | String? | 라벨 텍스트 |
| hintText | String? | 힌트 텍스트 |
| obscureText | bool | 비밀번호용 텍스트 숨김 |
| validator | Function? | 유효성 검사 함수 |
| borderWidth | double | 테두리 두께 (기본: 1.0) |
| borderRadius | double | 모서리 반경 (기본: 12) |

**파일 위치**: `./lib/widgets/theme/comic_text_form_field.dart`

---

## List-Based Widgets

ListTile 및 Compact Cards와 같은 목록 기반 콘텐츠용입니다.

### 디자인 원칙

- **테두리**: `1.0` 두께 (표준 2.0보다 얇음)
- **Border Radius**: `12`
- **Elevation**: `0`
- **Margin**: `EdgeInsets.zero`

### 구현 예제

```dart
Card(
  elevation: 0,
  margin: EdgeInsets.zero,
  color: Theme.of(context).colorScheme.surface,
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(12),
    side: BorderSide(
      color: Theme.of(context).colorScheme.outline,
      width: 1.0,
    ),
  ),
  child: InkWell(
    onTap: onTap,
    borderRadius: BorderRadius.circular(12),
    child: // ... 콘텐츠
  ),
)
```

---

## ComicModalBottomSheet

Comic 디자인 언어를 구현하는 모달 바텀 시트 컴포넌트입니다.

### 디자인 원칙

- **Border Radius**: 상단 좌우 `12.0`
- **Border Width**: 상단/좌/우 `2.0` (하단 없음)
- **Elevation**: `0`
- **드래그 핸들**: 32px × 4px

### 사용법

```dart
showModalBottomSheet(
  context: context,
  backgroundColor: Colors.transparent,
  elevation: 0,
  builder: (context) {
    return Container(
      decoration: BoxDecoration(
        color: Theme.of(context).colorScheme.surface,
        borderRadius: const BorderRadius.only(
          topLeft: Radius.circular(12.0),
          topRight: Radius.circular(12.0),
        ),
        border: Border(
          top: BorderSide(
            color: Theme.of(context).colorScheme.outline,
            width: 2.0,
          ),
          left: BorderSide(
            color: Theme.of(context).colorScheme.outline,
            width: 2.0,
          ),
          right: BorderSide(
            color: Theme.of(context).colorScheme.outline,
            width: 2.0,
          ),
        ),
      ),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          // 드래그 핸들
          const SizedBox(height: 8),
          Container(
            width: 32,
            height: 4,
            decoration: BoxDecoration(
              color: Theme.of(context).colorScheme.onSurfaceVariant.withValues(alpha: 0.4),
              borderRadius: BorderRadius.circular(2),
            ),
          ),
          const SizedBox(height: 8),
          // 콘텐츠
          Flexible(
            child: ListView(
              shrinkWrap: true,
              children: [
                // 목록 아이템
              ],
            ),
          ),
        ],
      ),
    );
  },
);
```

---

## Comic AppBar

앱 바와 헤더 섹션에 Comic 디자인을 구현합니다.

### 디자인 사양

| 속성 | 값 |
|------|-----|
| 높이 | 56px |
| 테두리 위치 | 하단만 |
| 테두리 두께 | 2.0px |
| 테두리 색상 | `colorScheme.outline` |
| 제목 스타일 | `titleLarge` |
| Elevation | 0 |

### 패턴 1: 표준 Scaffold AppBar

```dart
Scaffold(
  appBar: AppBar(
    title: Text(T.editProfile, style: theme.textTheme.titleLarge),
    bottom: PreferredSize(
      preferredSize: const Size.fromHeight(1),
      child: Container(height: 2, color: scheme.outline),
    ),
  ),
  body: // 콘텐츠
)
```

### 패턴 2: 커스텀 헤더

```dart
SafeArea(
  child: Container(
    height: 56,
    padding: const EdgeInsets.only(right: 4, left: 12),
    decoration: BoxDecoration(
      border: Border(
        bottom: BorderSide(
          color: Theme.of(context).colorScheme.outline,
          width: 2.0,
        ),
      ),
    ),
    child: Row(
      children: [
        Expanded(
          child: Text(
            T.pageTitle,
            style: Theme.of(context).textTheme.titleLarge
                ?.copyWith(fontWeight: FontWeight.w600),
            overflow: TextOverflow.ellipsis,
          ),
        ),
        IconButton(
          icon: const FaIcon(FontAwesomeIcons.someIcon),
          onPressed: () {},
        ),
      ],
    ),
  ),
)
```

---

## Comic SnackBar

Comic 디자인 원칙을 따르는 알림 메시지 시스템입니다.

### 디자인 사양

| 속성 | 값 |
|------|-----|
| 테두리 두께 | 2.0px |
| Border Radius | 12px |
| Elevation | 0 |
| 동작 | Floating |
| 마진 | 전체 16px |
| 패딩 | 가로 16px, 세로 12px |

### 사용 가능한 함수

#### showComicSuccessSnackBar

```dart
showComicSuccessSnackBar(context, T.profileUpdateSuccess);
```

- 배경: #F1F8F4 (밝은 녹색)
- 텍스트: #2E7D32 (진한 녹색)
- 테두리: #66BB6A (중간 녹색)

#### showComicErrorSnackBar

```dart
showComicErrorSnackBar(context, T.nicknameRequired);
```

- 배경: #FFF5F5 (밝은 빨간색)
- 텍스트: #C62828 (진한 빨간색)
- 테두리: #EF5350 (중간 빨간색)

#### showComicInfoSnackBar

```dart
showComicInfoSnackBar(context, T.pleaseWait);
```

- 배경: #F3F8FC (밝은 파란색)
- 텍스트: #1565C0 (진한 파란색)
- 테두리: #42A5F5 (중간 파란색)

#### showComicWarningSnackBar

```dart
showComicWarningSnackBar(context, T.pleaseCheckInput);
```

- 배경: #FFF8F0 (밝은 주황색)
- 텍스트: #EF6C00 (진한 주황색)
- 테두리: #FFA726 (중간 주황색)

### 사용 예제

```dart
// 폼 제출 성공 후
final user = await philgoApiUserUpdate(data);
if (mounted) {
  AppState.of(context).setUser(user);
  showComicSuccessSnackBar(context, T.profileUpdateSuccess);
}

// 유효성 검사 실패
if (_nicknameController.text.trim().isEmpty) {
  showComicErrorSnackBar(context, T.nicknameRequired);
  return;
}

// 에러 처리
try {
  await philgoApiFileDelete(photoUrl);
  showComicSuccessSnackBar(context, T.photoDeleted);
} catch (e) {
  showComicErrorSnackBar(context, '삭제 실패: $e');
}
```

**파일 위치**: `./lib/widgets/theme/comic_snackbar.dart`

```dart
import 'package:philgo/widgets/theme/comic_snackbar.dart';
```
