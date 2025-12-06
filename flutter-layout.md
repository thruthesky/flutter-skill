# Flutter Layout Reference

Flutter 레이아웃 및 스크롤 화면 구성을 위한 필수 가이드라인입니다.

## Table of Contents

- [Layout Selection Guide](#layout-selection-guide)
- [CustomScrollView with Slivers](#customscrollview-with-slivers)
- [ListView for Forms](#listview-for-forms)
- [Common Layout Patterns](#common-layout-patterns)
- [Responsive Design](#responsive-design)

---

## Layout Selection Guide

### 화면 유형별 권장 레이아웃

| 화면 유형 | 권장 레이아웃 | 이유 |
|----------|-------------|------|
| 앱바 + 스크롤 콘텐츠 | `CustomScrollView + Sliver` | 고급 UX 가능 |
| 고정 헤더 + 리스트 | `CustomScrollView + Sliver` | 조립 가능 |
| 로그인/입력 폼 | `ListView` | 키보드 처리 용이 |
| 단순 스크롤 | `SingleChildScrollView` | 간단한 구현 |
| 고정 레이아웃 | `Column/Row` | 스크롤 불필요 |

---

## CustomScrollView with Slivers

### 사용 시점

- 앱바/탭/고정 헤더 + 스크롤 콘텐츠 조합
- 고정되는 헤더 + 리스트 + 섹션이 섞인 화면
- pinned/float/stretch 같은 고급 UX 필요 시
- 대규모 화면 표준 레이아웃

### 기본 구조

```dart
CustomScrollView(
  slivers: [
    SliverAppBar(pinned: true, title: Text(T.title)),
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (_, i) => ItemWidget(i),
        childCount: 100,
      ),
    ),
  ],
)
```

### SliverAppBar 옵션

| 속성 | 설명 | 사용 사례 |
|------|------|----------|
| `pinned: true` | 스크롤해도 앱바 고정 | 항상 보이는 앱바 |
| `floating: true` | 위로 스크롤 시 앱바 표시 | 빠른 접근 필요 |
| `snap: true` | floating과 함께, 스냅 효과 | 부드러운 UX |
| `stretch: true` | 당기면 앱바 확장 | 리프레시 효과 |
| `expandedHeight` | 확장 시 높이 | 이미지 배경 앱바 |

### 다양한 Sliver 위젯

```dart
CustomScrollView(
  slivers: [
    // 고정 앱바
    SliverAppBar(
      pinned: true,
      expandedHeight: 200,
      flexibleSpace: FlexibleSpaceBar(
        title: Text(T.title),
        background: Image.network(url, fit: BoxFit.cover),
      ),
    ),

    // 고정 헤더
    SliverPersistentHeader(
      pinned: true,
      delegate: MySliverHeaderDelegate(),
    ),

    // 리스트
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ListTile(title: Text('Item $index')),
        childCount: items.length,
      ),
    ),

    // 그리드
    SliverGrid(
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        mainAxisSpacing: 8,
        crossAxisSpacing: 8,
      ),
      delegate: SliverChildBuilderDelegate(
        (context, index) => GridItem(index),
        childCount: gridItems.length,
      ),
    ),

    // 하단 패딩
    SliverPadding(
      padding: const EdgeInsets.only(bottom: 80),
      sliver: SliverToBoxAdapter(child: Container()),
    ),
  ],
)
```

### SliverToBoxAdapter

일반 위젯을 Sliver로 변환합니다.

```dart
CustomScrollView(
  slivers: [
    SliverAppBar(...),

    // 일반 위젯을 Sliver로 변환
    SliverToBoxAdapter(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Text(T.sectionTitle, style: theme.textTheme.titleLarge),
      ),
    ),

    SliverList(...),
  ],
)
```

---

## ListView for Forms

### 사용 시점

- 키보드 입력이 필요한 화면
- 로그인, 회원가입, 입력 폼
- 콘텐츠가 화면을 넘칠 수 있는 경우

### 기본 구조

```dart
ListView(
  padding: const EdgeInsets.all(16),
  keyboardDismissBehavior: ScrollViewKeyboardDismissBehavior.onDrag,
  children: [
    ComicTextFormField(
      controller: _emailController,
      labelText: T.email,
    ),
    const SizedBox(height: 16),
    ComicTextFormField(
      controller: _passwordController,
      labelText: T.password,
      obscureText: true,
    ),
    const SizedBox(height: 24),
    ComicPrimaryButton(
      onPressed: _submit,
      child: Text(T.login),
    ),
  ],
)
```

### 장점

- 키보드 올라올 때 스크롤/레이아웃이 덜 꼬임
- `keyboardDismissBehavior`로 키보드 닫기 UX 제공
- 폼은 사실상 "필드들의 리스트"라 ListView가 적합

### ListView.builder vs ListView

| 사용 방법 | 사용 시점 |
|----------|----------|
| `ListView(children: [...])` | 아이템 수가 적을 때 (10개 미만) |
| `ListView.builder()` | 아이템 수가 많거나 동적일 때 |
| `ListView.separated()` | 구분선이 필요할 때 |

```dart
// 많은 아이템 - builder 사용
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => ItemWidget(items[index]),
)

// 구분선 필요 - separated 사용
ListView.separated(
  itemCount: items.length,
  itemBuilder: (context, index) => ItemWidget(items[index]),
  separatorBuilder: (context, index) => const Divider(),
)
```

---

## Common Layout Patterns

### 패턴 1: 앱바 + 스크롤 리스트

```dart
Scaffold(
  body: CustomScrollView(
    slivers: [
      SliverAppBar(
        pinned: true,
        title: Text(T.title),
        bottom: PreferredSize(
          preferredSize: const Size.fromHeight(1),
          child: Container(height: 2, color: scheme.outline),
        ),
      ),
      SliverList(
        delegate: SliverChildBuilderDelegate(
          (context, index) => ListItem(items[index]),
          childCount: items.length,
        ),
      ),
    ],
  ),
)
```

### 패턴 2: 탭 + 스크롤 콘텐츠

```dart
DefaultTabController(
  length: 3,
  child: Scaffold(
    body: NestedScrollView(
      headerSliverBuilder: (context, innerBoxIsScrolled) => [
        SliverAppBar(
          pinned: true,
          title: Text(T.title),
          bottom: const TabBar(
            tabs: [
              Tab(text: 'Tab 1'),
              Tab(text: 'Tab 2'),
              Tab(text: 'Tab 3'),
            ],
          ),
        ),
      ],
      body: TabBarView(
        children: [
          TabContent1(),
          TabContent2(),
          TabContent3(),
        ],
      ),
    ),
  ),
)
```

### 패턴 3: 고정 헤더 + 리스트 + FAB

```dart
Scaffold(
  body: CustomScrollView(
    slivers: [
      SliverAppBar(pinned: true, title: Text(T.title)),
      SliverPadding(
        padding: const EdgeInsets.all(16),
        sliver: SliverList(
          delegate: SliverChildBuilderDelegate(
            (context, index) => Padding(
              padding: const EdgeInsets.only(bottom: 8),
              child: ItemCard(items[index]),
            ),
            childCount: items.length,
          ),
        ),
      ),
      // FAB 공간 확보
      const SliverToBoxAdapter(
        child: SizedBox(height: 80),
      ),
    ],
  ),
  floatingActionButton: FloatingActionButton(
    onPressed: _addItem,
    child: const Icon(Icons.add),
  ),
)
```

### 패턴 4: 검색 + 필터 + 리스트

```dart
CustomScrollView(
  slivers: [
    // 검색바
    SliverAppBar(
      floating: true,
      snap: true,
      title: SearchBar(controller: _searchController),
    ),

    // 필터 칩
    SliverToBoxAdapter(
      child: SingleChildScrollView(
        scrollDirection: Axis.horizontal,
        padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
        child: Row(
          children: filters.map((f) => FilterChip(label: Text(f))).toList(),
        ),
      ),
    ),

    // 결과 리스트
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ResultItem(results[index]),
        childCount: results.length,
      ),
    ),
  ],
)
```

---

## Responsive Design

### MediaQuery 사용

```dart
@override
Widget build(BuildContext context) {
  final screenWidth = MediaQuery.of(context).size.width;
  final isTablet = screenWidth > 600;

  return isTablet
      ? TwoColumnLayout(...)
      : SingleColumnLayout(...);
}
```

### LayoutBuilder 사용

```dart
LayoutBuilder(
  builder: (context, constraints) {
    if (constraints.maxWidth > 1200) {
      return DesktopLayout();
    } else if (constraints.maxWidth > 600) {
      return TabletLayout();
    } else {
      return MobileLayout();
    }
  },
)
```

### 그리드 반응형 레이아웃

```dart
SliverGrid(
  gridDelegate: SliverGridDelegateWithMaxCrossAxisExtent(
    maxCrossAxisExtent: 200,  // 최대 너비
    mainAxisSpacing: 16,
    crossAxisSpacing: 16,
    childAspectRatio: 1,
  ),
  delegate: SliverChildBuilderDelegate(
    (context, index) => GridItem(index),
    childCount: items.length,
  ),
)
```

---

## Quick Reference

| 상황 | 사용 위젯 |
|------|----------|
| 앱바 + 스크롤 | `CustomScrollView + SliverAppBar + SliverList` |
| 입력 폼 | `ListView + keyboardDismissBehavior` |
| 탭 + 스크롤 | `NestedScrollView + TabBarView` |
| 그리드 레이아웃 | `SliverGrid` 또는 `GridView.builder` |
| 일반 위젯을 Sliver로 | `SliverToBoxAdapter` |
| 고정 헤더 | `SliverPersistentHeader` |
