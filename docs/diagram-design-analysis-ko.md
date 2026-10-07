# Diagram Design 전수조사 분석 및 활용 정리 (한국어)

> 이 문서는 `diagram-design` 저장소를 전수조사한 결과와, 설치·활용·수익화 방안을 한국어로 정리한 기록입니다.

## 저장소 주소

| 구분 | 주소 |
|---|---|
| **내 저장소 (fork)** | <https://github.com/bmshin94/diagram-design> |
| **원본 (upstream)** | <https://github.com/cathrynlavery/diagram-design> |
| 라이브 갤러리 | <https://cathrynlavery.github.io/diagram-design/> |
| 제작자 | Cathryn Lavery — <https://littlemight.com> / <https://bestself.co> |

- 작성일: 2026-10-07
- 분석 대상 버전: `2.6.27`
- 라이선스: **MIT** (상업적 이용·수정·재배포 자유, 저작권 고지 유지)

---

## 1. 한 줄 정의

> **AI 코딩 에이전트(Claude Code 등)에게 "전문 서적/잡지 수준의 다이어그램 그리는 법"을 가르치는 설명서 묶음(Agent Skill)입니다.**

코드 라이브러리가 아닙니다. 실행되는 프로그램이 거의 없습니다. 핵심은 **마크다운 문서 56개 + HTML 예제 165개**이고, AI가 이것을 읽고 직접 HTML/SVG를 생성합니다.

### 기본 정보 (실측)

| 항목 | 값 |
|---|---|
| 전체 용량 | 13MB / 파일 524개 |
| 외부 의존성 | 없음 (Python 3 표준 라이브러리만, PNG 내보낼 때만 Playwright 선택) |
| API 키 | **전혀 사용하지 않음** (전수 grep으로 확인) |
| 다이어그램 타입 | 40종 (README 표기는 39종 — 최근 추가분 반영 차이) |
| 지원 호스트 | Claude Code, Codex, GitHub Copilot, Factory Droid, Pi, Kiro, OpenCode, Cursor, Cline |

---

## 2. 폴더 구조 전수조사

```
diagram-design/
├── skills/diagram-design/        ★ 진짜 본체 (이것만 있어도 동작)
│   ├── SKILL.md         40KB   — 두뇌. 철학 + 타입 선택표 + 체크리스트
│   ├── references/      756KB  — 문서 56개. 타입별 "그리는 법" 설명서
│   ├── assets/          2.5MB  — HTML 165개. 완성 예제 + 템플릿 + 갤러리
│   └── scripts/         132KB  — Python 4개 (파서 3 + 자체검사 1)
│
├── commands/               — Claude Code 슬래시 명령 6개
├── prompts/                — Pi 에디터용 프롬프트 템플릿 5개
│
├── .claude-plugin/         — Claude Code 플러그인 매니페스트
├── .codex-plugin/          — OpenAI Codex용
├── .factory-plugin/        — Factory Droid용
├── .agents/plugins/        — 범용 Agent Skills 규격
│   └ → 같은 스킬 하나를 5~8개 AI 툴에 동시 배포하는 구조
│
├── scripts/              60+개  — CI 검증 스크립트 (품질 게이트)
├── docs/
│   ├── adr/               11개  — 설계 결정 기록(왜 이렇게 만들었나)
│   ├── cookbook.md              — 실무 레시피
│   └── screenshots/       40+장 — README용 썸네일 + 원본
└── .github/workflows/     3개   — Linux/Windows/macOS 3중 CI
```

### 핵심 구성요소 5가지

#### ① `SKILL.md` — 두뇌 (40KB)

- **철학**: *"최고의 디자인은 삭제다"* — 노드 밀도 목표 4/10, 9개 넘으면 두 장으로 분리
- **타입 선택표 (40종)**: "컴포넌트+연결선 → Architecture", "분기 로직 → Flowchart" 식 라우팅
- **의미 패턴 라우팅 9종**: 레이아웃보다 *행동*이 중요할 때 먼저 선택 (큐/병목, 정책 추적, 보안 경계, 생애주기 등)
- **SVG 원시 규칙**: 화살표 마커 3종 필수 정의, 라벨 마스크 필수, 커넥터 규칙
- **4px 그리드 강제**: 모든 좌표·너비·간격이 4의 배수 — *"AI가 만든 것처럼 안 보이게 하는 핵심"*
- **출력 전 체크리스트 (Taste Gate)**

#### ② `references/` — Progressive Disclosure (문서 56개)

이 프로젝트의 가장 영리한 설계입니다.

| 요청 | AI가 실제로 읽는 파일 |
|---|---|
| "플로우차트 만들어줘" | `SKILL.md` + `type-flowchart.md` — **딱 2개** |
| "이 정책 두 개가 왜 다른지 비교" | `SKILL.md` + `semantic-patterns.md` + `type-flowchart.md` |
| "애니메이션 추가" | 위 + `animation.md` |
| "내 사이트 브랜드 적용" | `SKILL.md` + `onboarding.md` + `style-guide.md` |

→ 타입이 40개든 100개든 AI는 필요한 1개만 읽습니다. 컨텍스트 낭비 0.

주요 문서:
- `style-guide.md` — **단일 진실 공급원(SSOT)**. 여기만 고치면 40종 전부 변경
- `semantic-patterns.md` — 레이아웃과 분리된 행동 패턴 9종
- `animation.md` — 선택적 모션 4모드(`none`/`reveal`/`step`/`loop`) + 접근성 계약
- `onboarding.md` — 웹사이트 URL → 브랜드 토큰 자동 추출
- `profiles.md` — 클라이언트별 브랜드 프로필 저장/전환
- `import-drawio.md` / `import-mermaid.md` / `import-excalidraw.md`
- `output-spec.md` — 포맷 × 사이즈 × 디테일 명세
- `type-*.md` 40개, `primitive-annotation / sketchy / terminal / icons`

#### ③ `assets/` — HTML 예제 165개

모든 타입이 3가지 변형(미니멀 라이트 / 다크 / 풀 에디토리얼)으로 완성 상태.
전부 **자체 완결형 단일 HTML** — 빌드 없음, JS 없음, 외부 이미지 없음. Google Fonts만 외부 참조.
`index.html`은 40종 전부를 탭으로 넘겨보는 라이브 갤러리.

#### ④ `scripts/` — Python 도구

스킬 내장 4개:

| 파일 | 줄 수 | 역할 |
|---|---|---|
| `drawio_extract.py` | 897 | draw.io → 구조화 IR. 압축 base64, `.drawio.png` 내장까지 파싱 |
| `mermaid_extract.py` | 1,355 | Mermaid 문법 파서 |
| `excalidraw_extract.py` | 706 | Excalidraw 씬 파서 |
| `self_check.py` | 457 | **AI가 자기 출력을 스스로 검증** |

> 보안 설계: 파서는 **텍스트만 읽습니다.** 렌더링·JS 실행·브라우저·네트워크·클릭 추적 전부 없음.

저장소 CI용 60여 개: `verify-geometry.py`, `verify-treemap.py`, `verify-waterfall.py`, `lint-render.py` 등.

#### ⑤ 멀티호스트 플러그인 매니페스트

```
.claude-plugin/plugin.json      ┐
.codex-plugin/plugin.json       ├─→ 전부 ./skills/diagram-design/ 를 가리킴
.factory-plugin/plugin.json     │
.agents/plugins/marketplace.json┘
```

타입 1개 추가 → 8개 툴에서 동시 사용. 유지보수 비용 1/8. (ADR-0008)

---

## 3. 디자인 시스템

```
색      : 액센트 1색만. 포커스 노드 최대 1~2개
폰트 3종 : Instrument Serif(제목/이탤릭 콜아웃)
          Geist Sans(노드 이름)
          Geist Mono(기술 서브라벨 — 포트/URL/필드타입에만)
테두리   : 1px 헤어라인. 그림자 금지. border-radius 최대 10px
그리드   : 모든 수치 4의 배수 — 타협 불가
밀도     : 4/10
```

기본 팔레트: `white-smoke #f5f5f5`(종이) / `jet-black #2d3142`(잉크) / `atomic-tangerine #eb6c36`(액센트) / `blue-slate #4f5d75`(보조)

**토큰은 전부 의미 기반.** 문서 어디에도 `#eb6c36`을 직접 쓰지 않고 `accent`라고만 씁니다.

### 흔한 AI 다이어그램과의 차이

| 흔한 AI 다이어그램 | 에디토리얼 |
|---|---|
| 알록달록 7색 | 검정 + 강조색 1개 |
| 둥근 모서리 + 그림자 | 1px 헤어라인, 그림자 0 |
| 박스 15개 꽉꽉 | 4~8개, 여백 넉넉 |
| 폰트 아무거나 | 세리프 제목 + 산스 본문 + 모노 기술값 |
| 선이 삐뚤 | 모든 좌표가 4의 배수 |
| 파워포인트 느낌 | 하버드 비즈니스 리뷰 삽화 느낌 |

---

## 4. 브랜드 온보딩 (60초)

```
나:   "onboard diagram-design to https://내사이트.com"
AI:   → 홈페이지 가져옴
      → 지배 색상 + 폰트 스택 추출
      → 의미 역할에 매핑 (paper / ink / muted / accent / link)
      → WCAG AA 명암비 자동 검증 (9~12px 기준 미달 시 조정안 제시)
      → 제안 diff 표시
나:   "적용해"
```

### 추출 매핑

| 사이트에서 감지 | 토큰 |
|---|---|
| `<body>` 배경색 | `paper` |
| 본문 텍스트색 | `ink` |
| 캡션/보조 텍스트 | `muted` |
| 카드/컨테이너 | `paper-2` |
| 가장 많이 쓰인 브랜드색(CTA/링크) | `accent` |
| `<h1>` 폰트 | `title` |
| `<body>` 폰트 | `node-name` |
| `<code>` 폰트 | `sublabel` |

- **첫 실행 게이트**: 스타일 가이드가 기본값이면 멈추고 물어봅니다 (브랜드 프로젝트에 기본 스킨을 몰래 넣지 않음)
- **클라이언트 프로필**: `~/.diagram-design/profiles/<slug>.md` + 프로젝트 `.diagram-design` 마커 → 여러 브랜드 병렬 작업 가능

---

## 5. Import = 변환이 아니라 "재작도"

```
/diagram-design:import-drawio platform.drawio --size=slide-16x9 --detail=simplified --audience=executive
/diagram-design:import-mermaid README.md --diagram=all
/diagram-design:import-excalidraw whiteboard.excalidraw
```

### 4개의 다이얼

| 다이얼 | 옵션 | 바뀌는 것 |
|---|---|---|
| **Format** | html · svg · png · html+png | 산출물 (SVG=Figma, PNG=슬라이드, HTML=웹) |
| **Size** | doc-inline · doc-wide · slide-16x9 · slide-4x3 · social-og · social-square · print-a4 · print-letter · fit | viewBox **+ 타입 램프** (프로젝터용은 16px 노드명) |
| **Detail** | faithful(≤24노드) · balanced(≤12) · simplified(≤7) | 고정 열화 래더: 장식 → 중복 → 리프 클러스터 → 인프라 |
| **Audience** | engineer · mixed · executive | **개수가 아니라 단어** |

Audience 예시:
```
engineer  : Auth Service / JWT · RS256 · :8443
mixed     : Auth Service / token check
executive : Sign-in
```

### 충실도 원장 (fidelity ledger)

```
Detail: balanced · 12 source nodes → 8 drawn
Collapsed: "Token valid?" 결정 → Gateway→Auth 엣지 라벨로
Dropped:   스티키 노트 1개 ("legacy path") — 원본에서 미연결
Kept in full: 요청 경로 (Web/Mobile → Gateway → Orders → Postgres)
```

- **절대 안 넘어오는 것**: 원본 좌표, 팔레트, 폰트, draw.io 대각선 스파게티, Mermaid 자동 레이아웃, Excalidraw 손떨림 기하
- **항상 넘어오는 것**: 컴포넌트, 관계, 그룹핑, 방향

---

## 6. 품질 보증 — 기계적 검증 게이트

Linux/Windows/macOS 3중 CI에서 동작:

| 게이트 | 검사 내용 |
|---|---|
| `verify-geometry.py` | 라벨 마스크가 나중 선언 노드와 겹치면 실패 (렌더 시 텍스트 잘림) |
| `verify-treemap.py` | 셀 실제 면적 비율이 표시 숫자와 불일치하면 실패. **상대 오차**로 측정 |
| `verify-waterfall.py` | 시작·증감·종료 합계 불일치, 브리지 바 레벨 오류, 부호 누락 시 실패 |
| `lint-render.py` | 헤드리스 크로미움 스크린샷 2장(원본/overflow 해제) **픽셀 diff**로 잘림 판정 |
| 접근성 린트 | `role="img"`, 해석 가능한 `aria-labelledby`, 첫 자식 `<title>`/`<desc>` 없으면 거부 |
| 모션 컨트롤러 핀 | 검토된 스크립트 1개만 허용. 수정 인라인 스크립트·원격자산·`@import`·`onclick`·`srcdoc` 거부 |
| `verify-docs-sync.py` | README 트리에 없는 파일 명기, 갤러리 도달 불가, 상대 링크 깨짐 시 실패 |

**검증기를 검증하는 테스트**가 또 있습니다: `test-verify-*.py`가 양방향으로 확인하고, `lint-render.py --self-test`는 23개 케이스 중 절반 이상이 "플래그되면 안 되는" 케이스입니다.

> 숫자를 **말하는** 게 아니라 **그림이 실제로 그 숫자인지** 검사합니다.

`docs/adr/` 11개에 설계 결정 근거가 기록되어 있습니다 — *"재논쟁 전에 읽고, 새 정책을 정하면 하나 추가하라."*

---

## 7. 언제 쓰고, 언제 쓰지 않는가

### 쓸 때
- 시스템 아키텍처 문서화 (기술 블로그, 사내 위키, README)
- 투자자 피치덱 — 비즈니스 모델/플라이휠/시장 포지셔닝
- 기술 문서 — API 시퀀스, DB 스키마, 배포 토폴로지, 의존성 그래프
- 컨설팅 산출물 — IT 현행 분석, 사분면 우선순위, Wardley 맵
- 블로그/뉴스레터 삽화
- 기존 draw.io/Mermaid 자산 전면 리디자인

### 쓰지 말 때 (README 명시)
- 트윗용 유니코드 다이어그램 → 다른 스킬
- 목록 → 표나 불릿
- 전/후 비교 → 표
- 도형 하나짜리 "다이어그램" → 그냥 문장으로

> *"그리기 전에 물어라: 독자가 잘 쓴 문단보다 이 그림에서 더 배우는가? 아니면 그리지 마라."*

### 나에게 주는 도움

| 문제 | 해결 방식 |
|---|---|
| Figma에서 30분 색 고르기 | 자연어 한 문장 → 완성 HTML |
| Mermaid 결과물이 촌스러움 | 같은 내용을 에디토리얼 품질로 재작도 |
| 내 블로그/서비스와 톤 불일치 | 사이트 URL 하나로 브랜드 자동 적용 |
| 다이어그램마다 스타일 상이 | 단일 스타일 가이드 → 40종 일관 |
| 슬라이드/문서/소셜 사이즈 재작업 | 같은 소스, Size 다이얼만 변경 |
| 접근성/명암비 여력 없음 | WCAG AA 자동 검증 + SVG 접근성 계약 기본 탑재 |
| AI 에이전트 만드는 법을 모름 | **이 저장소 자체가 Agent Skill 설계의 교과서** |

---

## 8. 설치 및 사용법

### 설치

**Claude Code (내 포크 기준):**
```text
/plugin marketplace add bmshin94/diagram-design
/plugin install diagram-design@diagram-design
```
설치 후 `/plugin` → **Marketplaces** → **diagram-design** → **Enable auto-update** 한 번 켜기
(Claude Code는 서드파티 마켓플레이스 자동 업데이트를 기본 비활성화)

**다른 호스트:**
```bash
# Codex
codex plugin marketplace add bmshin94/diagram-design
codex plugin add diagram-design@diagram-design

# GitHub Copilot
copilot plugin marketplace add bmshin94/diagram-design
copilot plugin install diagram-design@diagram-design

# Factory Droid
droid plugin marketplace add https://github.com/bmshin94/diagram-design
droid plugin install diagram-design@diagram-design --scope user

# Pi
pi install https://github.com/bmshin94/diagram-design
```

**Kiro:** 스킬 URL 임포트 → `https://github.com/bmshin94/diagram-design/tree/main/skills/diagram-design`

**편집 가능 설치 (스타일 가이드를 직접 수정하려면 — 추천):**
```bash
git clone https://github.com/bmshin94/diagram-design ~/code/diagram-design
ln -s ~/code/diagram-design/skills/diagram-design ~/.claude/skills/diagram-design
```
관리형 설치는 `references/style-guide.md` 수정이 업데이트로 덮일 수 있습니다.
단, `~/.diagram-design/profiles/`의 프로필과 `.diagram-design` 마커 프로젝트는 영향 없음.

### 사용법

**① 자연어 (99% 이걸로 충분)**
```
"우리 앱 아키텍처 다이어그램 만들어줘: 프론트엔드, 백엔드, DB, Redis 캐시"
"Q2 프로젝트를 임팩트 vs 노력 사분면으로 보여줘"
"401에서 토큰 갱신하는 베어러 호출 시퀀스 그려줘"
"이 drawio 파일 발표자료용으로 다시 그려줘"
```

**② 슬래시 명령 (Claude Code)**
```
/diagram-design:import-drawio platform.drawio --size=slide-16x9 --detail=simplified --audience=executive
/diagram-design:import-mermaid README.md --diagram=all
/diagram-design:import-excalidraw whiteboard.excalidraw
/diagram-design:export-diagram my-diagram.html --png-only --scale=3
/diagram-design:profile        # 클라이언트 브랜드 프로필 관리
/diagram-design:doctor         # 환경 진단
```

**③ 템플릿에서 시작**
```bash
cp skills/diagram-design/assets/template.html my.html         # 미니멀 라이트
cp skills/diagram-design/assets/template-full.html my.html    # 에디토리얼 + 요약카드
cp skills/diagram-design/assets/template-motion.html my.html  # 접근성 모션
```

**④ 갤러리 (설치 전 먼저 확인 권장)**
```bash
open skills/diagram-design/assets/index.html     # macOS
xdg-open skills/diagram-design/assets/index.html # Linux
start skills/diagram-design/assets/index.html    # Windows
```

### 권장 첫 순서
```
1. 갤러리 열어 40종 확인
2. 설치
3. "onboard diagram-design to https://내사이트.com"
4. "아키텍처 그려줘 ..." 로 첫 다이어그램
5. python3 skills/diagram-design/scripts/self_check.py my.html  → OK 확인
```

### PNG 내보내기만 추가 설치
```bash
pip install playwright && playwright install chromium
```
HTML 생성과 SVG 내보내기는 아무것도 필요 없습니다.

---

## 9. 플러그인 / 스킬 / MCP 구분

### 정답: **스킬(Agent Skill)**이 본질, **플러그인**은 배송 포장. **MCP는 아님.**

```
┌─────────────────────────────────────────────┐
│  플러그인 (포장재)                            │
│  .claude-plugin/ .codex-plugin/             │
│  .factory-plugin/ .agents/plugins/          │
│  + commands/ (슬래시 명령 6개)                │
│  + prompts/ (Pi 템플릿 5개)                  │
│  ┌───────────────────────────────────────┐  │
│  │  ★ 스킬 (내용물 — 진짜 본체)            │  │
│  │  skills/diagram-design/               │  │
│  │    SKILL.md + references/             │  │
│  │    + assets/ + scripts/               │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘

MCP 아님 — 서버 없음, 프로세스 없음, JSON-RPC 없음, 네트워크 없음
```

| | **스킬** | **플러그인** | **MCP** |
|---|---|---|---|
| 정체 | AI가 읽는 문서 묶음 | 스킬+명령어 배포 패키지 | 별도 프로세스 서버 |
| 실행 | AI가 읽고 스스로 행동 | (포장만) | 서버 프로세스 상주 |
| 통신 | 없음 | 없음 | stdio / HTTP, JSON-RPC |
| 설정 | 없음 | 매니페스트 | `mcp.json` + 명령/인자/env |
| 토큰/키 | 없음 | 없음 | 보통 필요 |
| 비유 | 설명서 | 택배 상자 | 외부 기계에 꽂는 플러그 |

MCP였다면 서버를 띄우고 호스트별 등록을 하고 버전 호환성을 관리해야 합니다. 스킬이라 그 전부가 없습니다 — 파일만 두면 끝.

---

## 10. API 토큰 — 필요 없음

저장소 전체를 `api_key|api_token|ANTHROPIC|OPENAI|Bearer` 패턴으로 전수 검색한 결과 **토큰을 요구하는 코드 0건**입니다. (걸린 것은 CI의 `npx @anthropic-ai/claude-code plugin validate` 한 줄과, "베어러 토큰 시퀀스 다이어그램 예제"라는 문서 텍스트뿐)

| | |
|---|---|
| 스킬 자체 | 마크다운 + HTML. 읽히기만 함 |
| Python 스크립트 4개 | 표준 라이브러리만. 네트워크 호출 0 |
| 생성 결과 HTML | 자체 완결형. 외부 요청은 Google Fonts CSS 한 줄뿐 |
| import 파서 | 텍스트 파싱만. 렌더링·JS·브라우저·네트워크·클릭추적 없음 |
| 아이콘 87개 | 저장소 내장 (`currentColor`) |

- **드는 비용**: 이미 내고 있는 AI 구독료뿐 (추가 비용 0)
- **네트워크 사용**: 브랜드 온보딩 때 **AI 에이전트가 자기 WebFetch로** 홈페이지를 읽음. 스킬이 직접 요청하지 않고 키도 불필요
- **선택 설치**: Playwright (PNG 전용, 로컬 크로미움, 무료·오프라인)
- 참고: 저장소 CI는 브라우저 리졸버 단계에서 네트워크를 차단(WebSocket 우회까지). `--fonts` 옵션만 Google Fonts 2개 호스트를 HTTPS로 허용

---

## 11. AI 에이전트 구축에 주는 도움 — 베낄 패턴 8가지

### ① Progressive Disclosure — 컨텍스트 폭발 방지
```
시작    : 스킬 이름 + description만 (수십 토큰)
요청매칭 : SKILL.md 로드 (40KB)
타입결정 : type-flowchart.md 1개만 추가
```
문서 56개 중 필요한 1~2개만 읽습니다. ADR-0004는 SKILL.md에 **바이트 상한**까지 걸었습니다.

### ② 의미 토큰 — 단일 진실 공급원
```
X  문서 40개에 #eb6c36 하드코딩
O  전부 accent 라고만 쓰고 style-guide.md 한 곳에서 정의
```

### ③ 트리거 풍부한 description
`plugin.json`의 description이 40개 타입을 전부 나열하는 것은 **의도**입니다 — 호스트가 사용자 요청과 매칭할 때 쓰는 유일한 텍스트이기 때문. `verify-docs-sync.py`는 타입의 어휘 훅이 description에서 빠지면 **CI를 실패시킵니다.**

### ④ 선언적 명령 정의
```yaml
---
description: ...
argument-hint: <html-file> [--svg-only|--png-only] [--scale=N] [--registry]
allowed-tools: [Read, Write, Edit, Bash, Glob]
---
```
+ Required behaviour로 실패 모드를 못 박음:
```
"소스 경로 없음 → 물어봐라. 추측 금지"
"갤러리 index.html → 거부하고 어느 파일인지 물어봐라"
"Playwright 미설치 → 설치 안내를 그대로 출력하고 멈춰라. 자동설치 금지"
```

### ⑤ 로직은 한 곳에만
명령 파일: *"그 레퍼런스를 진실 공급원으로 취급하라 — 여기서 로직을 재구현하지 마라."*
→ 명령 = 얇은 라우터, 로직 = 레퍼런스. 멀티 호스트에서 동작이 갈리지 않는 비결.

### ⑥ 기계적 검증 게이트 (가장 배울 점)
```
verify-treemap.py    : 면적이 숫자와 일치하나? (상대 오차로)
verify-waterfall.py  : 합계가 보존되나?
verify-geometry.py   : 라벨이 노드에 가려지나?
lint-render.py       : 실제 픽셀이 잘렸나? (스크린샷 2장 diff)
self_check.py        : 에이전트가 자기 출력을 스스로 검사
```
→ 에이전트가 숫자/구조를 주장하면 **그 주장을 검증하는 스크립트를 함께 출하**한다. 그리고 그 검증기를 양방향 테스트한다.

### ⑦ ADR — 결정 기록
"왜 컨트롤러를 하나로 핀 고정했나", "왜 의미 패턴이 타입 수를 늘리지 않나", "왜 라벨 배치를 기하학적으로 검증하나" 등 11개.

### ⑧ 멀티호스트 추상화
스킬 1개 → 8개 호스트. 매니페스트만 분리.

### 보안 설계도 교과서급
```
O 텍스트 파싱만 — 렌더링/JS/브라우저/네트워크/클릭추적 없음
O 리소스 상한 (resource caps)
O 적대적 픽스처로 테스트 (sample-adversarial.mmd / .excalidraw)
O 모션 컨트롤러 1개 핀 고정 — 수정판·원격자산·@import·onclick·srcdoc 거부
O 신뢰 경계 동작 명시
```

---

## 12. React / PHP로 만들 수 있는가 — 가능

**핵심:** 스킬 자체는 "AI가 읽는 문서"라 이식 대상이 아닙니다. 이식할 것은 **주변 래퍼**입니다.

| 만들 것 | 가능? | 현실 |
|---|---|---|
| 완성 HTML 보는 갤러리/뷰어 | 가능 (매우 쉬움) | 1~2일 |
| 다이어그램 편집기 UI | 가능 | 2~4주 |
| 내 사이트에서 AI에게 생성 요청 | 가능 | API 키 필요(비용 발생) |
| AI 없이 규칙 엔진으로 직접 그리기 | 가능하나 품질 하락 | 어려움 |
| 스킬 본체를 React로 "이식" | 의미 없음 | 스킬은 코드가 아니라 문서 |

### 추천 아키텍처
```
┌──────────── 프론트엔드 (React/Next.js) ────────────┐
│  40종 타입 피커 / 자연어·폼 입력                     │
│  브랜드 토큰 에디터 / 실시간 미리보기(iframe srcDoc)  │
│  라이트·다크·에디토리얼 탭 / HTML·SVG·PNG 다운로드    │
└────────────────────┬───────────────────────────────┘
                     │ POST /api/generate
┌────────────────────▼───────────────────────────────┐
│  백엔드 (Node/Express or Next API Routes)           │
│  1. SKILL.md + type-<선택>.md 읽기                  │
│  2. 사용자 브랜드 토큰 주입                          │
│  3. Claude API 호출 (system=스킬, user=요청)         │
│  4. self_check.py 로 검증  ← 그대로 재사용            │
│  5. 실패 시 1회 재시도 / 6. HTML 반환 + DB 저장        │
└────────────────────┬───────────────────────────────┘
         ┌───────────▼───────────┐
         │ Playwright → PNG/SVG   │
         │ Postgres → 저장/히스토리│
         └───────────────────────┘
```

### 기술 스택
```
프론트  : Next.js 15 + TypeScript + Tailwind, react-colorful, iframe srcDoc
AI      : claude-opus-5(최고 품질) / claude-sonnet-5-5(비용 균형), @anthropic-ai/sdk
검증    : self_check.py 그대로 (child_process) 또는 TS 포팅
내보내기: Playwright(PNG), SVG는 문자열 추출
저장    : Postgres(Supabase) + S3/R2
결제    : Stripe / 토스페이먼츠
```

### PHP (Laravel) 예시
```php
Route::post('/api/generate', function (Request $r) {
    $skill = Storage::get('skills/diagram-design/SKILL.md');
    $type  = Storage::get("skills/diagram-design/references/type-{$r->type}.md");
    $tokens = auth()->user()->brandTokens;

    $res = Http::withHeaders([
        'x-api-key' => config('services.anthropic.key'),
        'anthropic-version' => '2023-06-01',
    ])->post('https://api.anthropic.com/v1/messages', [
        'model' => 'claude-opus-5',
        'max_tokens' => 16000,
        'system' => $skill . "\n\n" . $type . "\n\n" . $tokens,
        'messages' => [['role' => 'user', 'content' => $r->prompt]],
    ]);

    $html = extractHtml($res->json());

    // 스킬에 포함된 검증기 그대로 재사용
    exec("python3 skills/diagram-design/scripts/self_check.py " . escapeshellarg($tmp), $out, $code);
    if ($code !== 0) { /* 재시도 */ }

    return response()->json(['html' => $html]);
});
```
PHP도 문제없습니다 — 하는 일이 "파일 읽기 → HTTP 호출 → 파일 저장"이라서요. Playwright만 Node 마이크로서비스로 분리하는 편이 편합니다.

### 반드시 알아야 할 것
1. **비용 구조 전환** — 스킬 설치는 공짜(내 구독), 웹앱은 내가 API 비용 부담. 생성 1건당 입력 20~40K + 출력 8~16K 토큰. **요금제/크레딧 설계 먼저**
2. **토큰 절감 = Progressive Disclosure 그대로** — `references/` 56개 전부 보내면 파산. 선택된 타입 1개만
3. **XSS 방어** — `self_check.py`가 `onclick`/`srcdoc`/원격자산/`@import`를 거부. 그대로 쓰고 추가로 `iframe sandbox` 격리
4. **라이선스** — MIT OK. `LICENSE` + `THIRD_PARTY_LICENSES.md` 고지 유지. 아이콘은 Tabler(MIT) + Simple Icons(CC0) 상업 사용 가능. 폰트 라이선스 재확인

### 2주 MVP
```
Week 1: Next.js + 타입 피커 + Claude API 연동 + 미리보기
Week 2: 브랜드 토큰 에디터 + HTML/SVG 다운로드 + Stripe
→ 무료 3개/월, 유료 $12/월 무제한
```
PNG 내보내기·히스토리·팀 공유는 나중. 먼저 "말하면 예쁜 그림 나온다"만 증명.

---

## 13. 유튜브 강의 영상 — 가능, 소재 우수

### 왜 좋은 소재인가

| 조건 | 평가 |
|---|---|
| 비주얼 임팩트 | ★★★★★ Before/After가 화면으로 바로 보임 |
| 썸네일 | ★★★★★ 촌스러운 그림 vs 세련된 그림 = 클릭률 |
| 진입 장벽 | ★★★★★ 설치 2줄, API 키 없음 |
| 체감 가치 | ★★★★★ "Figma 30분 → 1분" |
| 경쟁 | ★★★★ 한국어 콘텐츠 거의 없음 |
| 지속성 | ★★★★ 40종 = 시리즈 소재 무한 |

### 법적 체크 — 문제없음
- **MIT** → 상업적 사용·수정·2차 저작물 전부 허용
- 영상에서 출처(`cathrynlavery/diagram-design`) 명시 권장 (의무는 아니나 매너이자 신뢰)
- 저장소 스크린샷·예제 HTML 사용 가능
- 폰트(Instrument Serif/Geist), 아이콘(Tabler MIT / Simple Icons CC0) 상업 사용 가능
- 단, "내가 만들었다"고 하지 말 것 → "이 오픈소스를 활용하는 방법"으로

### 단편 구성 (8~12분, 조회수용)
```
제목: "Claude Code로 1분 만에 전문 서적급 다이어그램 만들기 (무료, API키 X)"

0:00  훅 — 촌스러운 Mermaid vs 에디토리얼 (좌우 비교)
0:40  이게 뭔지 30초 설명
1:10  설치 (명령어 2줄, 화면 그대로)
2:00  첫 다이어그램 — 자연어로 아키텍처
3:30  브랜드 온보딩 — 내 사이트 URL 하나로 색/폰트 입히기
5:30  Before/After — 기존 drawio를 발표자료용으로 재작도
7:30  내보내기 (SVG/PNG)
8:30  40종 갤러리 빠른 투어
9:30  언제 쓰지 말아야 하나 (신뢰도 상승)
10:00 마무리 + 저장소 링크
```
썸네일: 좌 "AI가 만든 다이어그램"(촌스러움) / 우 "이 스킬로"(세련) / 중앙 큰 화살표

### 시리즈 구성 (10~15편, 수익용)
```
EP01  Agent Skill이 뭔가 — 플러그인/MCP와 차이
EP02  설치와 첫 다이어그램
EP03  브랜드 온보딩 완전 정복
EP04  40종 전체 투어 + 선택 가이드
EP05  아키텍처 다이어그램 실전
EP06  시퀀스 / 상태머신 / DB 스키마 (개발자편)
EP07  사분면 / Wardley / 워터폴 (비즈니스편)
EP08  draw.io / Mermaid / Excalidraw 재작도
EP09  4개 다이얼 — 발표자료·문서·소셜 맞춤
EP10  내보내기와 Figma 연동
EP11  ★ 스킬 구조 해부 — Progressive Disclosure
EP12  ★ 나만의 다이어그램 타입 추가하기
EP13  ★ 이 구조로 내 Agent Skill 만들기
EP14  ★ 기계적 검증 게이트 설계하기
EP15  클라이언트 프로필로 외주 작업 자동화
```
**EP11~14가 진짜 수익 구간** — "다이어그램 도구 소개"는 조회수, "Agent Skill 설계론"은 유료 강의.

### 숏폼 (15~40초)
```
"Mermaid 다이어그램이 촌스러운 이유" (4px 그리드)
"사이트 URL 하나로 다이어그램 브랜딩하는 법"
"AI가 만든 트리맵, 면적이 숫자랑 다른 거 알아요?" (검증 게이트)
"Figma 30분 vs AI 60초"
"Claude Code 플러그인 설치 2줄"
```

### 제작 팁
| | |
|---|---|
| 화면 녹화 | 터미널 폰트 16pt 이상 (모바일 시청자) |
| 속도 | 설치/대기 구간 2~4배속 + 자막 |
| Before/After | 반드시 좌우 분할 화면 — 이 콘텐츠의 전부 |
| 갤러리 활용 | `index.html` 스크롤 + 탭 전환 = 무료 B롤 |
| 실수 노출 | 첫 실행 게이트가 멈추는 장면 그대로 → 신뢰도 상승 |
| 자산 제공 | 프롬프트 모음 노션/깃허브 배포 → 구독 전환 |

### 수익 경로
```
애드센스 → 멤버십/슈퍼땡스 → 유료 강의 → 컨설팅/외주 유입 → 제휴 → 디지털 상품
```

### 가장 차별화되는 앵글
> "다이어그램 도구 리뷰"는 수십 개 나옵니다. 하지만 **"이 저장소를 해부해서 Agent Skill 설계법을 배운다"**는 거의 없습니다. Progressive Disclosure, 의미 토큰, ADR, 기계적 검증 게이트, 멀티호스트 배포를 한국어로 제대로 설명하는 채널이 지금 없습니다.

---

## 14. 수익화 아이디어 상세

### 한눈에 보기

| # | 아이디어 | 초기비용 | 시작까지 | 월 잠재수익 | 난이도 |
|---|---|---|---|---|---|
| 1 | 브랜드 다이어그램 **대행 서비스** | 0원 | 1주 | 100~500만 | ★ |
| 2 | **교육 콘텐츠** (유튜브→강의) | 0원 | 2주 | 50~1,000만 | ★★ |
| 3 | **수직 산업 스킬 팩** | 0원 | 3주 | 30~300만 | ★★ |
| 4 | **SaaS 웹 서비스** | 50~200만 | 6주 | 100~2,000만 | ★★★★ |
| 5 | **브랜드 프로필 팩** 판매 | 0원 | 1주 | 10~100만 | ★ |
| 6 | **기업 디자인 시스템** 컨설팅 | 0원 | 4주 | 300~2,000만 | ★★★ |
| 7 | **문서화 자동화 파이프라인** | 0원 | 4주 | 200~1,000만 | ★★★ |
| 8 | **피치덱 다이어그램** 전문 | 0원 | 1주 | 100~500만 | ★ |

> 수익 추정치는 시장 조건·역량·영업력에 따라 크게 달라지는 가정값입니다.

---

### 1. 브랜드 다이어그램 대행 서비스 (가장 빠름)

**파는 것:** "당신 브랜드 색/폰트가 적용된 전문가급 다이어그램"
**플랫폼:** Fiverr, Upwork, 크몽, 숨고

```
BASIC     $49  / 6만원  — 1종, HTML+PNG, 수정 1회, 2일
STANDARD  $149 / 18만원 — 3종 + 브랜드 온보딩, SVG 포함, 수정 2회, 3일
PREMIUM   $399 / 48만원 — 8종 + 프로필 저장 + 다크모드 + 슬라이드 리사이즈, 수정 무제한, 5일
RETAINER  $800/월 /96만원 — 월 10종, 48시간 턴어라운드
```

**이익률:**
```
고객 체감     : "Figma로 하면 건당 2~4시간"
실제 소요     : 15~30분
브랜드 온보딩 : 60초 (한 번만, 이후 전부 재사용)
→ 시간당 20~60만원
```

**결정적 무기 — 클라이언트 프로필:**
```
~/.diagram-design/profiles/acme.md
~/.diagram-design/profiles/beta-corp.md
~/.diagram-design/profiles/gamma.md
```
프로젝트에 `.diagram-design` 마커 하나 → 자동 전환. 30개 클라이언트 병렬 작업에도 안 섞입니다.

**포트폴리오:** `assets/` 165개 예제 + `docs/screenshots/` 40장 → 첫 고객 전에 이미 포트폴리오 보유.

**첫 주 체크리스트:**
```
Day 1   설치 + 갤러리 전체 확인
Day 2   가상 브랜드 3개로 샘플 12장 생성
Day 3   Fiverr/크몽 리스팅 (Before/After 썸네일 핵심)
Day 4   LinkedIn/X 샘플 공개 + "DM 주세요"
Day 5   첫 주문 처리 → 리뷰 요청
Day 6-7 리스팅 최적화 + 가격 조정
```

---

### 2. 교육 콘텐츠 (복리 효과 최고)

```
1단: 유튜브 무료 (유입)       → 애드센스 + 구독자
2단: 유료 강의 (수익)         → "Agent Skill 설계 마스터클래스" 199,000원
3단: 컨설팅 (고단가)          → 건당 300~2,000만원
```

**2단에서 가르칠 내용:**
```
① Progressive Disclosure — 문서 56개 중 1개만 읽히는 구조
② 의미 토큰 SSOT — 40종을 한 파일로 제어
③ 트리거 풍부한 description — 그리고 그걸 CI로 지키는 법
④ 기계적 검증 게이트 — AI "그럴듯한 거짓말" 잡는 법
⑤ 검증기를 검증하는 테스트 — 양방향 신뢰
⑥ ADR — 결정 기록으로 재논쟁 방지
⑦ 멀티호스트 — 스킬 1개 → 8개 툴
⑧ 보안 경계 — 신뢰할 수 없는 입력 처리
```
→ 다이어그램 강의가 아니라 **"AI 에이전트 아키텍처 강의"**. 단가 3~5배.

**예상 (보수적):**
```
유튜브 1만 구독 → 애드센스 월 30~80만
강의 199,000원 × 월 20명 → 400만
컨설팅 분기 1건 × 500만 → 월 환산 170만
───────────────────────────────
월 600~1,000만원
```

---

### 3. 수직 산업 스킬 팩

`references/type-*.md` 패턴을 복사해 산업별 타입 추가.

| 산업 | 추가할 타입 | 고객 |
|---|---|---|
| 의료 | 임상 경로, 환자 여정, HIPAA 데이터 흐름, 임상시험 플로우 | 병원, 헬스테크, 제약 |
| 금융 | 결제 정산 흐름, 리스크 매트릭스, 규제 보고 체인, KYC/AML | 은행, 핀테크, 보험 |
| 법률 | 계약 구조, 지분/소유 구조, 분쟁 타임라인, 규제 매핑 | 로펌, 법무팀 |
| 제조 | 공정 흐름, 공급망 맵, OEE 대시보드, 품질 게이트 | 제조사, 물류 |
| 교육 | 커리큘럼 맵, 학습 경로, 역량 체계 | 학교, 에듀테크 |
| 보안 | 위협 모델(STRIDE), 공격 트리, 제로트러스트 토폴로지 | 보안팀, SI |

**모델:**
```
A. 유료 마켓플레이스 스킬    $29~99 (일회성)
B. 산업팩 구독              $19/월
C. 기업 커스텀 제작         300~1,000만원
D. 무료 공개 + 컨설팅 유입   간접
```

**난이도가 낮은 이유:** 패턴 완전 정립(`type-*.md` 40개가 샘플), ADR-0002가 확장 규칙 명시, CI 검증 스크립트 재사용, MIT로 상업 배포 합법.

> 원본 저작권 고지 유지 + "cathrynlavery/diagram-design 기반" 명시가 윤리적·신뢰 측면에서 유리.

---

### 4. SaaS 웹 서비스 (상한 최대, 리스크 최대)

**해결 문제:** "Claude Code를 못/안 쓰는 95%" — 디자이너, 기획자, 마케터, PM, 컨설턴트, 창업자

```
FREE   0원      월 3개, 워터마크, 기본 스킨
PRO    $19/월   월 100개, 브랜드 1개, SVG/PNG, 히스토리
TEAM   $49/월   무제한, 브랜드 5개, 팀 공유, API
AGENCY $149/월  브랜드 무제한, 화이트라벨, 우선 지원
```

**원가 관리 (핵심):**
```
생성 1건: 입력 20~40K + 출력 8~16K 토큰
→ PRO 100개/월은 원가가 요금을 넘을 수 있음

대응:
① Progressive Disclosure 철저히 — 타입 1개 레퍼런스만 전송
② 프롬프트 캐싱 — SKILL.md는 고정 → 캐시 적중률 매우 높음  ★결정적
③ 크레딧 제도 (무제한 금지)
④ 티어별 모델 분리 — FREE=Haiku, PRO=Sonnet, TEAM=Opus
⑤ 동일 요청 결과 캐싱
```

**6주 로드맵:**
```
Week 1-2  Next.js + 타입 피커 + Claude API + iframe 미리보기
Week 3    브랜드 온보딩 (URL 파싱 → 토큰 추출 → 명암비 검증)
Week 4    SVG/PNG 내보내기 (Playwright 마이크로서비스)
Week 5    Stripe/토스 결제 + 요금제 + 크레딧
Week 6    랜딩페이지 + Product Hunt 런치
```

**리스크:** API 원가 폭주(→크레딧·캐싱), 원본 저자의 직접 SaaS 출시(→수직 특화·통합으로 차별화), AI 품질 변동(→`self_check.py` 실패 시 자동 재시도 + 무료 재생성)

---

### 5. 브랜드 프로필 팩 판매 (가장 쉬움)

```
"SaaS 스타트업 팩"     — Linear / Stripe / Vercel 풍 10종   $19
"컨설팅 팩"           — McKinsey / BCG / Deloitte 풍 8종    $29
"K-테크 팩"           — 토스 / 카카오 / 네이버 풍 8종       ₩29,000
"다크모드 프리미엄 팩"  — 다크 전용 12종                     $19
"인쇄용 팩"           — CMYK 안전 팔레트 10종              $24
```
제작 비용 거의 0 (색 토큰 + 폰트 + 명암비 검증 + 샘플 3장).
판매처: Gumroad, Lemon Squeezy, 크몽, 자체 사이트.
업셀: 팩 구매자 → 대행(#1) 또는 강의(#2).

> 주의: 특정 브랜드 실제 색을 그대로 복제해 "토스 팩"으로 파는 것은 상표권 리스크. **"~풍/inspired"**로 표현하고 색은 유사하되 동일하지 않게 조정.

---

### 6. 기업 디자인 시스템 컨설팅 (단가 최상위)

```
패키지: 기업 다이어그램 시스템 구축       1,000~3,000만원
├─ 브랜드 가이드 → 다이어그램 토큰 매핑
├─ 사내 전용 타입 추가 (그 회사 도메인)
├─ 기존 자산 마이그레이션 (draw.io/Visio/Mermaid 수백 장)
├─ 사내 Agent Skill 패키징 + 배포
├─ CI 검증 게이트 구축 (저장소 패턴 이식)
├─ 엔지니어 교육 (반일 워크숍)
└─ 3개월 유지보수
```

**타겟:** 기술 블로그 운영 테크 기업, SI/컨설팅펌, 금융권(문서 규정 엄격), 제약/의료(규제 문서)

**영업 포인트:**
- "다이어그램 1장당 2시간 × 연 500장 = 1,000시간 → 100시간으로"
- "모든 문서 다이어그램이 브랜드 가이드를 자동 준수"
- "접근성(WCAG AA) 자동 검증 — 공공/금융 조달 요건 충족"
- "**기계적 검증 게이트**로 다이어그램 숫자가 실제와 일치함을 보증" ← 금융/의료에서 특히 강력

**리테이너:** 월 200~500만원

---

### 7. 문서화 자동화 파이프라인 (가장 끈끈함)

```
┌─ GitHub Actions ─────────────────────────┐
│ on: push                                 │
│ 1. 코드/스키마/IaC 분석                   │
│ 2. 변경 감지                              │
│ 3. diagram-design 스킬로 재생성            │
│ 4. self_check.py 검증                     │
│ 5. docs/ 커밋 + PR에 미리보기 코멘트        │
└──────────────────────────────────────────┘
```

| 소스 | → 다이어그램 |
|---|---|
| `schema.prisma` / DDL | DB 스키마, ER |
| `openapi.yaml` | 시퀀스, 아키텍처 |
| `docker-compose.yml` | 배포 토폴로지 |
| Terraform / CloudFormation | 인프라 아키텍처 |
| `package.json` / `go.mod` | 의존성 그래프 |
| CI 워크플로 | 플로우차트 |
| 깃 히스토리 | 타임라인, 간트 |

```
GitHub App / Action      $29~99/월 per repo
온프레미스 라이선스        연 1,000~3,000만원
구축 컨설팅               500~2,000만원
```
한 번 CI에 박히면 안 뺍니다 — 이탈률 극히 낮음.
> `verify-docs-sync.py`가 이미 "문서와 실제 파일이 어긋나면 CI 실패"를 구현해뒀습니다. 그 아이디어의 제품화입니다.

---

### 8. 피치덱 다이어그램 전문 (니치, 고단가, 경쟁 적음)

```
스타트업 피치덱 비주얼 패키지                 150~500만원
├─ 비즈니스 모델 다이어그램
├─ 제품 아키텍처 (audience=executive)
├─ 시장 포지셔닝 (사분면)
├─ 성장 플라이휠 (Loop — 공유 메모리 허브)
├─ 경쟁 비교 (레이더 차트)
├─ 로드맵 (간트 / 타임라인)
├─ 유닛 이코노믹스 (워터폴 — 합계 보존 검증)
├─ 고객 여정 (User Journey)
└─ 전부 16:9 슬라이드 + SVG(Figma) + PNG(3x)
```

**무기 ①** `--audience=executive` 다이얼 — 같은 아키텍처를 투자자 눈높이로 자동 재작성. 기술 창업자가 가장 못하는 일.
**무기 ②** 워터폴 검증 — 유닛 이코노믹스 합계가 실제로 맞는지 CI가 검증. 투자자 앞에서 숫자 안 틀림.

**타겟:** 시드~시리즈A 스타트업, 액셀러레이터/VC, IR 대행사
**영업:** 데모데이 참석, 액셀러레이터 파트너십, 창업 커뮤니티

---

### 전략 권고 — 단계별 로드맵

```
Phase 1 (1~4주) — 현금흐름 확보, 리스크 0
  #1 대행 + #8 피치덱 동시 시작
  → 월 100~300만원, 투자금 0원, 동시에 실제 고객 니즈 학습

Phase 2 (2~3개월) — 레버리지
  #2 교육 콘텐츠 (Phase 1 실적을 증거로)
  #5 프로필 팩 (Phase 1 작업물 재활용)
  → 월 300~600만원

Phase 3 (4~6개월) — 확장
  #3 수직 팩 (Phase 1에서 수요 많던 산업 선택)
  #6 컨설팅 (Phase 2 수강생 중 기업 담당자 전환)
  → 월 600~1,500만원

Phase 4 (6개월+) — 제품화
  #4 SaaS 또는 #7 파이프라인
  → Phase 1~3에서 검증된 수요 기반으로만 진입
```

**순서를 지켜야 하는 이유:** #4 SaaS를 먼저 하면 API 원가 구조, 실제 수요 타입, 지불 의사를 모르고 6주를 씁니다. #1 대행을 먼저 하면 2주 안에 (a) 어떤 타입이 90% 수요인지 (b) 사람들이 얼마 내는지 (c) 포트폴리오 (d) 현금이 확보됩니다.

**가장 저평가된 기회:** #2의 "Agent Skill 설계론" 파트. 다이어그램 도구는 대체재가 생기지만, "잘 만든 Agent Skill을 어떻게 설계하는가"는 앞으로 수요가 폭발할 영역이고 이 저장소가 완벽한 교재입니다. 한국어로 이를 제대로 가르치는 사람이 아직 없습니다.

---

## 15. 핵심 요약

> **"AI에게 디자인 감각을 주입하는 설명서 세트. 설치하고 한국말로 시키면, 내 브랜드 색으로 책에 실릴 수준의 그림을 HTML 파일로 뽑아준다."**

| 질문 | 답 |
|---|---|
| 이게 뭔가? | Agent Skill (AI가 읽는 문서 묶음) |
| 플러그인/스킬/MCP? | **스킬**이 본질, 플러그인은 포장, MCP 아님 |
| API 토큰 필요? | **불필요** |
| 설치 난이도? | 명령어 2줄 |
| 상업적 이용? | MIT — 자유 (고지 유지) |
| 에이전트 구축에 도움? | 매우 큼 — 설계 패턴 교과서 |
| React/PHP 가능? | 래퍼는 가능, 스킬 본체는 이식 대상 아님 |
| 유튜브 소재? | 우수 — Before/After 비주얼 + 한국어 콘텐츠 공백 |
| 수익화? | 8가지 경로, #1 대행부터 시작 권장 |

---

*전수조사 및 정리: Claude Code / 2026-10-07*
*저장소: <https://github.com/bmshin94/diagram-design>*
