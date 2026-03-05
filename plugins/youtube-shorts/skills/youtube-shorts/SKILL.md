---
name: youtube-shorts
description: Remotion + Playwright 기반 YouTube Shorts 영상 생성. 웹사이트 스크린샷 캡처 → React 컴포넌트 제작 → 무료 BGM 적용 → MP4 렌더링까지 전 과정을 자동화한다.
user-invocable: true
argument-hint: [주제 또는 URL]
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, WebFetch, WebSearch
---

# YouTube Shorts 영상 생성 Skill

Remotion + Playwright를 사용하여 YouTube Shorts 형식의 세로형(1080x1920) 프로모션 영상을 생성한다.
웹사이트 스크린샷 자동 캡처 → React 컴포넌트 제작 → 무료 BGM → MP4 렌더링 전체 파이프라인.

## 프로젝트 구조

```
studio/
├── src/
│   ├── index.ts              # Remotion 엔트리 포인트
│   ├── Root.tsx               # Composition 등록
│   └── ThisStoryShorts.tsx    # 참고 템플릿 (완성 예시)
├── public/
│   ├── bgm.mp3               # 배경 음악 (Pixabay 로열티 프리)
│   ├── chat.png              # 캡처된 스크린샷
│   ├── dashboard.png
│   └── landing.png
├── screenshots/               # Playwright 캡처 원본
├── out/                       # 렌더링 출력물
│   └── shorts.mp4
├── capture.mjs                # Playwright 스크린샷 캡처 스크립트
├── package.json
└── tsconfig.json
```

## 영상 사양

| 항목 | 값 |
|------|-----|
| 해상도 | 1080 x 1920 (세로, 9:16) |
| FPS | 30 |
| 기본 길이 | 15초 (450 프레임) |
| 코덱 | H.264 (video) + AAC (audio) |
| 포맷 | MP4 |

## 전체 작업 순서

### Step 1: 주제 확인
`$ARGUMENTS`로 전달된 주제/URL을 확인한다. 없으면 사용자에게 물어본다.

### Step 2: 웹사이트 스크린샷 캡처 (Playwright)

대상 웹사이트가 있으면 Playwright로 주요 화면을 자동 캡처한다.

```bash
cd studio
CAPTURE_EMAIL="..." CAPTURE_PASSWORD="..." node capture.mjs
```

**capture.mjs 패턴:**
```javascript
import { chromium } from "playwright";
const VIEWPORT = { width: 390, height: 844 }; // 모바일 사이즈
const browser = await chromium.launch({ headless: true });
const context = await browser.newContext({ viewport: VIEWPORT, deviceScaleFactor: 2 });
const page = await context.newPage();
// 페이지 이동 → waitForTimeout → screenshot
await page.goto(url, { waitUntil: "networkidle" });
await page.screenshot({ path: "screenshots/01-page.png" });
```

- 로그인이 필요하면 환경변수로 인증정보를 전달 (코드에 하드코딩 금지)
- `deviceScaleFactor: 2`로 레티나 품질 캡처
- 캡처한 이미지를 `studio/public/`에 복사하여 Remotion에서 `staticFile()`로 사용

### Step 3: 무료 BGM 다운로드

Pixabay에서 로열티 프리 음악을 다운로드한다.

```bash
curl -L -o studio/public/bgm.mp3 "<pixabay-cdn-url>"
```

- **출처**: Pixabay (https://pixabay.com/music/) - 로열티 프리, 저작권 무료
- 15초 이상 길이의 업비트/인스파이어링 트랙 선택
- ffmpeg로 자체 생성도 가능 (sine wave 합성):
  ```bash
  ffmpeg -y -f lavfi -i "sine=frequency=220:duration=15" ... -c:a libmp3lame studio/public/bgm.mp3
  ```

### Step 4: React 컴포넌트 작성

`studio/src/` 에 새 TSX 컴포넌트를 만든다.

**15초 (450프레임) 기준 5개 장면 구성:**

| 장면 | 프레임 | 시간 | 내용 |
|------|--------|------|------|
| Scene 1 | 0-89 | 0~3초 | 후킹 (관심 끌기) |
| Scene 2 | 90-179 | 3~6초 | 브랜드/서비스 소개 |
| Scene 3 | 180-269 | 6~9초 | 핵심 기능 1 (스크린샷) |
| Scene 4 | 270-359 | 9~12초 | 핵심 기능 2 (스크린샷) |
| Scene 5 | 360-449 | 12~15초 | CTA (행동 유도) |

**핵심 Remotion API:**
```tsx
import { AbsoluteFill, Img, Sequence, useCurrentFrame, useVideoConfig,
         spring, interpolate, staticFile } from "remotion";
import { Audio } from "@remotion/media";

// 애니메이션: spring() + interpolate() 조합
const scale = spring({ fps, frame, config: { damping: 12, mass: 0.8 } });
const opacity = interpolate(frame, [start, end], [0, 1], {
  extrapolateLeft: "clamp", extrapolateRight: "clamp",
});

// 오디오: 페이드인/아웃 볼륨 제어
const volume = interpolate(frame, [0, 15, 420, 450], [0, 0.45, 0.45, 0], {
  extrapolateLeft: "clamp", extrapolateRight: "clamp",
});
<Audio src={staticFile("bgm.mp3")} volume={volume} />

// 스크린샷 삽입
<Img src={staticFile("chat.png")} style={{ width: "100%" }} />
```

### Step 5: 디자인 가이드

**This Story 앱 색상 팔레트 (크림/오렌지 테마):**
```
배경: #FDF8F3 (크림), #F5EDE4 (다크 크림)
강조: #D4882B (오렌지), #E8A94E (라이트 오렌지)
텍스트: #2D2520 (다크 브라운), #7A6E64 (서브 텍스트)
```

**애니메이션 패턴:**
- 텍스트: `fadeIn` + `slideUp` (opacity + translateY)
- 카드/이미지: `spring scale` + `slideUp`
- Pill 태그: 순차적 `spring` (delay 적용)
- CTA 화살표: `interpolate(frame % 24, ...)` 바운스
- Glow: `Math.sin(frame * 0.1)` 펄스

**폰 목업 스타일:**
```tsx
style={{
  borderRadius: 36,
  overflow: "hidden",
  boxShadow: "0 24px 64px rgba(45,37,32,0.08), 0 10px 28px rgba(212,136,43,0.18)",
  border: "3px solid rgba(212,136,43,0.2)",
}}
```

### Step 6: Root.tsx에 등록
```tsx
<Composition
  id="VideoName"
  component={VideoComponent}
  durationInFrames={450}  // 15초
  fps={30}
  width={1080}
  height={1920}
/>
```

### Step 7: 렌더링
```bash
cd studio && npx remotion render src/index.ts <CompositionId> out/<filename>.mp4
```

### Step 8: 결과 전달
렌더링 완료 후:
1. 출력 파일 경로 안내
2. `ffprobe`로 영상 스펙 확인 (해상도, 길이, 오디오 존재 여부)
3. YouTube Shorts 제목 추천 (후킹형/공감형/트렌드형 각 2-3개)

## YouTube Shorts 제목 추천 패턴

| 유형 | 예시 |
|------|------|
| 후킹형 | "하루 5분이 인생을 바꾼다고?", "AI가 내 하루를 분석해줌ㅋㅋ" |
| 공감형 | "일기 쓰기 귀찮은 사람 여기여기 모여라", "AI한테 오늘 하루 얘기했을 뿐인데..." |
| 직관형 | "AI 일기장 앱 써봄", "5분 AI 대화로 하루 정리하는 법" |
| 트렌드형 | "요즘 매일 이거 하는 중", "나만 알고 싶은 자기계발 앱" |

## 필수 의존성

```bash
# studio/ 에서
npm install remotion @remotion/cli @remotion/bundler @remotion/media react react-dom typescript
npm install playwright  # 스크린샷 캡처용
npx playwright install chromium

# 시스템
brew install ffmpeg    # 오디오 생성/확인
brew install yt-dlp    # YouTube 오디오 다운로드 (선택)
```

## 참고 템플릿

`studio/src/ThisStoryShorts.tsx` - This Story 웹사이트 소개 15초 Shorts 완성 예시.
5개 장면, 실제 스크린샷, Pixabay BGM, spring/interpolate 애니메이션 포함.

## 주의사항

- 인증정보는 환경변수로만 전달 (코드에 직접 기재 금지)
- 이모지는 영상 내 아이콘으로 사용 가능
- 404 페이지 등 에러 화면은 영상에서 제외
- 렌더링 전 `npm install`이 studio/에서 실행되어 있어야 함
- BGM은 반드시 로열티 프리 (Pixabay, Mixkit 등) 사용
