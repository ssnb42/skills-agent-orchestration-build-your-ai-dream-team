# Project Pulse 구현 계획

## 1. 목표

Mona의 팀이 프로젝트 현황을 빠르게 파악할 수 있는 의존성 없는 정적
대시보드를 구현한다. 첫 화면에서 다음 정보를 읽을 수 있어야 한다.

- 프로젝트 이름과 담당자
- 현재 상태와 최근 활동
- 우선순위 또는 위험 수준
- 기여자가 이해하기 쉬운 짧은 프로젝트 요약
- 전체 프로젝트 수, 활성 프로젝트 수, 높은 우선순위 프로젝트 수

대시보드는 `app/` 디렉터리에서 정적 HTTP 서버로 실행되며, VS Code의
**Run Project Pulse Dashboard** 설정으로 디렉터리 목록이 아닌
`index.html`을 직접 연다.

## 2. 구현 범위와 파일 담당 배정

| 파일 | 담당 에이전트 | 책임 |
|---|---|---|
| `app/index.html` | Coder | 시맨틱 페이지 구조, 데이터 로딩, 카드 렌더링, 오류·빈 상태 처리 |
| `app/styles.css` | Designer | 레이아웃, 시각 계층, 상태·우선순위 스타일, 반응형·접근성 스타일 |
| `app/project-data.json` | Coder | `projects` 배열과 대표 프로젝트 데이터 정의 |
| `.vscode/launch.json` | Coder | `app/`를 작업 디렉터리로 사용하는 실행 및 브라우저 오픈 설정 |

Designer는 `app/index.html`에 필요한 구조와 CSS 훅을 설계 기준으로
제공하지만, 최종 HTML 파일은 Coder가 작성한다. Coder는 Designer가 만든
`app/styles.css`의 선택자와 상태 이름을 변경하지 않고 사용한다.

## 3. 에이전트 역할 분담

### Planner

요구사항과 저장소를 조사하고 데이터 구조, 작업 단계, 파일 소유권,
의존성, 예외 상황, 검증 기준을 정의한다. Planner는 코드를 작성하지
않으며 Orchestrator가 실행할 수 있는 계획을 제공한다.

### Designer

정보 구조, 사용자 흐름, 카드 레이아웃, 색상과 타이포그래피, 접근성,
반응형 동작을 결정한다. `.dashboard`, `.project-card`,
`.status-badge`, `.priority-badge` 같은 결정적인 CSS 훅을 기준으로
polished dashboard를 구현한다.

### Coder

Planner의 데이터 계약과 Designer의 UI 계약에 따라 HTML, JSON, 실행
설정을 작성한다. JSON을 안전하게 읽어 카드로 렌더링하고, 로딩 실패,
잘못된 데이터, 빈 목록을 사용자에게 명확히 표시한다. 변경 사항을
검증하되 Git stage, commit, push는 수행하지 않는다.

### Orchestrator

Planner의 계획을 바탕으로 작업을 단계화하고 Designer와 Coder의 파일
소유권을 조정한다. 병렬 실행 가능 여부를 판단하고, 구현 완료 후 파일
간 연결과 실행 결과를 통합 검토한다.

## 4. 구현 단계(phases)

### Phase 1: 요구사항과 데이터 계약 확정

**담당:** Planner, Orchestrator

1. Project Pulse brief와 저장소 구조를 확인한다.
2. `project-data.json`의 최상위 `projects` 배열과 필수 필드를 확정한다.
3. 상태와 우선순위의 표시 규칙, 오류 처리, 검증 기준을 문서화한다.
4. 각 파일의 단독 담당자를 확정해 동시 편집 충돌을 방지한다.

각 프로젝트는 다음 필드를 포함한다.

```json
{
  "name": "Project name",
  "owner": "Owner name",
  "status": "Active",
  "recentActivity": "Short recent update",
  "priority": "High",
  "summary": "Contributor-friendly summary"
}
```

권장 상태 값은 `Active`, `Planning`, `At risk`, `Blocked`, `Complete`이며,
권장 우선순위 값은 `High`, `Medium`, `Low`이다.

### Phase 2: UI/UX 및 접근성 기준 설계

**담당:** Designer

1. 헤더, 요약 지표, 프로젝트 목록의 정보 계층을 정의한다.
2. 카드에 프로젝트 이름, 상태, 담당자, 최근 활동, 요약, 우선순위를
   표시하는 구조를 정의한다.
3. 상태와 우선순위를 색상만으로 구분하지 않고 텍스트 배지로 표시한다.
4. `.dashboard`, `.project-grid`, `.project-card`,
   `.project-card__header`, `.project-card__owner`,
   `.project-card__activity`, `.project-card__summary`,
   `.project-card__priority`, `.status-badge`,
   `.priority-badge` 훅을 사용한다.
5. 카드의 `border-radius`, `box-shadow`, 명확한 간격과 대비를 정의한다.
6. 모바일에서는 한 열, 넓은 화면에서는 반응형 다중 열 그리드를 사용한다.
7. 긴 텍스트 줄바꿈, 키보드 focus 표시, 200% 확대, reduced motion을
   고려한다.

### Phase 3: 데이터와 화면 구현

**담당:** Coder

1. `app/project-data.json`에 여러 대표 프로젝트를 작성한다. 활성, 계획 중,
   위험 또는 차단, 완료 상태와 다양한 우선순위를 포함한다.
2. `app/index.html`에 `Project Pulse` 제목과 `styles.css` 링크를 추가한다.
3. inline script에서 `project-data.json`을 HTTP로 읽고 `projects` 배열을
   검증한다.
4. 각 프로젝트를 안전한 DOM API와 `textContent`로 렌더링한다.
5. 각 카드에 `project-card` 클래스를 부여하고 필수 정보를 모두 표시한다.
6. 데이터 로드 실패, 잘못된 구조, 빈 배열, 필드 누락, 알 수 없는 상태나
   우선순위를 명시적인 오류·빈 상태 또는 중립 스타일로 처리한다.
7. `.vscode/launch.json`을 작성해 `app/`에서 HTTP 서버를 실행하고
   `index.html`을 직접 열도록 한다.

### Phase 4: 통합 및 검증

**담당:** Orchestrator, Coder

1. HTML의 선택자와 CSS 훅, JSON 필드 이름이 일치하는지 확인한다.
2. launch 설정이 올바른 작업 디렉터리와 URL을 사용하는지 확인한다.
3. 정적 서버로 실제 페이지와 JSON이 로드되는지 확인한다.
4. 데스크톱, 태블릿, 모바일 폭과 키보드 탐색을 확인한다.
5. 요구사항을 충족하지 못한 항목이 있으면 담당 에이전트가 해당 파일만
   수정하고 통합 검토를 반복한다.

## Dependencies

파일 간 의존성에 대한 설명입니다.

```text
app/project-data.json
        │
        └── app/index.html (fetch 및 카드 렌더링)
                         │
                         └── app/styles.css (HTML CSS 훅을 스타일링)

.vscode/launch.json
        └── app/ 디렉터리의 HTTP 서버 실행
                └── app/index.html이 project-data.json을 로드
```

세부 의존성:

| 의존하는 파일 | 의존 대상 | 이유 |
|---|---|---|
| `app/index.html` | `app/styles.css` | 문서의 CSS 클래스와 시각 상태를 스타일링 |
| `app/index.html` | `app/project-data.json` | `projects` 배열을 읽어 카드를 생성 |
| `.vscode/launch.json` | `app/index.html` | 서버 시작 위치와 브라우저 오픈 URL 결정 |
| `app/styles.css` | `app/index.html` | HTML의 클래스와 상태 훅에 맞춰 스타일 적용 |

`app/index.html`과 `app/styles.css`는 같은 CSS 훅 계약을 공유한다.
`app/project-data.json`의 필드 이름은 HTML 렌더러가 기대하는 이름과
일치해야 한다. `fetch()`가 동작하도록 페이지는 `file://`가 아니라
HTTP 서버를 통해 열어야 한다.

### 데이터 파일을 먼저 준비하는 이유

`app/project-data.json`은 `app/index.html`이 렌더링할 데이터 계약의
기준이므로 HTML보다 먼저 스키마와 샘플 데이터를 확정한다. HTML의 inline
script는 `projects` 배열과 `name`, `owner`, `status`, `recentActivity`,
`priority`, `summary` 필드를 기대한다. 데이터 파일을 먼저 만들면 다음을
확인한 뒤 화면을 구현할 수 있다.

- JSON 문법과 최상위 `projects` 배열이 유효한지
- 모든 프로젝트가 HTML 렌더러가 요구하는 필드를 제공하는지
- 상태와 우선순위의 실제 값이 CSS 상태 훅과 일치하는지
- HTML이 임의의 필드 이름이나 빈 값에 의존하지 않는지

단, `index.html`이 존재해야 브라우저에서 JSON을 요청할 수 있으므로
“먼저”는 스키마와 파일을 먼저 확정한다는 의미다. 실제 HTTP 로딩 확인은
HTML과 `launch.json`이 완성된 뒤 함께 수행한다.

## Ordering

작업 순서에 대한 설명입니다.

파일과 담당 에이전트의 구현 순서는 다음과 같다.

1. **Planner — 계약과 수용 기준 확정**
   - Project Pulse brief를 읽고 필수 필드, 상태·우선순위 값, 오류 상태,
     검증 기준을 확정한다.
   - 이 단계에서는 구현 파일을 수정하지 않는다.
2. **Designer — 구조·스타일 계약 정의 및 `app/styles.css` 작성**
   - HTML이 사용할 CSS 훅과 상태 이름을 정한다.
   - 반응형 카드, 배지, 접근성 focus, 오류·빈 상태 스타일을 작성한다.
   - Coder가 이 계약을 기준으로 HTML을 작성하므로 Designer의 CSS 훅
     확정이 HTML 구현보다 선행되어야 한다.
3. **Coder — `app/project-data.json` 작성**
   - 확정된 스키마로 대표 프로젝트 데이터를 먼저 작성하고 JSON을
     파싱해 기본 데이터 계약을 검증한다.
   - 데이터 파일은 HTML의 렌더러가 소비할 실제 필드와 값의 기준이 된다.
4. **Coder — `app/index.html` 작성**
   - Designer가 정한 CSS 훅을 사용하고 `project-data.json`을 로드한다.
   - 카드 렌더링, 요약 지표, 로딩·오류·빈 상태를 구현한다.
   - JSON 값은 `textContent` 등 안전한 DOM API로 삽입한다.
5. **Coder — `.vscode/launch.json` 작성**
   - `app/`를 `cwd`로 지정하고 HTTP 서버와 `/index.html` 오픈 URL을
     설정한다.
   - 이 파일은 HTML과 JSON을 함께 로드할 실행 환경을 제공한다.
6. **Orchestrator — 통합 및 최종 검증**
   - 네 파일의 필드명, CSS 훅, 경로, 실행 URL을 서로 대조한다.
   - HTTP 서버로 실제 페이지를 열어 데이터가 카드로 표시되는지 확인한다.

### Ordering과 병렬 작업

Planner의 요구사항 조사와 Designer의 초기 시각 설계는 병렬로 진행할 수
있다. Designer가 CSS 계약을 확정한 뒤에는 Coder의 JSON 작성과
`.vscode/launch.json` 작성도 서로 다른 파일이므로 병렬 처리할 수 있다.
그러나 `app/index.html`은 JSON 필드와 Designer의 CSS 훅을 모두 소비하므로
두 계약이 확정된 뒤에 작성한다. 네 파일이 모두 완성되기 전에는 통합
검증을 실행하지 않는다.

Designer and Coder can work in parallel on `app/styles.css` and
`app/project-data.json` after the shared contracts are agreed.

## Validation

완료 후 검증 방법에 대한 설명입니다.

### 정적 파일 및 데이터 검증

1. 네 파일이 모두 존재하는지 확인한다.
2. `python3 -m json.tool app/project-data.json`으로 프로젝트 데이터
   문법을 확인한다.
3. `python3 -m json.tool .vscode/launch.json`으로 launch 설정이 주석
   없는 유효한 JSON인지 확인한다.
4. `projects`가 배열인지 확인하고, 각 항목에 `name`, `owner`, `status`,
   `recentActivity`, `priority`, `summary`가 있는지 확인한다.
5. 상태와 우선순위 값이 의도한 대표 사례를 포함하는지 확인한다.

### HTML·CSS 연결 검증

1. HTML title이 정확히 `Project Pulse`인지 확인한다.
2. HTML이 `styles.css`와 `project-data.json`을 참조하는지 확인한다.
3. HTML에 `.dashboard`와 `.project-card` 렌더링 경로가 있는지 확인한다.
4. CSS에 `.dashboard`, `.project-card`, `border-radius`,
   `box-shadow` 및 Designer가 정의한 배지·상태 훅이 있는지 확인한다.
5. JSON 값이 안전한 DOM API로 삽입되는지 확인하고, 신뢰할 수 없는
   데이터를 직접 `innerHTML`에 삽입하지 않는지 확인한다.

### 실제 실행 검증

1. `python3 -m http.server 5500 --directory app`로 로컬 서버를 시작한다.
2. `curl` 또는 브라우저로 `/index.html`이 성공적으로 응답하는지
   확인한다.
3. `/project-data.json`이 성공적으로 응답하는지 확인한다.
4. 브라우저에서 디렉터리 목록이 아니라 Project Pulse UI가 열리는지
   확인한다.
5. JSON의 모든 프로젝트가 카드로 표시되고 각 카드에 이름, 담당자,
   상태, 최근 활동, 요약, 우선순위가 보이는지 확인한다.
6. JSON 로드 실패, 빈 `projects` 배열, 필드 누락, 알 수 없는 상태나
   우선순위가 빈 화면 없이 처리되는지 확인한다.

The final validation confirms that the data, rendered interface, and launch
configuration work together through the configured HTTP server.

### UI·접근성 검증

- 모바일 폭에서 카드가 한 열로 표시되고 가로 스크롤이 없는지 확인한다.
- 태블릿과 데스크톱에서 카드 그리드가 자연스럽게 확장되는지 확인한다.
- 긴 프로젝트명과 활동 내용이 잘리지 않고 줄바꿈되는지 확인한다.
- 키보드만으로 탐색할 때 focus 표시와 논리적인 읽기 순서를 확인한다.
- 상태·우선순위가 색상 없이도 텍스트로 이해되는지 확인한다.
- 200% 확대와 `prefers-reduced-motion` 환경에서 핵심 정보가 usable한지
  확인한다.
- 마지막으로 `git diff --check`를 실행해 문서와 구현 파일의 공백 오류를
  확인한다.

## 8. 병렬 작업 가능 여부

| 작업 | 병렬 가능 여부 | 조건 |
|---|---|---|
| Planner의 저장소 조사와 Designer의 초기 UX 조사 | 가능 | 서로 구현 파일을 수정하지 않음 |
| Designer의 `app/styles.css` 작성과 Planner의 계획 작성 | 가능 | Planner가 CSS 파일을 소유하지 않음 |
| Coder의 `app/project-data.json` 작성과 `.vscode/launch.json` 작성 | 가능 | 두 파일의 내용이 직접 겹치지 않음 |
| Coder의 `app/index.html` 작성과 Designer의 `app/styles.css` 작성 | 불가 | HTML의 최종 CSS 훅 계약이 먼저 합의되어야 함 |
| 통합 검증 | 불가 | 네 개 파일이 모두 완성된 뒤 실행해야 함 |

권장 순서는 Phase 1을 먼저 완료하고, Phase 2와 데이터 초안 작업을
병렬로 진행한 다음, Designer의 CSS 계약이 확정되면 Coder가 HTML과
나머지 파일을 완성하는 것이다. 동일 파일을 여러 에이전트가 동시에
수정하지 않는다.

## 9. 실행 설정 계약

`.vscode/launch.json`은 주석 없는 유효한 JSON이어야 하며 다음 조건을
만족해야 한다.

- configuration 이름: `Run Project Pulse Dashboard`
- `type`: `node-terminal`
- `request`: `launch`
- command: `python3 -m http.server 5500`
- `cwd`: `${workspaceFolder}/app`
- 서버 준비 후 URL: `http://localhost:%s/index.html`

## 10. 검증 기준

### 구조와 데이터

- `app/index.html`, `app/styles.css`, `app/project-data.json`,
  `.vscode/launch.json`이 존재한다.
- `app/project-data.json`이 유효한 JSON이고 최상위 `projects` 배열을
  포함한다.
- 모든 프로젝트에 `name`, `owner`, `status`, `recentActivity`,
  `priority`, `summary`가 있다.
- 데이터에 여러 프로젝트와 다양한 상태·우선순위가 포함된다.

### 화면과 동작

- HTML title이 정확히 `Project Pulse`이다.
- HTML이 `styles.css`와 `project-data.json`을 참조한다.
- 모든 데이터 프로젝트가 `.project-card`로 렌더링된다.
- 각 카드에 이름, 담당자, 상태, 최근 활동, 요약, 우선순위가 보인다.
- `.dashboard`, `.project-card`, `border-radius`, `box-shadow`가 CSS에
  존재한다.
- JSON 로드 실패와 빈 목록이 빈 화면으로 남지 않는다.
- JSON 값은 안전한 DOM API로 삽입되며 신뢰할 수 없는 값을 직접
  `innerHTML`에 삽입하지 않는다.

### 시각·접근성·반응형

- 상태와 우선순위가 텍스트로도 이해된다.
- 상태 배지, 우선순위 배지, 본문 텍스트의 대비가 충분하다.
- 키보드 focus가 명확하고 논리적인 읽기 순서를 유지한다.
- 모바일에서는 한 열로 표시되고 가로 스크롤이 발생하지 않는다.
- 긴 프로젝트명과 활동 문장이 카드 밖으로 넘치거나 잘리지 않는다.
- 200% 확대에서도 핵심 정보와 조작 흐름을 사용할 수 있다.

### 실행과 통합

- `.vscode/launch.json`이 strict JSON으로 파싱된다.
- 실행 설정의 `cwd`가 `app/`이고 URL이 `/index.html`로 끝난다.
- HTTP 서버에서 `index.html`과 `project-data.json`이 정상 응답한다.
- 브라우저가 디렉터리 목록이 아닌 Project Pulse 화면을 연다.
- 변경 후 `git diff --check`가 통과한다.

## 11. 완료 조건

Orchestrator가 네 개의 담당 파일과 위 검증 기준을 모두 확인하고,
Designer의 시각·접근성 계약과 Coder의 데이터·실행 구현이 일치한다고
판단하면 Project Pulse 구현을 완료로 보고한다. Git stage, commit, push는
계획 범위에 포함하지 않으며 사용자가 직접 관리한다.
