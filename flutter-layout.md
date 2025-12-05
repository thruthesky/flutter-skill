## 스크롤 화면

- 일반 스크롤 화면 또는 `앱바/탭/고정 헤더 + 스크롤 콘텐츠` 같은 복합 스크롤 화면은 반드시 `CustomScrollView + Sliver` 조합으로 만듭니다.

- 고정되는 헤더 + 리스트 + 섹션 등이 섞이면 CustomScrollView가 훨씬 깔끔하고 강력해집니다.


```dart
CustomScrollView(
  slivers: [
    SliverAppBar(pinned: true, title: Text("Title")),
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (_, i) => ItemWidget(i),
        childCount: 100,
      ),
    ),
  ],
);
```

장점
	•	스크롤되는 영역을 “조립” 가능
	•	pinned/float/stretch 같은 고급 UX 가능
	•	대규모 화면에서 표준



##  키보드/폼 중심 화면(로그인, 입력 폼)

- 키보드 입력이 필요한 경우, `ListView` (특히 `padding + keyboardDismissBehavior`) 를 사용합니다.

- 폼은 사실상 `필드들의 리스트`라 ListView가 더 잘 맞고 또한 대부분의 경우, 콘텐츠가 화면을 넘치기 때문에 스크롤이 필요합니다.


```dart
ListView(
  padding: EdgeInsets.all(16),
  keyboardDismissBehavior: ScrollViewKeyboardDismissBehavior.onDrag,
  children: [
    TextField(...),
    TextField(...),
    ...
  ],
);
```

장점
	•	키보드 올라올 때 스크롤/레이아웃이 덜 꼬임
	•	dismiss UX도 좋음