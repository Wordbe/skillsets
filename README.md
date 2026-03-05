# Skillsets

Claude Code 플러그인 마켓플레이스 — 크리에이티브 콘텐츠 생성 및 보안 리뷰 스킬 모음.

## Plugins

| Plugin | Description |
|--------|-------------|
| [vibe-slides](plugins/vibe-slides/) | Zero-dependency HTML 프레젠테이션 생성 |
| [youtube-shorts](plugins/youtube-shorts/) | Remotion + Playwright 기반 YouTube Shorts 영상 생성 |
| [security-review](plugins/security-review/) | OWASP Top 10 기반 보안 리뷰 |

## Installation

### 마켓플레이스 등록 (전체 플러그인 설치)

```bash
claude plugin add --marketplace github:Wordbe/skillsets
```

### 개별 플러그인 설치

```bash
claude plugin add vibe-slides@skillsets
claude plugin add youtube-shorts@skillsets
claude plugin add security-review@skillsets
```

## Structure

```
plugins/<plugin-name>/
  .claude-plugin/plugin.json   # 플러그인 매니페스트
  skills/<skill-name>/SKILL.md # 스킬 정의
  README.md
```

## Contributing

새 플러그인을 추가하려면:

1. `plugins/<plugin-name>/` 디렉토리 생성
2. `.claude-plugin/plugin.json` 매니페스트 작성:
   ```json
   {
     "name": "your-plugin",
     "version": "1.0.0",
     "description": "What your plugin does"
   }
   ```
3. `skills/<skill-name>/SKILL.md` 스킬 정의 작성
4. `.claude-plugin/marketplace.json`에 플러그인 등록
5. `README.md` 추가
