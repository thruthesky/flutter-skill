# 카카오톡 오픈채팅으로 연결하기

Flutter 앱에서 카카오톡 오픈채팅 링크를 열어 1:1 채팅을 시작하는 방법을 설명합니다.

## 중요: 프로필 QR 링크 vs 1:1 오픈채팅 링크

### ❌ 프로필 QR 링크 (권장하지 않음)

카카오톡 프로필 QR 코드 링크는 다음과 같은 형식입니다:

```
http://qr.kakao.com/talk/XXXXX
```

**문제점**:
- 프로필 QR 링크로는 **친구 추가가 제대로 작동하지 않습니다**
- Flutter 앱에서 `launchUrl()`로 열어도 친구 추가 화면으로 이동하지 않는 경우가 많음
- Android와 iOS 모두에서 불안정한 동작

### ✅ 1:1 오픈채팅 링크 (권장)

**반드시 "1:1 오픈채팅방" 링크를 사용해야 합니다**:

```
https://open.kakao.com/o/sXXXXXX
```

**장점**:
- 카카오톡이 자연스럽게 열림
- 오픈채팅이지만 **1:1 채팅방**으로 연결됨
- 채팅 연결 성공률이 높음
- `launchUrl()`로 직접 열어도 잘 작동함

## 1:1 오픈채팅방 만드는 방법

1. 카카오톡 앱 열기
2. 하단 탭에서 "채팅" 선택
3. 우측 상단 "+" 버튼 → "오픈채팅" 선택
4. "오픈채팅방 만들기" 선택
5. **"1:1 채팅"** 선택 (중요!)
6. 채팅방 이름, 프로필 사진 설정
7. 생성 후 채팅방 설정에서 **"링크 복사"**

## Flutter 구현 (권장 방식)

1:1 오픈채팅 링크는 `launchUrl()`로 직접 열어도 잘 작동합니다:

```dart
case ContactType.kakaotalk:
  // 카카오톡: 1:1 오픈채팅 링크를 직접 열기
  // 반드시 "1:1 오픈채팅방" 링크를 사용해야 함 (프로필 QR 링크는 작동하지 않음)
  if (url != null && url!.isNotEmpty) {
    targetUrl = url;
  }
  break;

// ...

if (targetUrl != null && targetUrl.isNotEmpty) {
  final uri = Uri.parse(targetUrl);
  await launchUrl(uri, mode: LaunchMode.externalApplication);
}
```

## Deep Link 문제 해결

만약 앱에서 `philgo.com` 도메인을 deep link로 처리하고 있다면, 해당 도메인의 URL을 `launchUrl()`로 열 때 앱 자체가 URL을 처리하려고 합니다.

### 해결책 1: 서브도메인 사용

`philgo.com` 대신 `link.philgo.com` 같은 서브도메인 사용:

```dart
targetUrl = 'https://link.philgo.com/redirect?url=$encodedUrl';
```

### 해결책 2: 카카오톡 링크 직접 사용

1:1 오픈채팅 링크(`https://open.kakao.com/...`)는 앱의 deep link에 등록되어 있지 않으므로 직접 사용 가능:

```dart
// 카카오톡 오픈채팅 링크는 직접 열기
targetUrl = url; // https://open.kakao.com/o/sXXXXXX
await launchUrl(Uri.parse(targetUrl), mode: LaunchMode.externalApplication);
```

---

## 참고: 웹 브라우저 중간 페이지 방식 (선택사항)

직접 열기가 불안정한 경우, 웹 브라우저를 통해 중간 페이지를 열 수도 있습니다:

```
1. Flutter 앱에서 카카오톡 연락처 카드 터치
2. 외부 웹 브라우저로 중간 페이지 열기
   → https://link.philgo.com/link/kakaotalk.php?link={encodedKakaoUrl}
3. 중간 페이지에서 "카카오톡 열기" 버튼 표시
4. 사용자가 버튼 클릭
5. 카카오톡 앱 실행 및 오픈채팅 연결
```

---

## 플랫폼별 설정 (참고용)

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
