


# Flutter 디자인 개요

- 이 스킬은 Comic 스타일 UI 디자인 가이드라인과 코드를 제공합니다.
- Flutter 애플리케이션이 Comic UI 스타일을 갖도록 보장합니다.

# 사용 시점

- 사용자가 Flutter 애플리케이션의 UI 디자인 또는 개선을 요청할 때 이 스킬을 사용하세요.
- 사용자가 Comic 스타일 UI 디자인을 언급할 때 이 스킬을 사용하세요.

# 디자인 가이드라인

- 자세한 디자인 가이드라인과 코드 예제는 [comic-theme.md](./comic-theme.md)를 참조하세요.

# Comic 디자인 규칙

- Flutter에서 Comic 스타일 UI를 구현할 때 다음의 중요한 디자인 규칙을 따르세요.

- **재사용 가능한 Flutter 위젯**: 항상 코드 반복을 피하기 위해 재사용 가능한 위젯을 사용하거나 생성하세요.

  - 예를 들어, 각 버튼을 개별적으로 스타일링하는 대신 `./lib/widgets/theme/ComicButton` 위젯을 생성하세요.
  - 모든 `Comic` 위젯은 `./lib/widgets/theme/comic_<component_name>.dart`에 저장해야 합니다.
  - 버튼, 카드, 텍스트 필드 등 공통 UI 요소에 대해 재사용 가능한 위젯을 생성해야 합니다.

- **Comic 디자인**: 항상 다음 Comic 디자인 원칙을 따르세요:

  - **테두리**:
    - 표준 요소: `2.0` 두께
    - 목록 기반 위젯 (ListTile, 컴팩트 카드): 가벼운 시각적 무게를 위해 `1.5` 두께
  - **그림자 없음**: 항상 그림자 없음
  - **높이 없음**: 항상 elevation 0
  - **외곽선**: 테두리에 outline 색상 사용
  - **둥근 모서리**: 큰 요소는 border radius `12`, 다른 요소는 `8`.
  - **색상**: 주요 요소에는 테마 Primary Color 사용.
    - 보조 요소에는 테마 Secondary Color 사용.
    - 카드와 컨테이너에는 테마 Surface Color 사용.
    - 표면 색상에는 테마 Background Color 사용.
    - 표면 위의 텍스트와 아이콘에는 테마 onSurface Color 사용.
  - **폰트**: 모든 텍스트 요소에 테마 Text Styles 사용.
    - 일반 텍스트에는 `bodyLarge` 사용.
    - 제목에는 `titleMedium` 사용.
    - 버튼에는 `labelLarge` 사용.

- **간격**: 8의 배수 사용 (8, 16, 24, 32 등)

## 테마 디자인 가이드라인

**중요한 Flutter 규칙:**

- **테마만 사용**: 항상 색상, 폰트, 스타일에 `Theme.of(context)`를 사용하세요. 절대 하드코딩하지 마세요.

❌ 절대 하지 마세요:

```dart
Text('Login')  // 하드코딩된 텍스트
color: Colors.blue  // 하드코딩된 색상
fontSize: 16  // 하드코딩된 크기
border: Border.all()  // 테두리 허용 안됨
elevation: 4  // 0이어야 함
ElevatedButton(child: Text('Click', style: TextStyle(...)))  // 인라인 스타일이 테마를 덮어씀
```

✅ 항상 이렇게 하세요:

```dart
Text(T.login)  // i18n 번역
color: Theme.of(context).colorScheme.primary  // 테마 색상
style: Theme.of(context).textTheme.bodyLarge  // 테마 텍스트 스타일
elevation: 0  // 플랫 디자인
ElevatedButton(child: Text(T.click))  // 테마가 스타일링 처리
```

# ComicButton 시스템

## 개념 및 의도

**ComicButton** 시스템은 전체 Flutter 애플리케이션에서 Comic 디자인 언어를 구현하는 통합된 재사용 가능한 버튼 컴포넌트 라이브러리입니다. 인라인 스타일로 버튼을 개별적으로 스타일링하는 대신, 모든 버튼은 `ComicButton` 기본 위젯 또는 그 변형을 사용합니다.

**주요 이점:**

- **일관성**: 모든 버튼이 동일한 Comic 디자인 DNA를 공유 (2px 테두리, 그림자 없음, 둥근 모서리)
- **유연성**: 세 가지 디자인 옵션 (`rounded`, `padding`, `textSize`)으로 모든 버튼 스타일 생성 가능
- **유지보수성**: 한 곳에서 버튼 스타일을 변경하면 모든 곳에서 업데이트
- **테마 통합**: 자동으로 테마 색상과 텍스트 스타일 사용

## 핵심 디자인 원칙

모든 ComicButton 변형은 다음 Comic 디자인 규칙을 따릅니다:

- **테두리**: outline 색상으로 2.0px 두께
- **Elevation**: 항상 0 (플랫 디자인, 그림자 없음)
- **모서리**: borderRadius 12 (normal) 또는 완전히 둥근 형태 (full/pill)로 둥글게
- **색상**: 테마 기반 (surface, primary, secondary)
- **타이포그래피**: 테마 텍스트 스타일 (bodyLarge, titleMedium, bodyMedium)
- **간격**: 8의 배수로 패딩

## 디자인 옵션

모든 `ComicXxxButton` 변형은 세 가지 디자인 옵션 enum을 지원합니다:

### ComicButtonRounded

버튼의 모서리 형태를 제어합니다.
| 값 | 설명 | 사용 사례 |
|-------|-------------|----------|
| `full` | 알약/스타디움 형태 (StadiumBorder) | CTA 버튼, 로그인 버튼 |
| `normal` | 둥근 모서리 (borderRadius: 12) | 표준 버튼 |

### ComicButtonPadding

버튼의 내부 패딩을 제어합니다.
| 값 | 패딩 | 사용 사례 |
|-------|---------|----------|
| `large` | 32×20 | 중요한 액션 버튼 (로그인, 제출) |
| `medium` | 24×16 | 표준 버튼 (기본값) |
| `small` | 16×12 | 컴팩트 버튼, 인라인 액션 |

### ComicButtonTextSize

버튼 내부의 텍스트 크기를 제어합니다.
| 값 | 텍스트 스타일 | 사용 사례 |
|-------|------------|----------|
| `large` | titleMedium | 중요한 액션 버튼 |
| `medium` | bodyLarge | 표준 버튼 (기본값) |
| `small` | bodyMedium | 컴팩트 버튼 |

## 버튼 변형

### ComicButton (기본)

중립 색상 (surface 배경, onSurface 텍스트)을 가진 기본 위젯입니다.

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

### ComicPrimaryButton

테마 primary 색상을 사용하는 주요 액션 버튼입니다.

```dart
ComicPrimaryButton(
  onPressed: () => submit(),
  rounded: ComicButtonRounded.full,
  padding: ComicButtonPadding.large,
  textSize: ComicButtonTextSize.large,
  child: Text(Lo.of(context)!.submit),
)
```

### ComicSecondaryButton

테마 secondary 색상을 사용하는 보조 액션 버튼입니다.

```dart
ComicSecondaryButton(
  onPressed: () => cancel(),
  padding: ComicButtonPadding.small,
  textSize: ComicButtonTextSize.small,
  child: Text(Lo.of(context)!.cancel),
)
```

## 빠른 참조

| 버튼 스타일 | rounded  | padding  | textSize |
| ------------ | -------- | -------- | -------- |
| 큰 CTA    | `full`   | `large`  | `large`  |
| 표준     | `normal` | `medium` | `medium` |
| 컴팩트      | `normal` | `small`  | `small`  |

## 파일 위치

모든 ComicButton 위젯은 다음에 정의되어 있습니다:
`./lib/widgets/theme/comic_button.dart`

# ComicTextFormField

## 개념 및 의도

**ComicTextFormField**는 Comic 디자인 언어를 구현하는 재사용 가능한 텍스트 입력 컴포넌트입니다. Comic 스타일 테두리, 적절한 간격, 테마 통합으로 전체 Flutter 애플리케이션에서 일관된 텍스트 입력 필드를 생성하는 방법을 제공합니다.

**주요 이점:**

- **일관성**: 모든 텍스트 필드가 동일한 Comic 디자인 DNA를 공유 (2px 테두리, 그림자 없음, 둥근 모서리)
- **유연성**: Comic 디자인 원칙을 유지하면서 광범위한 커스터마이징 옵션 제공
- **유지보수성**: 한 곳에서 텍스트 필드 스타일을 변경하면 모든 곳에서 업데이트
- **테마 통합**: 자동으로 테마 색상과 텍스트 스타일 사용

## 디자인 원칙

모든 ComicTextFormField 인스턴스는 다음 Comic 디자인 규칙을 따릅니다:

- **테두리**: outline 색상으로 1.0px 두께 (커스터마이징 가능) - 컴팩트 입력 필드를 위한 가벼운 무게
- **Elevation**: 항상 0 (플랫 디자인, 그림자 없음)
- **모서리**: borderRadius 12로 둥글게 (커스터마이징 가능)
- **색상**: 테마 기반 (surface 배경, outline 테두리, primary 포커스, error)
- **타이포그래피**: 테마 텍스트 스타일 (콘텐츠는 bodyLarge, 라벨/힌트는 bodyMedium)
- **간격**: 8의 배수로 콘텐츠 패딩

## 주요 기능

- **완전한 TextFormField API**: 모든 표준 TextFormField 매개변수 지원
- **커스터마이징 가능한 테두리**: 테두리 색상, 포커스 색상, 에러 색상, 두께, 반경 설정 가능
- **테마 통합**: 모든 시각적 요소에 테마 색상 사용
- **유효성 검사 지원**: 내장 validator와 에러 텍스트 표시
- **입력 제어**: 읽기 전용, 활성화/비활성화, 텍스트 숨김, 키보드 타입 지원
- **아이콘**: prefix 및 suffix 아이콘 지원
- **여러 줄**: 단일 또는 여러 줄 입력을 위한 min/max lines 설정 가능
- **길이 제어**: 카운터가 있는 최대 길이 설정

## 사용 예제

### 기본 텍스트 필드

```dart
ComicTextFormField(
  controller: _nameController,
  labelText: T.fullName,
  hintText: T.fullNameHint,
)
```

### 비밀번호 필드

```dart
ComicTextFormField(
  controller: _passwordController,
  labelText: T.password,
  hintText: T.enterPassword,
  obscureText: true,
  prefixIcon: Icon(Icons.lock),
)
```

### 유효성 검사 포함

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

### 여러 줄 텍스트 영역

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

### 비활성화된 필드

```dart
ComicTextFormField(
  controller: _nicknameController,
  labelText: T.nickname,
  hintText: T.nicknameHint,
  enabled: false, // 또는 readOnly: true
)
```

## 커스터마이징 옵션

### 테두리 스타일링

```dart
ComicTextFormField(
  controller: _controller,
  labelText: T.label,
  // 커스텀 테두리 스타일링
  borderWidth: 1.0,        // 기본값: 1.0
  borderRadius: 12,        // 기본값: 12
  borderColor: Colors.grey,
  focusedBorderColor: Colors.blue,
  errorBorderColor: Colors.red,
)
```

### 콘텐츠 패딩

```dart
ComicTextFormField(
  controller: _controller,
  labelText: T.label,
  // 커스텀 패딩 (기본값: symmetric horizontal: 16, vertical: 16)
  contentPadding: EdgeInsets.all(20),
)
```

## 매개변수

| 매개변수 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| controller | TextEditingController? | null | 텍스트 필드 컨트롤러 |
| labelText | String? | null | 입력 위에 표시되는 라벨 텍스트 |
| hintText | String? | null | 비어있을 때 표시되는 힌트 텍스트 |
| errorText | String? | null | 표시할 에러 메시지 |
| prefixIcon | Widget? | null | 필드 시작 부분의 아이콘 |
| suffixIcon | Widget? | null | 필드 끝 부분의 아이콘 |
| keyboardType | TextInputType? | null | 입력용 키보드 타입 |
| obscureText | bool | false | 비밀번호용 텍스트 숨김 |
| onChanged | ValueChanged<String>? | null | 텍스트 변경 시 콜백 |
| onFieldSubmitted | ValueChanged<String>? | null | 제출 시 콜백 |
| validator | String? Function(String?)? | null | 유효성 검사 함수 |
| readOnly | bool | false | 편집 방지 |
| enabled | bool | true | 필드 활성화/비활성화 |
| maxLines | int? | 1 | 최대 줄 수 |
| minLines | int? | null | 최소 줄 수 |
| maxLength | int? | null | 최대 문자 수 |
| autofocus | bool | false | 로드 시 자동 포커스 |
| borderColor | Color? | null | 테두리 색상 (기본값: outline) |
| focusedBorderColor | Color? | null | 포커스 테두리 (기본값: primary) |
| errorBorderColor | Color? | null | 에러 테두리 (기본값: error) |
| borderWidth | double | 1.0 | 테두리 두께 |
| borderRadius | double | 12 | 모서리 반경 |
| contentPadding | EdgeInsetsGeometry? | null | 내부 패딩 |

## 테두리 상태

ComicTextFormField는 자동으로 다섯 가지 테두리 상태를 처리합니다:

1. **Enabled 테두리**: 기본 상태 (outline 색상, 1.0px)
2. **Focused 테두리**: 필드에 포커스가 있을 때 (primary 색상, 1.0px)
3. **Error 테두리**: 유효성 검사 실패 시 (error 색상, 1.0px)
4. **Focused Error 테두리**: 포커스가 있는 에러 상태 (error 색상, 1.0px)
5. **Disabled 테두리**: 필드가 비활성화되었을 때 (outline 50% 불투명도, 1.0px)

## 파일 위치

ComicTextFormField는 다음에 정의되어 있습니다:
`./lib/widgets/theme/comic_text_form_field.dart`

# 목록 기반 위젯 디자인

`ListTile` 및 `Compact Cards`와 같은 목록 기반 콘텐츠용:

## 디자인 원칙

- **테두리**: 컴팩트 목록에서 가벼운 시각적 무게를 위해 `1.0` 두께 사용 (표준 2.0보다 얇음)
- **Border Radius**: 둥근 모서리를 위해 `12`
- **Elevation**: `0` (그림자 없음)
- **Margin**: `EdgeInsets.zero` (부모가 간격 제어)
- **색상**: 테마 색상 사용 (surface, outline, onSurface)

### 주요 기능

```dart
Card(
  elevation: 0, // Comic 디자인: 그림자 없음
  margin: EdgeInsets.zero, // 마진 없음 (부모가 간격 제어)
  color: Theme.of(context).colorScheme.surface,
  // Comic 디자인: 컴팩트 목록을 위한 얇은 테두리와 둥근 모서리
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(12),
    side: BorderSide(
      color: Theme.of(context).colorScheme.outline,
      width: 1.0, // 목록 아이템을 위한 얇은 테두리
    ),
  ),
  child: InkWell(
    onTap: onTap,
    borderRadius: BorderRadius.circular(12), // 리플을 위해 카드 반경과 일치
    child: // ... 콘텐츠
  ),
)
```

# ComicModalBottomSheet

## 개념 및 의도

**ComicModalBottomSheet**는 Comic 디자인 언어를 구현하는 재사용 가능한 모달 바텀 시트 컴포넌트입니다. Comic 스타일의 둥근 상단 모서리, 적절한 간격, 통합된 드래그 핸들로 화면 하단에서 모달 콘텐츠를 표시하는 일관된 방법을 제공합니다.

## 디자인 원칙

- **Border Radius**: 상단 좌측과 상단 우측에 `12.0` (Comic 스타일의 둥근 상단 생성)
- **Border Width**: 상단, 좌측, 우측에 `2.0` 두께 (하단에는 테두리 없음)
- **Elevation**: `0` (플랫 디자인, 그림자 없음)
- **색상**: 테마 기반 (surface 배경, outline 테두리)
- **간격**: 8의 배수를 사용한 일관된 패딩
- **드래그 핸들**: 컨테이너 내부에 통합된 커스텀 드래그 핸들 (32px × 4px)

## 사용법

```dart
// 통합 드래그 핸들이 있는 Comic Modal Bottom Sheet 표시
showModalBottomSheet(
  context: context,
  backgroundColor: Colors.transparent, // Comic 디자인: 커스텀 스타일링을 위해 투명
  elevation: 0, // Comic 디자인: 그림자 없음
  builder: (context) {
    return Container(
      decoration: BoxDecoration(
        color: Theme.of(context).colorScheme.surface,
        borderRadius: const BorderRadius.only(
          topLeft: Radius.circular(12.0), // Comic 디자인: 둥근 상단 모서리
          topRight: Radius.circular(12.0),
        ),
        border: Border(
          top: BorderSide(
            color: Theme.of(context).colorScheme.outline,
            width: 2.0, // Comic 디자인: 2.0 테두리
          ),
          left: BorderSide(
            color: Theme.of(context).colorScheme.outline,
            width: 2.0,
          ),
          right: BorderSide(
            color: Theme.of(context).colorScheme.outline,
            width: 2.0,
          ),
          // 하단 테두리 없음
        ),
      ),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          // 드래그 핸들 인디케이터
          // Comic 디자인: 상단에서 8px 간격
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
          // 여기에 콘텐츠 (스크롤 가능하면 Flexible로 감싸기)
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

## 주요 기능

- **둥근 상단 모서리**: Comic 스타일을 위해 상단 좌측과 상단 우측에 12.0 반경
- **선택적 테두리**: 상단, 좌측, 우측에만 테두리 (하단 테두리 없음)
- **테마 통합**: surface와 outline에 테마 색상 사용
- **그림자 없음**: 플랫 디자인을 위해 Elevation 0으로 설정
- **투명 배경**: 커스텀 스타일링을 위해 부모 backgroundColor를 투명으로 설정
- **통합 드래그 핸들**: 컨테이너 내부에 적절한 간격으로 커스텀 드래그 핸들 (32×4px)
- **유연한 레이아웃**: 적절한 크기 조정을 위해 MainAxisSize.min과 Flexible이 있는 Column 사용

## 드래그 핸들 사양

- **크기**: 너비 32px × 높이 4px
- **색상**: 40% 불투명도의 `onSurfaceVariant`
- **Border Radius**: 둥근 모서리를 위해 2px
- **간격**: 상단과 하단 8px (8의 배수)
- **위치**: 컨테이너 내부, 콘텐츠 위

## 모범 사례

- `showModalBottomSheet`에서 항상 `backgroundColor: Colors.transparent` 사용
- 플랫 디자인 유지를 위해 `elevation: 0` 적용
- 일관성을 위해 테마 색상 사용
- Container의 자식을 `mainAxisSize: MainAxisSize.min`이 있는 Column으로 감싸기
- Column의 첫 번째 요소로 드래그 핸들 추가
- 스크롤 가능한 콘텐츠 (ListView, GridView 등)에는 `Flexible` 래퍼 사용
- 적절한 크기 조정을 위해 ListView에 `shrinkWrap: true` 사용
- 콘텐츠 주변에 적절한 간격 추가 (8의 배수)

# Comic AppBar 디자인

## 개념 및 의도

**Comic AppBar**는 앱 바와 헤더 섹션에 Comic 디자인 언어를 구현합니다. outline 색상을 사용한 특징적인 2.0px 하단 테두리로 헤더와 콘텐츠 영역 사이에 명확한 시각적 분리를 만드는 일관된 스타일링을 제공합니다.

## 디자인 원칙

- **테두리**: outline 색상으로 2.0px 하단 테두리
- **타이포그래피**: 제목에 테마 titleLarge 텍스트 스타일 사용
- **높이**: 표준 AppBar 높이 (56px) 또는 특수한 경우 커스텀 높이
- **색상**: 테마 기반 (surface 배경, outline 테두리, onSurface 텍스트)
- **Elevation 없음**: 항상 0 (플랫 디자인, 그림자 없음)

## 사용 패턴

### 패턴 1: 표준 Scaffold AppBar

`appBar` 속성이 있는 `Scaffold` 위젯이 있을 때 이 패턴을 사용하세요.

```dart
Scaffold(
  appBar: AppBar(
    title: Text(T.editProfile, style: theme.textTheme.titleLarge),
    bottom: PreferredSize(
      preferredSize: const Size.fromHeight(1),
      child: Container(height: 2, color: scheme.outline),
    ),
  ),
  body: // 여기에 콘텐츠
)
```

**주요 기능:**
- 표준 `AppBar` 위젯 사용
- 제목은 테마의 `titleLarge` 텍스트 스타일 사용
- 2.0px 높이 Container가 있는 `PreferredSize`를 사용하여 하단 테두리 구현
- 테두리 색상은 테마 `outline` 색상 사용

### 패턴 2: 커스텀 헤더 (Scaffold 없이)

Scaffold 없이 앱 바와 같은 헤더가 필요할 때 이 패턴을 사용하세요 (예: 특수 레이아웃, 커스텀 스크롤 뷰, 중첩 네비게이션).

```dart
SafeArea(
  child: Container(
    height: 56, // 표준 AppBar 높이
    padding: const EdgeInsets.only(right: 4, left: 12),
    decoration: BoxDecoration(
      border: Border(
        bottom: BorderSide(
          // Comic 디자인: outline 색상으로 2.0px 테두리
          color: Theme.of(context).colorScheme.outline,
          width: 2.0,
        ),
      ),
    ),
    child: Row(
      children: [
        // 제목
        Expanded(
          child: Text(
            T.pageTitle,
            style: Theme.of(context)
                .textTheme
                .titleLarge
                ?.copyWith(fontWeight: FontWeight.w600),
            overflow: TextOverflow.ellipsis,
          ),
        ),
        // 액션 버튼 (선택사항)
        IconButton(
          icon: const FaIcon(FontAwesomeIcons.someIcon),
          onPressed: () {
            // 액션 핸들러
          },
        ),
      ],
    ),
  ),
)
```

**주요 기능:**
- 하단 테두리 데코레이션이 있는 `Container` 사용
- 적절한 상단 간격을 위해 `SafeArea`로 감싸기
- 높이를 56px로 설정 (표준 AppBar 높이)
- 좌우 간격을 위한 패딩 조정
- 하단에만 테두리 적용 (outline 색상으로 2.0px)
- 제목은 `titleLarge` 텍스트 스타일 사용
- `IconButton` 또는 다른 위젯을 통한 액션 버튼 지원

## 디자인 사양

| 속성 | 값 | 설명 |
|----------|-------|-------------|
| **높이** | 56px | 표준 AppBar 높이 |
| **테두리 위치** | 하단만 | 콘텐츠와의 시각적 분리 |
| **테두리 두께** | 2.0px | Comic 표준 두께 |
| **테두리 색상** | `colorScheme.outline` | 테마 outline 색상 |
| **제목 스타일** | `titleLarge` | 테마 텍스트 스타일 |
| **Elevation** | 0 | 플랫 디자인 (그림자 없음) |
| **패딩** | 좌측: 12px, 우측: 4px | 텍스트와 아이콘을 위한 균형 잡힌 간격 |

## 각 패턴 사용 시점

### 패턴 1 (Scaffold AppBar) 사용 시:
- Scaffold로 표준 화면을 구축할 때
- 표준 AppBar 기능 (뒤로 버튼, 액션 등)이 필요할 때
- Flutter의 내장 AppBar 동작을 원할 때

### 패턴 2 (커스텀 헤더) 사용 시:
- Scaffold 위젯 없이 작업할 때
- 커스텀 스크롤 레이아웃을 구축할 때
- 중첩 네비게이션 구조를 만들 때
- 헤더 레이아웃과 동작에 대한 더 많은 제어가 필요할 때
- 복잡한 헤더 상호작용 구현 시 (예: 드롭다운 메뉴, 카테고리 선택기)

## 예제: 액션 버튼이 있는 헤더

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
            T.categories,
            style: Theme.of(context)
                .textTheme
                .titleLarge
                ?.copyWith(fontWeight: FontWeight.w600),
            overflow: TextOverflow.ellipsis,
          ),
        ),
        IconButton(
          icon: const FaIcon(FontAwesomeIcons.penToSquare),
          onPressed: () {
            // 새 아이템 생성
          },
        ),
        IconButton(
          icon: const FaIcon(FontAwesomeIcons.chevronDown),
          onPressed: () {
            // 드롭다운 메뉴 표시
          },
        ),
      ],
    ),
  ),
)
```

## 모범 사례

- **항상** 테마 색상 사용 (테두리에 outline, 텍스트에 titleLarge)
- **항상** 일관성을 위해 테두리 두께를 2.0px로 설정
- **항상** 하단에만 테두리 적용 (상단/좌측/우측 아님)
- **항상** 커스텀 헤더 구현 시 SafeArea 사용
- **절대** elevation이나 그림자 추가 안함
- 긴 제목에는 `overflow: TextOverflow.ellipsis` 사용
- 패딩과 간격에 8의 배수 사용
- 표준 AppBar와의 일관성을 위해 56px 높이 유지

# Comic SnackBar

## 개념 및 의도

**Comic SnackBar** 시스템은 Comic 디자인 원칙을 따르는 알림 메시지를 표시하는 헬퍼 함수를 제공합니다. 인라인 스타일링과 함께 표준 `ScaffoldMessenger.of(context).showSnackBar()`를 사용하는 대신, 일관되고 테마 기반의 알림을 위해 미리 만들어진 Comic SnackBar 함수를 사용하세요.

## 디자인 원칙

모든 Comic SnackBar는 다음 디자인 규칙을 따릅니다:

- **테두리**: 시맨틱 색상 (녹색, 빨간색, 파란색, 주황색)으로 2.0px 두께
- **Elevation**: 항상 0 (플랫 디자인, 그림자 없음)
- **모서리**: borderRadius 12로 둥글게
- **색상**: 부드러운 외관을 위한 낮은 대비의 시맨틱 색상:
  - 성공: 매우 밝은 녹색 배경 (#F1F8F4)에 중간 녹색 테두리 (#66BB6A)
  - 에러: 매우 밝은 빨간색 배경 (#FFF5F5)에 중간 빨간색 테두리 (#EF5350)
  - 정보: 매우 밝은 파란색 배경 (#F3F8FC)에 중간 파란색 테두리 (#42A5F5)
  - 경고: 매우 밝은 주황색 배경 (#FFF8F0)에 중간 주황색 테두리 (#FFA726)
- **타이포그래피**: 편안한 가독성을 위한 중간 톤 색상 텍스트와 테마 bodyMedium 텍스트 스타일
- **간격**: 8의 배수로 패딩과 마진
- **동작**: 적절한 마진을 가진 플로팅 스타일

## 사용 가능한 함수

### showComicSuccessSnackBar

녹색 색상 스킴으로 성공 메시지를 표시합니다.

```dart
showComicSuccessSnackBar(context, T.profileUpdateSuccess);
```

**사용 사례:**
- 폼 제출 성공
- 프로필 업데이트 성공
- 데이터 저장 성공
- 액션 완료 성공

**시각적 스타일:**
- 배경: 매우 밝은 녹색 (#F1F8F4)
- 텍스트: 중간 진한 녹색 (#2E7D32)
- 테두리: 중간 녹색 (#66BB6A, 2.0px)

### showComicErrorSnackBar

빨간색 색상 스킴으로 에러 메시지를 표시합니다.

```dart
showComicErrorSnackBar(context, T.nicknameRequired);
```

**사용 사례:**
- 유효성 검사 에러
- 실패한 작업
- 필수 필드 경고
- API 에러

**시각적 스타일:**
- 배경: 매우 밝은 빨간색 (#FFF5F5)
- 텍스트: 중간 진한 빨간색 (#C62828)
- 테두리: 중간 빨간색 (#EF5350, 2.0px)

### showComicInfoSnackBar

파란색 색상 스킴으로 정보 메시지를 표시합니다.

```dart
showComicInfoSnackBar(context, T.pleaseWait);
```

**사용 사례:**
- 정보 메시지
- 상태 업데이트
- 일반 알림
- 중립적 메시지

**시각적 스타일:**
- 배경: 매우 밝은 파란색 (#F3F8FC)
- 텍스트: 중간 진한 파란색 (#1565C0)
- 테두리: 중간 파란색 (#42A5F5, 2.0px)

### showComicWarningSnackBar

주황색 색상 스킴으로 경고 메시지를 표시합니다.

```dart
showComicWarningSnackBar(context, T.pleaseCheckInput);
```

**사용 사례:**
- 경고 메시지
- 주의 공지
- 입력 유효성 검사 경고
- 비치명적 문제

**시각적 스타일:**
- 배경: 매우 밝은 주황색 (#FFF8F0)
- 텍스트: 중간 진한 주황색 (#EF6C00)
- 테두리: 중간 주황색 (#FFA726, 2.0px)

## 사용 예제

### 기본 성공 메시지

```dart
// 폼 제출 성공 후
final user = await philgoApiUserUpdate(data);
if (mounted) {
  AppState.of(context).setUser(user);
  showComicSuccessSnackBar(context, T.profileUpdateSuccess);
}
```

### 유효성 검사 에러

```dart
// 폼 유효성 검사
if (_nicknameController.text.trim().isEmpty) {
  showComicErrorSnackBar(context, T.nicknameRequired);
  return;
}
```

### 에러 처리

```dart
try {
  await philgoApiFileDelete(photoUrl);
  showComicSuccessSnackBar(context, T.photoDeleted);
} catch (e) {
  showComicErrorSnackBar(context, '삭제 실패: $e');
}
```

### 정보 메시지

```dart
// 처리 인디케이터
showComicInfoSnackBar(context, T.processingRequest);
await performLongOperation();
```

## 디자인 사양

| 속성 | 값 | 설명 |
|----------|-------|-------------|
| **테두리 두께** | 2.0px | Comic 표준 두께 |
| **Border Radius** | 12px | 둥근 모서리 |
| **Elevation** | 0 | 플랫 디자인 (그림자 없음) |
| **동작** | Floating | SnackBar가 콘텐츠 위에 떠있음 |
| **마진** | 전체 16px | 화면 가장자리에서의 간격 |
| **패딩** | 가로: 16px, 세로: 12px | 콘텐츠 간격 |
| **텍스트 스타일** | bodyMedium | 테마 텍스트 스타일 |

## 비교: 이전 vs 이후

### ❌ 이전 (Non-Comic 스타일)

```dart
// 이전 스타일: 인라인 스타일링, 테두리 없음, 일관성 없음
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: Text(T.nicknameRequired),
    backgroundColor: Theme.of(context).colorScheme.error,
  ),
);
```

### ✅ 이후 (Comic 스타일)

```dart
// Comic 스타일: 일관된 디자인, 적절한 테두리, 테마 기반
showComicErrorSnackBar(context, T.nicknameRequired);
```

## 모범 사례

- **항상** SnackBar를 수동으로 만드는 대신 Comic SnackBar 함수 사용
- 메시지 유형에 따라 적절한 함수 **선택**:
  - 성공: `showComicSuccessSnackBar`
  - 에러/유효성 검사: `showComicErrorSnackBar`
  - 정보/중립: `showComicInfoSnackBar`
  - 경고/주의: `showComicWarningSnackBar`
- 하드코딩된 문자열 대신 지역화된 텍스트 (T.xxx 또는 Lo.xxx) **전달**
- 비동기 작업에서 SnackBar를 표시하기 전에 `mounted` **확인**
- **절대** 인라인 스타일링이나 하드코딩된 색상으로 SnackBar 생성 안함

## 파일 위치

Comic SnackBar 헬퍼 함수는 다음에 정의되어 있습니다:
`./lib/widgets/theme/comic_snackbar.dart`

Import:
```dart
import 'package:philgo/widgets/theme/comic_snackbar.dart';
```


# 스크립트

- 이 스킬은 스크립트를 제공하지 않습니다.
