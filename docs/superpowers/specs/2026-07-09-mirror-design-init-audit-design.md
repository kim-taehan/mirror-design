# mirror-design: 통합 워크스페이스 + init/audit 커맨드 설계

날짜: 2026-07-09 · 상태: 사용자 리뷰 대기

## 배경

interface-design(Dammyjay93) 분석에서 가져오기로 확정한 구조적 장치 중, 이번 사이클의 범위는 세 가지다:
① 산출물 워크스페이스 통합, ② `/mirror-design:init` 커맨드, ③ `/mirror-design:audit` 커맨드.

가져오지 **않는** 것(명시적 제외): plan/implement 커맨드 분리, census 완성 예시 동봉, Decisions 테이블 형식, 커뮤니케이션 규율 문구, 그리고 interface-design의 디자인 원칙 일체(타이포 비율, 60/30/10 등 — mirror-design의 "창작 금지" 첫 원칙과 충돌).

판단 원칙은 유지된다: **모든 판단의 근거는 취향이 아니라 census의 측정 증거(file:line + 사용 빈도)다.**

## 1. 통합 워크스페이스

대상 저장소에 mirror-design이 만드는 모든 산출물을 `.mirror-design/` 하나로 통합한다.

```
.mirror-design/
├── .gitignore          # 내용: "plan/" — 커밋 정책을 도구가 스스로 보장
├── census.md           # 커밋✅ 측정 조사 (팀 공유 자산)
├── mockup-chrome.html  # 커밋✅ repo당 1회 만드는 공용 페이지 크롬
└── plan/               # 커밋❌ 목업 HTML · 스크린샷 PNG · 플랜 PDF · audit 목업
```

변경 사항:
- `SKILL.md` · `references/mockup-guide.md` · `README.md`의 `mirror-design-plan/` 경로를 전부 `.mirror-design/plan/`으로 교체.
- "루트 .gitignore에 추가 권장" 안내는 삭제 — 내부 `.gitignore`가 대신하므로 사용자 할 일이 없어진다.
- 이 저장소 자체의 `.gitignore`에서 `mirror-design-plan/` 항목 제거.

## 2. `/mirror-design:init` — commands/init.md

스캐폴드 + 센서스. 온보딩과 갱신의 명시적 진입점. **디자인 취향은 아무것도 묻지 않는다** — 답은 코드에 있다.

동작 순서:
1. **전제 확인** — 프론트엔드 repo인지(페이지 디렉토리·라우팅 존재) 확인. 아니면 경로를 묻고 중단.
2. **스캐폴드** — `.mirror-design/` 생성 + 내부 `.gitignore`(`plan/`) 작성.
3. **센서스** — `references/census.md` 지침대로 조사해 `census.md` 작성.
   - 이미 있으면: 전체 재조사 대신 file:line 스팟체크로 드리프트된 섹션만 갱신 (기존 캐시 규율 그대로).
4. **보고** — 요약(스택, 토큰 수, 지배 스켈레톤, 상위 공유 컴포넌트) + **트랩 목록 강조** + `census.md` 커밋 권장.

자동 경로 유지: init 없이 화면을 요청해도 스킬이 지금처럼 census를 자동 구축한다(zero-config). init은 선택적 진입점이다.

## 3. `/mirror-design:audit` — commands/audit.md

기존/신규 화면이 census 기준에서 이탈했는지 감사한다. interface-design design-review의 골격(심각도 + false-positive 필터 + 판정만)을 가져오되, **판단 근거를 취향에서 census 증거로 교체**한다.

동작 순서:
1. **census 로드** — 없으면 "`/mirror-design:init`을 먼저 실행하세요" 안내 후 중단.
2. **스코프 확정** — 인수로 받은 파일/디렉토리. 인수 없으면: git 작업 트리에 diff가 있으면 그 diff, 없으면 한 가지 질문으로 범위 확인.
3. **증거 대조 패스** — 스코프의 각 UI 요소를 census 지배 패턴과 대조. 지적 후보마다 양쪽 file:line(화면의 값 ↔ census 근거)을 확보.
4. **심각도 분류**
   - **Blocker** — 발명: census에 선례 없는 새 패턴/컨트롤 도입, 토큰 대신 하드코딩 값.
   - **Should-fix** — 지배 패턴이 따로 있는데 소수 패턴 사용, 형제 일관성(sibling-consistency) 위반.
   - **Note** — 경미한 이탈.
5. **False-positive 필터**
   - census가 기록한 정당한 변형은 위반이 아니다.
   - 스코프 밖 제외.
   - **census 근거를 못 대는 지적은 버린다** — audit 자신도 발명 금지. "내가 보기에 이상함"은 무효.
   - lint/포맷 소관 제외.
6. **판정 보고 (목업 필수)**
   - **Before/After 대조 목업**을 `.mirror-design/plan/<screen>-audit.html`에 항상 생성: 현재 화면 재현(위반 요소에 `DRIFT` 배지) ↔ census 기준으로 고친 모습을 나란히. 하단에 판정 테이블 임베드.
   - 목업의 모든 스타일 값은 실측값만 사용한다(기존 mockup-guide 규칙 그대로). After 판의 값은 census 근거 값.
   - 배지 체계 확장: `NEW` / `CHANGED` / **`DRIFT`** (audit 전용, 시각적으로 구분).
   - 텍스트 보고: 표(요소 | 화면의 값 | census 기준 | 근거 file:line | 심각도) + 종합 판정("Blocker 0 → 통과").
   - **수정은 하지 않는다** — 판정만. 사용자가 수정을 요청하면 기존 Phase 1.5 수정 루프로 진입한다: 같은 경로의 목업을 덮어쓰며(화면을 보면서 반복 변경), 승인 후 구현.

## 파일 변경 목록

| 파일 | 변경 |
|---|---|
| `commands/init.md` | 신규 |
| `commands/audit.md` | 신규 |
| `skills/mirror-design/SKILL.md` | 경로 통합, 커맨드 언급 추가, 자연어 폴백 한 줄("초기화/감사를 자연어로 요청받으면 해당 커맨드와 동일하게 수행") |
| `skills/mirror-design/references/mockup-guide.md` | 경로 통합, `DRIFT` 배지 추가 |
| `README.md` | 경로 통합, 커맨드 2개 소개 |
| `.gitignore` (이 repo) | `mirror-design-plan/` 제거 |
| `.claude-plugin/plugin.json` | `version` 0.1.0 → 0.2.0 |
| `CHANGELOG.md` | 0.2.0 항목 추가 |

## 검증 기준

1. `claude plugin validate .` 통과.
2. 프로젝트 전체에서 `mirror-design-plan` 문자열 0건.
3. 실제 프론트엔드 repo에서 `/mirror-design:init` 실행 → `.mirror-design/` 구조와 census.md 생성, 트랩 보고 확인.
4. 같은 repo에서 화면 하나를 일부러 이탈시킨 뒤 `/mirror-design:audit <파일>` 실행 → DRIFT 배지가 있는 before/after 목업 + census 근거가 붙은 판정 테이블 확인.
