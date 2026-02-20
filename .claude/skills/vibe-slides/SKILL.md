---
name: vibe-slides
description: Zero-dependency, animation-rich HTML 프레젠테이션 생성. 신규 제작, PPT 변환, 기존 개선 모두 지원. 비주얼 탐색을 통해 스타일을 발견하는 "Show, Don't Tell" 방식.
user-invocable: true
argument-hint: [주제 또는 PPT 파일 경로]
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, WebFetch, WebSearch
---

# Vibe Slides

Zero-dependency HTML 프레젠테이션 생성 스킬.

## 핵심 원칙

1. **Zero Dependencies** — 단일 HTML 파일, inline CSS/JS. npm/빌드 도구 없음.
2. **Show, Don't Tell** — 추상적 선택지 대신 시각적 프리뷰 생성 후 선택.
3. **Distinctive Design** — 제네릭 "AI slop" 금지. 커스텀 느낌의 디자인.
4. **Viewport Fitting** — 모든 슬라이드는 정확히 뷰포트에 맞춤. 스크롤 금지.
5. **꽉 찬 타이포그래피** — 텍스트는 슬라이드 공간을 넉넉하게 채운다. 여백에 묻히지 않는, 존재감 있는 글씨 크기. 제목은 bold하고 크게, 본문도 충분히 읽기 편한 사이즈로.

---

## Viewport Fitting (필수)

### 슬라이드별 콘텐츠 한도

| 슬라이드 타입 | 최대 콘텐츠 |
|--------------|------------|
| Title | 제목 1 + 부제 1 |
| Content | 제목 1 + 불릿 4-6개 (각 2줄 이내) |
| Feature grid | 제목 1 + 카드 6개 (2x3 또는 3x2) |
| Code | 제목 1 + 코드 8-10줄 |
| Quote | 인용 1 (3줄 이내) + 출처 |

**초과 시 → 슬라이드 분할. 스크롤 절대 금지.**

### 필수 CSS

```css
html, body { height: 100%; overflow-x: hidden; }
html { scroll-snap-type: y mandatory; scroll-behavior: smooth; }

.slide {
    width: 100vw;
    height: 100vh;
    height: 100dvh;
    overflow: hidden;
    scroll-snap-align: start;
    display: flex;
    flex-direction: column;
    position: relative;
}

.slide-content {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    max-height: 100%;
    overflow: hidden;
    padding: var(--slide-padding);
}

:root {
    --title-size: clamp(2rem, 6vw, 5rem);
    --h2-size: clamp(1.5rem, 4vw, 3rem);
    --body-size: clamp(0.875rem, 1.8vw, 1.25rem);
    --small-size: clamp(0.75rem, 1.2vw, 1rem);
    --slide-padding: clamp(1rem, 4vw, 4rem);
    --content-gap: clamp(0.5rem, 2vw, 2rem);
}

.card, .container { max-width: min(90vw, 1000px); max-height: min(80vh, 700px); }
img { max-width: 100%; max-height: min(50vh, 400px); object-fit: contain; }
.grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(min(100%, 220px), 1fr));
    gap: clamp(0.5rem, 1.5vw, 1rem);
}

@media (max-height: 700px) {
    :root { --slide-padding: clamp(0.75rem, 3vw, 2rem); --title-size: clamp(1.5rem, 5vw, 3rem); }
}
@media (max-height: 500px) {
    :root { --slide-padding: clamp(0.4rem, 2vw, 1rem); --title-size: clamp(1.25rem, 4vw, 2rem); }
    .nav-dots, .keyboard-hint, .decorative { display: none; }
}
@media (max-width: 600px) { .grid { grid-template-columns: 1fr; } }
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after { animation-duration: 0.01ms !important; transition-duration: 0.2s !important; }
}
```

---

## 작업 흐름

### Phase 0: 모드 판별

| 모드 | 조건 | 다음 단계 |
|------|------|----------|
| A: 신규 제작 | 처음부터 만들기 | Phase 1 |
| B: PPT 변환 | .ppt/.pptx 파일 있음 | Phase 4 |
| C: 기존 개선 | HTML 프레젠테이션 수정 | 파일 읽고 개선 |

### Phase 1: 콘텐츠 파악

AskUserQuestion으로 확인:
- **목적**: Pitch deck / 강의 / 컨퍼런스 발표 / 내부 발표
- **분량**: Short(5-10) / Medium(10-20) / Long(20+)
- **콘텐츠 준비 상태**: 완성 / 러프 노트 / 주제만 있음

### Phase 2: 스타일 탐색

**직접 선택**: 프리셋 이름 지정 시 바로 Phase 3으로. (프리셋 목록은 STYLE_PRESETS.md 참조)

**가이드 탐색 (기본)**:
1. 분위기 질문 (Impressed / Excited / Calm / Inspired, 복수 선택 가능)
2. 답변 기반 3개 스타일 프리뷰 HTML 생성 → `.claude-design/slide-previews/`
3. 각 프리뷰는 타이틀 슬라이드 1장, 인라인 CSS/JS, 50-100줄
4. 사용자에게 프리뷰 파일 안내 후 선택 받기

| 분위기 | 추천 프리셋 |
|--------|-----------|
| Impressed/Confident | Bold Signal, Electric Studio, Dark Botanical |
| Excited/Energized | Creative Voltage, Neon Cyber, Split Pastel |
| Calm/Focused | Notebook Tabs, Paper & Ink, Swiss Modern |
| Inspired/Moved | Dark Botanical, Vintage Editorial, Pastel Geometry |

### Phase 3: 프레젠테이션 생성

**파일 구조:**
```
[name].html          # 단일 파일 프레젠테이션
[name]-assets/       # 이미지 등 (필요 시)
```

**필수 JS 기능:**
- 키보드(화살표, 스페이스)/터치/휠 네비게이션
- Intersection Observer 기반 스크롤 애니메이션 (.visible 클래스 토글)
- 프로그레스 바 + 네비게이션 도트

**타이포그래피 (꽉 찬 글씨):**
- 제목: 슬라이드 폭의 70-90%를 채울 정도로 크게
- 본문: 최소 `clamp(0.875rem, 1.8vw, 1.25rem)` 이상
- 불릿/카드 텍스트도 충분한 사이즈 유지
- 여백보다 텍스트가 주인공

### Phase 4: PPT 변환

python-pptx로 슬라이드별 제목/텍스트/이미지 추출 → assets/ 저장 → 사용자 확인 → Phase 2로.

### Phase 5: 전달

1. `open [filename].html`로 브라우저 실행
2. 네비게이션 안내 (화살표, 스페이스, 스크롤/스와이프)
3. 커스터마이징 안내 (`:root` CSS 변수, 폰트 링크)

---

## 애니메이션 패턴

```css
.reveal { opacity: 0; transform: translateY(30px); transition: opacity 0.6s var(--ease-out-expo), transform 0.6s var(--ease-out-expo); }
.visible .reveal { opacity: 1; transform: translateY(0); }
.reveal:nth-child(1) { transition-delay: 0.1s; }
.reveal:nth-child(2) { transition-delay: 0.2s; }
.reveal:nth-child(3) { transition-delay: 0.3s; }

.reveal-scale { opacity: 0; transform: scale(0.9); }
.reveal-blur { opacity: 0; filter: blur(10px); }
```

**배경 효과:**
```css
/* Gradient Mesh */
background: radial-gradient(ellipse at 20% 80%, rgba(120,0,255,0.3) 0%, transparent 50%),
            radial-gradient(ellipse at 80% 20%, rgba(0,255,200,0.2) 0%, transparent 50%), var(--bg-primary);

/* Grid Pattern */
background-image: linear-gradient(rgba(255,255,255,0.03) 1px, transparent 1px),
                  linear-gradient(90deg, rgba(255,255,255,0.03) 1px, transparent 1px);
background-size: 50px 50px;
```

---

## 금지 패턴 (Generic AI Slop)

- **폰트**: Inter, Roboto, Arial, 시스템 폰트
- **색상**: `#6366f1` 제네릭 인디고, 보라 그라데이션 on 화이트
- **레이아웃**: 전부 가운데 정렬, 동일한 카드 그리드
- **장식**: 사실적 일러스트, 무분별한 글래스모피즘
