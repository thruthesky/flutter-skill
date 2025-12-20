# 카카오톡 프로필 QR 링크로 친구 추가하기

Flutter 앱에서 카카오톡 프로필 QR 링크를 열어 친구 추가 기능을 구현하는 방법을 설명합니다.

## 개요

카카오톡은 프로필 QR 코드 링크를 통해 친구 추가를 할 수 있습니다. 이 링크를 `url_launcher` 패키지를 사용하여 열면 카카오톡 앱이 실행되어 친구 추가 화면으로 이동합니다.

## 카카오톡 프로필 QR 링크 형식

카카오톡 프로필 QR 링크는 다음과 같은 형식입니다:

```
https://open.kakao.com/...
```

또는 카카오톡 오픈채팅 링크:

```
https://open.kakao.com/o/sXXXXXX
```

## 중요: 웹 브라우저를 통한 카카오톡 열기 (권장)

### 문제점

Flutter 앱에서 `launchUrl()`을 사용하여 카카오톡 프로필 QR 링크를 직접 열 경우, Android와 iOS 모두에서 **불안정한 동작**이 발생합니다:
- 카카오톡 앱이 열리지 않는 경우
- 앱이 열리더라도 친구 추가 화면으로 이동하지 않는 경우
- 플랫폼별 설정(`<queries>`, `LSApplicationQueriesSchemes`)이 올바르더라도 간헐적 실패

### 해결책: 웹 브라우저 중간 페이지 방식

카카오톡 프로필 QR 링크를 직접 열지 않고, **웹 브라우저를 통해 중간 페이지**를 열어서 사용자가 클릭하여 카카오톡 앱을 열도록 합니다.

#### 동작 흐름

```
1. Flutter 앱에서 카카오톡 연락처 카드 터치
2. 외부 웹 브라우저로 중간 페이지 열기
   → https://philgo.com/link/kakaotalk.php?link={encodedKakaoUrl}
3. 중간 페이지에서 "카카오톡 열기" 버튼 표시
4. 사용자가 버튼 클릭
5. 카카오톡 앱 실행 및 친구 추가 화면 이동
```

#### Flutter 코드 (권장 방식)

```dart
case ContactType.kakaotalk:
  // 카카오톡: 직접 launchUrl()로 열지 않고, 웹 브라우저를 통해 중간 페이지를 열어서
  // 사용자가 클릭하여 카카오톡 앱을 열도록 함 (앱에서 직접 열기가 불안정하기 때문)
  if (url != null && url!.isNotEmpty) {
    // 카카오톡 프로필 QR 링크를 URL 인코딩하여 파라미터로 전달
    final encodedUrl = Uri.encodeComponent(url!);
    targetUrl = 'https://philgo.com/link/kakaotalk.php?link=$encodedUrl';
  }
  break;
```

#### 중간 페이지 (kakaotalk.php) 예제

```php
<?php
// https://philgo.com/link/kakaotalk.php
$link = $_GET['link'] ?? '';
$decodedLink = urldecode($link);
?>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>카카오톡 친구 추가</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            background: #F7C815;
        }
        .btn {
            background: #3C1E1E;
            color: white;
            padding: 16px 32px;
            border-radius: 12px;
            text-decoration: none;
            font-size: 18px;
            font-weight: bold;
        }
        .info {
            margin-top: 16px;
            color: #3C1E1E;
            font-size: 14px;
        }
    </style>
</head>
<body>
    <a class="btn" href="<?= htmlspecialchars($decodedLink) ?>">카카오톡 열기</a>
    <p class="info">위 버튼을 터치하면 카카오톡 앱이 열립니다</p>
</body>
</html>
```

### 왜 이 방식이 더 안정적인가?

1. **사용자 제스처 기반**: 웹 브라우저에서 사용자가 직접 링크를 클릭하면, 앱 스위칭이 더 안정적으로 동작합니다.
2. **플랫폼 독립적**: 웹 브라우저가 카카오톡 앱을 여는 것은 각 플랫폼에서 이미 잘 테스트된 경로입니다.
3. **설정 간소화**: Flutter 앱에서 복잡한 `<queries>` 설정 없이도 동작합니다.
4. **디버깅 용이**: 중간 페이지에서 로그를 남기거나 폴백 처리를 할 수 있습니다.

---

## Flutter 직접 구현 (참고용)

> ⚠️ 아래 방식은 불안정할 수 있습니다. 위의 "웹 브라우저 중간 페이지 방식"을 권장합니다.

### 1. url_launcher 패키지 설치

```yaml
# pubspec.yaml
dependencies:
  url_launcher: ^6.2.0
```

### 2. Dart 코드

```dart
import 'package:url_launcher/url_launcher.dart';

/// 카카오톡 프로필 QR 링크로 친구 추가
///
/// [qrUrl] - 카카오톡 프로필 QR 링크 (예: https://open.kakao.com/...)
Future<void> openKakaoTalkFriend(String qrUrl) async {
  final uri = Uri.parse(qrUrl);

  if (await canLaunchUrl(uri)) {
    // 외부 앱(카카오톡)으로 열기
    await launchUrl(uri, mode: LaunchMode.externalApplication);
  } else {
    // 카카오톡이 설치되지 않은 경우 외부 브라우저로 열기
    await launchUrl(uri, mode: LaunchMode.externalNonBrowserApplication);
  }
}
```

### 3. LaunchMode 옵션

- `LaunchMode.externalApplication`: 외부 앱(카카오톡)으로 열기 (권장)
- `LaunchMode.externalNonBrowserApplication`: 외부 앱으로 열되, 브라우저는 제외
- `LaunchMode.inAppBrowserView`: Custom Tab으로 앱 내에서 열기
- `LaunchMode.platformDefault`: 플랫폼 기본 동작

## 플랫폼별 설정

### Android (AndroidManifest.xml)

Android API 레벨 30 이상에서는 `<queries>` 설정이 **필수**입니다. 이 설정이 없으면 `launchUrl`이 제대로 작동하지 않습니다.

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- API 레벨 30 이상에서 필수 visibility 설정 -->
    <queries>
        <!-- 텍스트 처리 인텐트 -->
        <intent>
            <action android:name="android.intent.action.PROCESS_TEXT"/>
            <data android:mimeType="text/plain"/>
        </intent>

        <!-- SMS 지원 확인용 (선택사항) -->
        <intent>
            <action android:name="android.intent.action.VIEW" />
            <data android:scheme="sms" />
        </intent>

        <!-- 전화 지원 확인용 (선택사항) -->
        <intent>
            <action android:name="android.intent.action.VIEW" />
            <data android:scheme="tel" />
        </intent>

        <!-- ⚠️ Custom Tab 지원 - 필수! -->
        <!-- 이 설정이 없으면 외부 브라우저/Custom Tab으로 URL을 열 수 없음 -->
        <intent>
            <action android:name="android.support.customtabs.action.CustomTabsService" />
        </intent>
    </queries>

    <application>
        ...
    </application>
</manifest>
```

#### 중요 사항

- **`android.support.customtabs.action.CustomTabsService`**: 이 인텐트가 **필수**입니다. 없으면 `launchUrl`에서 카카오톡을 직접 실행하거나 외부 웹 브라우저를 열 때 external (custom tab)으로 열 수 없습니다.
- `sms`, `tel` 스킴: 선택사항이지만, 해당 기능을 사용한다면 추가해야 합니다.

### iOS (Info.plist)

iOS에서는 `LSApplicationQueriesSchemes`를 설정합니다:

```xml
<!-- ios/Runner/Info.plist -->
<dict>
    ...
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>sms</string>
        <string>tel</string>
        <string>kakaolink</string>
        <string>kakaokompassauth</string>
        <string>kakaotalk</string>
    </array>
    ...
</dict>
```

## canLaunchUrl() 함수 이해

`canLaunchUrl()` 함수는 특정 URL 스킴을 앱에서 열 수 있는지 확인합니다.

### Android

- API 30+ 에서는 `<queries>`에 해당 스킴이 등록되어 있어야 `canLaunchUrl()`이 `true`를 반환합니다.
- 등록하지 않으면 항상 `false`를 반환합니다.

### iOS

- `LSApplicationQueriesSchemes`에 스킴이 등록되어 있어야 `canLaunchUrl()`이 `true`를 반환합니다.
- 등록하지 않으면 항상 `false`를 반환합니다.

### 주의사항

`sms`, `tel` 스킴을 등록하지 않으면 `canLaunchUrl(Uri.parse('sms:123'))`은 항상 `false`를 반환합니다. 이것이 카카오톡 프로필 QR 링크 열기와 직접적인 연관은 없지만, `canLaunchUrl()` 함수의 동작 방식을 이해하는 데 중요합니다.

## 실제 사용 예제

```dart
import 'package:flutter/material.dart';
import 'package:url_launcher/url_launcher.dart';

class KakaoTalkContactCard extends StatelessWidget {
  /// 카카오톡 ID
  final String kakaoId;

  /// 카카오톡 프로필 QR URL
  final String? qrUrl;

  const KakaoTalkContactCard({
    super.key,
    required this.kakaoId,
    this.qrUrl,
  });

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _openKakaoTalk,
      child: Container(
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: const Color(0xFFF7C815), // 카카오톡 노란색
          borderRadius: BorderRadius.circular(12),
        ),
        child: Row(
          children: [
            const Icon(Icons.chat_bubble, color: Color(0xFF3C1E1E)),
            const SizedBox(width: 12),
            Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                const Text('카카오톡', style: TextStyle(color: Color(0xFF3C1E1E))),
                Text(kakaoId, style: const TextStyle(
                  fontWeight: FontWeight.bold,
                  color: Color(0xFF3C1E1E),
                )),
              ],
            ),
          ],
        ),
      ),
    );
  }

  /// 카카오톡 프로필 QR 링크 열기
  Future<void> _openKakaoTalk() async {
    if (qrUrl == null || qrUrl!.isEmpty) return;

    final uri = Uri.parse(qrUrl!);

    if (await canLaunchUrl(uri)) {
      await launchUrl(uri, mode: LaunchMode.externalApplication);
    }
  }
}
```

## 트러블슈팅

### 1. URL이 열리지 않는 경우

- Android: `AndroidManifest.xml`에 `<queries>` 설정이 있는지 확인
- iOS: `Info.plist`에 `LSApplicationQueriesSchemes` 설정이 있는지 확인
- URL 형식이 올바른지 확인

### 2. canLaunchUrl()이 항상 false를 반환하는 경우

- 해당 스킴이 플랫폼 설정에 등록되어 있는지 확인
- Android API 30+ 에서는 `<queries>` 설정이 필수

### 3. Custom Tab으로 열리지 않는 경우

- `android.support.customtabs.action.CustomTabsService` 인텐트가 `<queries>`에 등록되어 있는지 확인

## 참고

- [url_launcher 패키지](https://pub.dev/packages/url_launcher)
- [Android Package Visibility](https://developer.android.com/training/package-visibility)
- [iOS LSApplicationQueriesSchemes](https://developer.apple.com/documentation/bundleresources/information_property_list/lsapplicationqueriesschemes)
