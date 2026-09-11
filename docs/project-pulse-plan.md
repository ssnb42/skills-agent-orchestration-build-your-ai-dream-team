# Project Pulse 구현 계획

## 목표

Mona의 팀이 프로젝트 상태를 빠르게 파악할 수 있는 의존성 없는 정적 대시보드를 만든다. 대시보드는 다음 정보를 보여준다.

- 활성 프로젝트
- 프로젝트 소유자
- 현재 상태
- 최근 활동
- 우선순위 또는 위험 수준
- 기여자 친화적인 짧은 요약

최종 구현 파일은 다음과 같다.

- `app/index.html`
- `app/styles.css`
- `app/project-data.json`
- `.vscode/launch.json`

## 에이전트 역할

### Orchestrator

전체 작업을 조율한다. Planner의 계획과 Designer의 UI 기준을 Coder에게 전달하고, 파일 소유권과 작업 순서를 관리하며, 최종 결과를 통합 검토한다.

### Planner

저장소와 요구사항을 조사하고 데이터 스키마, 구현 단계, 의존 관계, 예외 상황, 검증 기준을 정의한다.

### Designer

정보 구조, 카드 레이아웃, 접근성, 상태 및 우선순위 표시 방식, 반응형 동작과 시각적 스타일 기준을 정의한다.

### Coder

Planner와 Designer의 기준에 따라 네 개의 구현 파일을 작성하고, 데이터 로딩 및 실행 설정을 구현한다.

## 구현 단계

### 1. 요구사항 및 데이터 구조 확정

**담당:** Planner

`app/project-data.json`은 최상위 `projects` 배열을 사용한다. 각 프로젝트에는 다음 필드를 포함한다.

```json
{
  "name": "Project name",
  "owner": "Owner name",
  "status": "Active",
  "recentActivity": "Recent contributor-facing update",
  "priority": "High",
  "summary": "Short project summary"
}
```

필수 필드는 `name`, `owner`, `status`, `recentActivity`, `priority`이며, contributor-friendly summary 요구사항을 충족하기 위해 `summary`를 추가한다.

권장 상태 값:

- `Active`
- `Planning`
- `At risk`
- `Blocked`
- `Complete`

권장 우선순위 값:

- `High`
- `Medium`
- `Low`

### 2. UI/UX 설계

**담당:** Designer  
**대상:** `app/index.html`, `app/styles.css`의 구조와 스타일 기준

화면은 다음 정보 계층을 따른다.

1. `Project Pulse` 제목과 기여자 친화적인 설명을 포함한 페이지 헤더
2. 전체 프로젝트 수, 활성 프로젝트 수, 높은 우선순위 프로젝트 수를 보여주는 요약 영역
3. 프로젝트 카드를 포함하는 프로젝트 목록 영역
4. 각 카드의 프로젝트 이름, 상태, 소유자, 최근 활동, 요약, 우선순위 정보

필수 CSS 선택자는 다음과 같다.

- `.dashboard`
- `.project-card`

권장 추가 선택자는 다음과 같다.

- `.dashboard-header`
- `.project-grid`
- `.project-card__header`
- `.project-card__owner`
- `.project-card__activity`
- `.project-card__summary`
- `.project-card__priority`
- `.status-badge`
- `.priority-badge`

디자인 기준:

- 카드에 `border-radius`, `box-shadow`, 명확한 내부 여백을 적용한다.
- 상태와 우선순위는 색상만으로 구분하지 않고 텍스트 라벨을 함께 표시한다.
- 상태와 우선순위는 서로 다른 시각적 체계로 표현한다.
- 작은 화면에서는 한 열 레이아웃으로 전환한다.
- 긴 프로젝트명과 활동 내용은 자연스럽게 줄바꿈한다.
- 키보드 포커스가 필요한 요소에는 명확한 focus 스타일을 적용한다.
- 200% 확대와 좁은 화면에서도 가로 스크롤이 발생하지 않도록 한다.
- 필수 정보가 hover 상태에서만 나타나지 않도록 한다.

### 3. 정적 대시보드 구현

**담당:** Coder  
**파일 소유권:**

- `app/index.html`
- `app/styles.css`
- `app/project-data.json`
- `.vscode/launch.json`

#### `app/index.html`

- 정확한 문서 제목을 `Project Pulse`로 설정한다.
- `styles.css`를 연결한다.
- `project-data.json`을 로드한다.
- JSON의 `projects` 배열에서 프로젝트 카드를 동적으로 생성한다.
- 각 카드에 `project-card` 클래스를 적용한다.
- 이름, 소유자, 상태, 최근 활동, 요약, 우선순위를 표시한다.
- 별도의 JavaScript 파일 없이 작은 inline script를 사용한다.
- JSON 로드 실패나 잘못된 데이터에 대해 명시적인 오류 메시지를 표시한다.
- 원본 데이터를 안전하게 삽입하고, 신뢰할 수 없는 값을 직접 `innerHTML`에 넣지 않는다.

`fetch()`를 사용하므로 `file://`로 직접 여는 대신 HTTP 서버를 통해 실행한다.

#### `app/styles.css`

- `.dashboard` 기반의 전체 레이아웃을 구성한다.
- `.project-card` 기반의 프로젝트 카드 스타일을 구성한다.
- 상태 및 우선순위별 시각적 구분을 제공한다.
- CSS Grid로 반응형 레이아웃을 구현한다.
- 카드 간격, 타이포그래피, 대비, focus 상태를 정의한다.
- 긴 텍스트가 카드 밖으로 넘치지 않도록 한다.
- 애니메이션을 추가할 경우 `prefers-reduced-motion`을 고려한다.

#### `app/project-data.json`

여러 프로젝트를 포함하여 활성, 계획 중, 높은 우선순위, 위험 또는 차단, 완료 상태가 화면에 드러나도록 구성한다.

#### `.vscode/launch.json`

주석 없는 유효한 JSON으로 작성한다. 설정 이름은 정확히 `Run Project Pulse Dashboard`로 지정하고, `app/` 디렉터리에서 Python HTTP 서버를 실행하여 `index.html`을 직접 연다.

권장 설정:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node-terminal",
      "request": "launch",
      "name": "Run Project Pulse Dashboard",
      "command": "python3 -m http.server 5500",
      "cwd": "${workspaceFolder}/app",
      "serverReadyAction": {
        "pattern": "Serving HTTP on .* port (\\d+)",
        "uriFormat": "http://localhost:%s/index.html",
        "action": "openExternally"
      }
    }
  ]
}
```

## 작업 의존 관계

| 단계 | 담당 | 선행 조건 |
|---|---|---|
| 요구사항 및 데이터 스키마 확정 | Planner | Project Pulse brief |
| 레이아웃·접근성·시각 기준 결정 | Designer | 요구사항 및 데이터 구조 |
| JSON 데이터 작성 | Coder | 데이터 스키마 |
| HTML 및 CSS 작성 | Coder | Designer의 UI 기준 |
| VS Code 실행 설정 작성 | Coder | `app/` 경로 및 실행 방식 |
| 통합 검토 및 실행 확인 | Orchestrator | 모든 구현 파일 완료 |

Planner와 Designer의 조사는 병렬로 진행할 수 있다. Coder의 최종 HTML과 CSS 구현은 두 결과를 전달받은 뒤 진행한다. `app/index.html`과 `app/styles.css`는 충돌 방지를 위해 Coder가 단독으로 소유한다.

## 예외 및 오류 처리

- JSON 로드 실패: 사용자에게 데이터 로드 실패 메시지를 표시한다.
- `projects`가 없거나 빈 배열: 접근 가능한 empty state를 표시한다.
- 필드 누락: 안전한 대체 텍스트를 사용하거나 해당 항목을 명확히 제외한다.
- 알 수 없는 상태 또는 우선순위: 중립 스타일로 렌더링한다.
- 긴 텍스트: 고정 높이로 잘라내지 않고 줄바꿈을 허용한다.
- `file://` 실행: `fetch()` 실패 가능성이 있으므로 HTTP 서버를 사용한다.
- 색상 대비 부족: 텍스트 라벨과 충분한 명도 대비를 유지한다.

## 최종 검증

기존 저장소에는 별도의 빌드 시스템이나 테스트 스위트가 없으므로 새로운 도구를 추가하지 않는다. 다음 항목을 확인한다.

1. `app/project-data.json`이 유효한 JSON인지 확인한다.
2. `projects` 배열과 각 필수 필드가 존재하는지 확인한다.
3. `app/index.html`이 데이터 파일을 로드하고 카드를 렌더링하는지 확인한다.
4. 모든 카드에 `project-card` 클래스가 있는지 확인한다.
5. 상태, 최근 활동, 우선순위가 실제 데이터에서 표시되는지 확인한다.
6. `app/styles.css`에 `.dashboard`, `.project-card`, `border-radius`, `box-shadow`가 있는지 확인한다.
7. 모바일, 태블릿, 데스크톱 폭에서 레이아웃을 확인한다.
8. 키보드 탐색과 focus 표시를 확인한다.
9. `Run Project Pulse Dashboard`로 실행한다.
10. 브라우저가 디렉터리 목록이 아닌 `app/index.html`을 여는지 확인한다.
11. 오류 또는 빈 데이터 상태가 빈 화면으로 남지 않는지 확인한다.

## 실행 순서 요약

1. Orchestrator가 Project Pulse brief를 읽고 작업 범위를 분배한다.
2. Planner가 데이터 스키마와 검증 기준을 확정한다.
3. Designer가 레이아웃, 접근성, 상태·우선순위 표시 기준을 정의한다.
4. Coder가 네 개의 구현 파일을 작성한다.
5. Orchestrator가 파일 간 연결과 실행 결과를 통합 검토한다.
