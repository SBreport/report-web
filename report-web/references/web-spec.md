# 웹페이지 생성 상세 스펙

Phase 4에서 coder 에이전트에 전달할 정확한 스펙. **이 문서 내용을 coder 프롬프트에 그대로 포함**할 것.

## 파일 구조

```
<project>_web/
├── index.html              ← 빌드 결과물 (더블클릭 실행)
├── src/
│   └── shell.html          ← 쉘 뼈대 (head, 사이드바, 메인, 네비게이션 바, <!-- {{SECTIONS}} --> 마커)
├── sections/
│   ├── 01_<name>.html      ← 각 챕터별 <template> 블록들
│   ├── 02_<name>.html
│   └── ...
├── css/
│   └── styles.css
├── js/
│   └── app.js
├── build.bat               ← Windows 빌드 스크립트
├── build.sh                ← Mac/Linux 빌드 스크립트
└── README.md               ← 편집·빌드·배포 가이드
```

## 빌드 스크립트 원리

- `src/shell.html`에 `<!-- {{SECTIONS}} -->` 마커
- `sections/*.html` 파일들을 이름 순으로 읽어 하나의 문자열로 연결
- 마커를 치환하여 `index.html`에 저장
- **UTF-8 인코딩 보존 필수** (Windows 배치는 `chcp 65001` 또는 PowerShell `-Encoding utf8`)

`build.bat` 예시 (PowerShell):
```batch
@echo off
powershell -NoProfile -Command ^
  "$shell = Get-Content -Raw -Encoding UTF8 src/shell.html; ^
   $sections = (Get-ChildItem sections/*.html | Sort-Object Name | ForEach-Object { Get-Content -Raw -Encoding UTF8 $_.FullName }) -join \"`n\"; ^
   $out = $shell -replace '<!-- {{SECTIONS}} -->', $sections; ^
   Set-Content -Path index.html -Value $out -Encoding UTF8"
echo index.html built successfully.
```

`build.sh` 예시 (Python 활용):
```bash
#!/bin/bash
python3 - <<'PY'
from pathlib import Path
shell = Path('src/shell.html').read_text(encoding='utf-8')
sections = '\n'.join(p.read_text(encoding='utf-8') for p in sorted(Path('sections').glob('*.html')))
Path('index.html').write_text(shell.replace('<!-- {{SECTIONS}} -->', sections), encoding='utf-8')
print('index.html built successfully.')
PY
```

## 섹션 로드 방식 — `<template>` 태그

- 각 섹션은 `<template id="tpl-<chapter>-<n>">...</template>`로 감쌈
- 브라우저가 파싱만 하고 렌더링 X (성능 중립)
- JS `cloneNode()`로 탭·서브슬라이드 전환 시 삽입
- **fetch 기반 외부 파일 로드 금지** — `file://` CORS 때문에 더블클릭 실행 불가해짐

## 디자인 스펙

### 컬러 팔레트 (블루)

- Primary Dark: `#051E4F`
- Primary: `#0A3D91`
- Accent: `#4A9EFF`
- Background: `#fff` / `#f7f9fc`
- Border: `#dce3ef`
- Text Sub: `#6b7a99`
- Vacancy: `#9ca3af`

### 타이포 스케일

- 슬라이드 타이틀: `clamp(32px, 4vw, 56px)` / 굵게
- 히어로 숫자·인용: `clamp(48px, 6vw, 96px)` / 매우 굵게 / Primary 컬러
- 섹션 소제목: `clamp(20px, 2vw, 28px)`
- 본문: `clamp(16px, 1.4vw, 20px)`
- 표 본문: 16~18px
- 라인 하이트 1.5~1.7

### 레이아웃

- `#app-shell`은 16:9 비율 `min(100vw, 100vh * 16/9)` 식으로 자동 계산
- 좌측 사이드바 탭 7개(챕터 수만큼), 너비 약 160~200px
- 메인 영역: 슬라이드 header + body + 하단 네비게이션 바
- body는 `overflow:hidden` 기본, 내부는 카드로 구획

### 네비게이션

- **챕터**: 좌측 사이드바 탭 (click)
- **서브 슬라이드**: 하단 중앙 `<` `1 / N` `>` (click)
- **키보드**: ← → 화살표로 서브 슬라이드, 선택적으로 PgUp/PgDn로 챕터
- 서브 슬라이드 1개일 때는 네비게이션 숨김 처리
- 챕터 전환 시 첫 슬라이드로 자동 이동

### 간격 규칙 (⚠ 중요)

**`.child + .child { margin }` 같은 인접 형제 전역 규칙 사용 금지.** flex/grid의 `gap`과 중복되어 레이아웃이 12px씩 밀리는 이슈가 반복적으로 발생한다. 이 규칙이 언제 트리거되는지 모른 채 디버깅하면 시간을 많이 잡아먹는다.

올바른 패턴:
- flex/grid 컨테이너 → 부모에 `gap: Npx` 지정
- Block flow 스택 → 부모를 `display:flex;flex-direction:column;gap:Npx` wrapper로 변경
- 단일 요소 → `margin-bottom:Npx` 인라인 OK

### 상태 처리 필수

- Loading: 스피너 + "로딩 중..." 표시
- Empty: "준비 중" 플레이스홀더 카드
- Error: 아이콘 + 에러 메시지

### 금지 사항 (AI Slop 회피)

- 이모지 UI (텍스트 본문 이모지는 OK지만 UI 요소는 X)
- 과한 `rounded-xl+`
- 보라·핑크 강조
- 전면 그라데이션 (배너·히어로 등 제한적 사용만 OK)
- 3열 feature grid 장식
- `text-[9px]` 이하

## 콘텐츠 컴포넌트 패턴

### 숫자 카드 (히어로)

```html
<div class="stat-grid stat-grid-3">
  <div class="stat-card">
    <div class="hero-number">2023</div>
    <div class="hero-label">설립</div>
  </div>
  ...
</div>
```

`.stat-grid`는 `display:grid; gap:12px` + `.stat-grid-3 { grid-template-columns: repeat(3, 1fr); }`.

### 일반 정보 카드

```html
<div class="content-card">
  <h2 class="section-heading">섹션명</h2>
  <table class="data-table">...</table>
</div>
```

### 여러 카드를 세로 스택

```html
<div style="display:flex;flex-direction:column;gap:12px;">
  <div class="content-card">...</div>
  <div class="content-card">...</div>
</div>
```

### 좌우 2열 레이아웃

```html
<div style="display:grid;grid-template-columns:65fr 35fr;gap:16px;">
  <div>...</div>
  <div>...</div>
</div>
```

### 외부 링크 버튼

```html
<a class="notion-link-btn" href="..." target="_blank" rel="noopener">
  상세 보기 →
</a>
```

CSS: outlined 스타일, 배경 `#eef4ff`, 테두리 `#4A9EFF`, 텍스트 `#0A3D91`.

## 흔한 이슈 (반복 출현 시 점검 체크리스트)

1. **카드 수직 위치 불일치** — `.content-card + .content-card { margin-top }` 전역 규칙 의심. 즉시 제거 + 부모 `gap` 적용.
2. **같은 flex 행 내 카드 크기 차이** — margin이 적용된 자식이 섞여있어 행 높이 변동. 인라인 `margin-top:0`으로 잠시 덮거나 전역 규칙 제거.
3. **16:9 화면에서 스크롤 발생** — 슬라이드 body에 `overflow:hidden`, 자식 grid/flex는 `min-height:0` 설정.
4. **한 슬라이드에 정보 과다** — 서브 슬라이드 1장 더 쪼갤 것.
5. **같은 숫자/성과 반복 노출** — 한 슬라이드에만 남기고 다른 곳은 서술형 메시지로 대체.

## coder 에이전트 위임 형식

coder에게 맡길 때 반드시 포함할 것:
- 저장 경로
- 파일 구조 (위 구조 복사)
- 디자인 스펙 (팔레트·타이포·간격 규칙)
- 챕터별 콘텐츠 확정본 (Phase 2에서 정리한 마크다운)
- 챕터별 서브 슬라이드 분할표 (Phase 3에서 합의)
- PPT 강조 포인트 리스트
- "간격은 부모 gap으로만" 규칙 **명시**
- 빌드 스크립트 UTF-8 요구사항 **명시**
- 완료 후 보고할 항목 (파일 목록·슬라이드 수·강조 위치)
