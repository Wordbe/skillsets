---
name: security-review
description: 기능 개발 후 보안 점검. OWASP Top 10, 인증/인가, 민감 데이터 노출, 인젝션 공격 등을 체계적으로 검사한다.
user-invocable: true
argument-hint: [파일 경로, 디렉토리, 또는 git diff 범위]
allowed-tools: Read, Bash, Glob, Grep, Agent
---

# Security Review

기능 개발 완료 후 코드의 보안 취약점을 체계적으로 점검하는 스킬.

## 작업 흐름

### Step 1: 변경 범위 파악

`$ARGUMENTS`로 전달된 대상을 확인한다.

| 입력 | 동작 |
|------|------|
| 파일/디렉토리 경로 | 해당 파일들을 직접 스캔 |
| `HEAD~N` 등 git 범위 | `git diff` 로 변경된 파일 목록 추출 |
| 없음 | `git diff --name-only HEAD~1` 로 최근 커밋 변경 파일 스캔 |

### Step 2: 보안 점검 체크리스트

아래 카테고리별로 코드를 분석한다. 각 항목은 **PASS / WARN / FAIL** 로 판정.

#### 2-1. 입력값 검증 (Input Validation)

- [ ] 사용자 입력이 서버에 도달하기 전 검증/새니타이징 되는가
- [ ] SQL 쿼리에 파라미터 바인딩을 사용하는가 (string interpolation 금지)
- [ ] 파일 경로 입력에 path traversal (`../`) 방어가 있는가
- [ ] URL/리다이렉트 입력에 allowlist 또는 origin 검증이 있는가
- [ ] 정규식이 ReDoS에 취약하지 않은가

#### 2-2. 인젝션 (Injection)

- [ ] **SQL Injection**: ORM 사용 또는 prepared statement
- [ ] **XSS**: 사용자 입력의 HTML 이스케이프, unsafe innerHTML 사용 여부
- [ ] **Command Injection**: `exec`, `spawn`, `system` 호출 시 입력 새니타이징
- [ ] **Template Injection**: 서버 사이드 템플릿에 사용자 입력 직접 삽입 여부
- [ ] **Header Injection**: HTTP 헤더에 사용자 입력 삽입 여부

#### 2-3. 인증/인가 (Auth)

- [ ] API 엔드포인트에 인증 미들웨어가 적용되어 있는가
- [ ] 권한 체크가 리소스 접근 전에 수행되는가 (IDOR 방어)
- [ ] JWT/세션 토큰의 유효성 검증이 올바른가
- [ ] 비밀번호 해싱에 bcrypt/scrypt/argon2를 사용하는가 (MD5/SHA 금지)
- [ ] Rate limiting이 로그인/API 엔드포인트에 적용되어 있는가

#### 2-4. 민감 데이터 (Sensitive Data)

- [ ] API 키, 비밀번호, 토큰이 코드에 하드코딩되어 있지 않은가
- [ ] `.env` 파일이 `.gitignore`에 포함되어 있는가
- [ ] 로그에 민감 정보(PII, 토큰, 비밀번호)가 출력되지 않는가
- [ ] 에러 응답에 스택 트레이스나 내부 정보가 노출되지 않는가
- [ ] 민감 데이터 전송 시 HTTPS를 사용하는가

#### 2-5. 설정/의존성 (Configuration)

- [ ] CORS 설정이 `*`가 아닌 특정 origin으로 제한되어 있는가
- [ ] 보안 헤더 설정 (CSP, X-Frame-Options, HSTS 등)
- [ ] npm/pip 등 의존성에 알려진 취약점이 없는가 (`npm audit`)
- [ ] 프로덕션에서 디버그 모드가 비활성화되어 있는가
- [ ] 불필요한 포트/서비스가 노출되지 않는가

#### 2-6. 파일/리소스 (File/Resource)

- [ ] 파일 업로드 시 확장자/MIME 타입/크기를 제한하는가
- [ ] 업로드된 파일이 실행 가능한 위치에 저장되지 않는가
- [ ] 임시 파일이 사용 후 삭제되는가
- [ ] 파일 접근 권한이 최소 권한 원칙을 따르는가

#### 2-7. 프론트엔드 (Frontend-specific)

- [ ] `localStorage`/`sessionStorage`에 토큰을 저장하지 않는가 (httpOnly cookie 선호)
- [ ] 외부 스크립트 로드 시 `integrity` 속성 사용
- [ ] `postMessage` 사용 시 origin 검증이 있는가
- [ ] 민감한 데이터가 브라우저 히스토리/URL에 노출되지 않는가

### Step 3: 결과 보고

아래 형식으로 보고서를 출력한다.

```
## Security Review Report

**대상**: [파일 목록 또는 커밋 범위]
**점검일**: [날짜]

### 요약

| 등급 | 건수 |
|------|------|
| FAIL (즉시 수정 필요) | N |
| WARN (권장 수정) | N |
| PASS | N |
| N/A (해당 없음) | N |

### FAIL 항목

#### [F-1] [카테고리] 제목
- **파일**: `path/to/file.ts:42`
- **위험도**: Critical / High / Medium
- **설명**: 무엇이 문제인지
- **코드**:
  // 취약한 코드 예시
- **수정 방안**:
  // 수정된 코드 예시

### WARN 항목

#### [W-1] [카테고리] 제목
- **파일**: `path/to/file.ts:15`
- **설명**: 왜 주의가 필요한지
- **권장 사항**: 어떻게 개선할 수 있는지

### PASS 항목 (요약)

- Input Validation: PASS
- SQL Injection: N/A (DB 미사용)
- ...
```

### Step 4: 수정 제안

FAIL 항목이 있을 경우:
1. 수정 코드를 제시한다
2. "보안 이슈를 수정할까요?" 라고 사용자에게 확인을 받는다
3. 승인 시 수정 적용 후 해당 부분만 재점검한다

## 주의사항

- `.env`, `.env.local` 등 환경변수 파일의 **내용**을 읽지 않는다 (존재 여부와 `.gitignore` 포함 여부만 확인)
- 민감 데이터를 보고서에 출력하지 않는다
- 오탐(false positive) 최소화: 실제 사용자 입력이 도달하는 경로만 추적
- 프레임워크 내장 보호 기능(예: React의 자동 이스케이프)을 인지하고 중복 경고하지 않는다
