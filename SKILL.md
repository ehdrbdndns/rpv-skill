---
name: rpv
description: 사용자가 /rpv를 호출하거나, 구현 전에 research.md, plan.md, verification-scenarios.md를 작성하고 사용자 검토를 거친 뒤 개발하길 원할 때 사용한다. 이 스킬은 AI가 의도를 추측해 구현하지 않도록 상세한 조사 문서, 구현 계획 문서, 검증 시나리오 문서를 먼저 만들고, manual verification이 모두 통과할 때까지 구현과 검증을 반복한다.
---

# RPV: Research, Plan, Verify

`/rpv`는 빠르게 구현하는 스킬이 아니다.
목표는 사용자의 의도를 발명하지 않고, 먼저 사실을 문서화하고, 구현 범위를 합의하고, 성공 기준을 검증 시나리오로 고정한 뒤 개발하는 것이다.

이 스킬은 다음 작업에 사용한다.

- 사용자-facing 기능
- 상태, 인증, 세션, 권한, 저장, 라우팅이 얽힌 작업
- 여러 파일이나 모듈을 건드리는 작업
- 원인이 불확실한 버그
- 사용자가 구현 전 collaborative planning을 원하는 작업
- `/rpv`를 명시적으로 호출한 작업

작은 문구 수정이나 단순 기계적 변경에는 전체 문서 루프를 자동 적용하지 않는다. 다만 사용자가 `/rpv`를 호출했다면 작업이 작아도 이 절차를 따른다.

## 언어 규칙

사용자와의 모든 대화는 사용자의 언어로 한다.

한국어 사용자의 경우 다음을 한국어로 작성한다.

- intake 질문
- 진행 상황 공유
- review 요청
- `research.md`
- `plan.md`
- `verification-scenarios.md`
- 최종 보고

문서는 AI의 내부 메모가 아니라 사용자가 검토하는 계약 문서다. 따라서 사용자가 가장 편하게 검토할 수 있는 언어로 작성한다.

## 문서 형식 규칙

사용자가 검토하는 `research.md`, `plan.md`, `verification-scenarios.md`, 최종 보고는 항목을 나열할 때 bullet list보다 numbered list를 우선 사용한다.
체크박스, 파일 트리, 코드 블록, 표처럼 numbered list보다 다른 형식이 더 명확한 경우에만 예외로 둔다.

## 코드베이스 분석 도구 규칙

코드 구조, symbol, 참조 관계, 데이터 흐름, 기존 패턴을 분석할 때는 Serena MCP를 우선이 아니라 필수 도구로 사용한다.
Serena MCP가 사용 가능한 환경에서는 `get_symbols_overview`, `find_symbol`, `find_referencing_symbols`, `search_for_pattern` 같은 semantic/symbolic 도구로 먼저 범위를 좁힌다.
파일 전체를 무작정 읽거나 grep 결과만으로 구조를 판단하지 않는다.
필요한 symbol과 주변 맥락을 Serena MCP로 좁힌 뒤, 부족한 부분만 `rg`, `sed`, `cat` 같은 일반 파일 읽기 도구로 보완한다.
Serena MCP를 사용할 수 없는 환경이라면 그 이유를 `research.md`의 `Unknowns` 또는 `Facts`에 명시하고, 어떤 대체 근거로 코드 구조를 확인했는지 적는다.

## 문서 저장 위치

`/rpv` 실행 중 생성되는 문서는 현재 작업 루트에 `rpv/` 폴더를 만들고 그 안에 저장한다.

기본 파일:

```text
rpv/
├── research.md
├── plan.md
└── verification-scenarios.md
```

기존 `rpv/` 문서가 현재 작업과 관련 있어 보이면 이어서 갱신한다. 관련 없어 보이면 날짜나 짧은 작업명을 붙인 하위 폴더를 만든다.

```text
rpv/
└── feature-name/
    ├── research.md
    ├── plan.md
    ├── verification-scenarios.md
    ├── hotfix-issue-name.md
    └── final-report.md
```

`hotfix-*.md`는 검증 중 발견된 좁은 차단 이슈를 기록하는 선택 문서다. 원래 RPV 목표를 달성하기 위해 필요한 최소 수정에만 사용한다.

문서를 만들기 전에 기존 `rpv/` 폴더가 있는지 확인한다. 문서는 사용자 검토용 계약 문서이므로, 임시 메모처럼 숨겨진 위치나 시스템 디렉터리에 저장하지 않는다.

## 전체 흐름

1. Intake 질문을 통해 사용자의 의도, 제약, 예시, 성공 기준을 파악한다.
2. `rpv/research.md`를 작성한다.
3. 멈추고 사용자에게 `rpv/research.md` 검토를 요청한다.
4. 사용자가 정확히 `승인`이라고 말하면 `rpv/plan.md`를 작성한다. 피드백이 있으면 반영한 뒤 다시 `승인`을 기다린다.
5. 멈추고 사용자에게 `rpv/plan.md` 검토를 요청한다.
6. 사용자가 정확히 `승인`이라고 말하면 사용자가 원하는 검증 시나리오가 있는지 묻는다. 피드백이 있으면 반영한 뒤 다시 `승인`을 기다린다.
7. `rpv/verification-scenarios.md`를 작성한다.
8. 멈추고 사용자에게 `rpv/verification-scenarios.md` 검토를 요청한다.
9. 사용자가 정확히 `승인`이라고 말하면 구현을 시작한다. 피드백이 있으면 반영한 뒤 다시 `승인`을 기다린다.
10. `rpv/plan.md`의 구현 순서에 따라 구현한다.
11. `rpv/verification-scenarios.md`에 정한 검증 도구와 환경에 따라 수동 검증한다.
12. 검증 실패 시 먼저 실패가 기존 plan 범위 안의 구현 버그인지 판단한다.
13. 기존 plan 범위 안이면 구현 단계로 돌아가 원인을 수정한다.
14. 기존 plan 밖의 blocker, contract drift, DB/infra/schema mismatch라면 `hotfix-*.md`를 작성하고 사용자 검토를 받은 뒤 처리한다.
15. hotfix 적용 후 원래 검증 시나리오로 돌아와 재검증한다.
16. 필수 검증 시나리오가 모두 통과할 때까지 구현과 검증 루프를 반복한다.
17. 최종 보고를 작성한다.

각 문서 review gate 이후에는 사용자 피드백을 반영하거나, 사용자가 정확히 `승인`이라고 말하기 전까지 다음 단계로 넘어가지 않는다.
`좋아`, `괜찮아`, `계속해`, `진행해`, `LGTM`처럼 긍정적이지만 `승인`이라는 단어가 없는 표현은 다음 단계 진행 승인으로 간주하지 않는다.
사용자 메시지에 `승인`이 포함되어 있더라도 조건, 질문, 수정 요청이 함께 있으면 먼저 그 내용을 반영하고 다시 review gate를 연다.
각 문서의 `Review Questions`에는 질문만 적지 말고, 왜 이 질문이 필요한지와 답변이 다음 단계에 어떤 결정을 만들거나 막는지 함께 적는다.
사용자가 `Review Questions`에 답변하면, 답변 내용을 해당 문서의 적절한 섹션에 반영한 뒤 해결된 질문은 `Review Questions`에서 제거한다.
답변을 반영하는 과정에서 새로운 미확정 결정, blocker, scope 변화, 검증 공백이 드러나면 새 `Review Questions` 항목을 추가한다.
새 질문이 생긴 경우 해당 문서의 review gate를 다시 열고, 사용자가 정확히 `승인`이라고 말하기 전까지 다음 단계로 넘어가지 않는다.
`Review Questions`에는 아직 답변되지 않았거나 다음 단계 진행을 막는 질문만 남긴다.

검증 실패가 `rpv/research.md`, `rpv/plan.md`, `rpv/verification-scenarios.md`의 오류나 누락을 드러내면, 해당 문서를 수정하고 사용자 검토를 다시 요청한 뒤 계속한다.

## 1. Intake

`rpv/research.md`를 작성하기 전에, 잘못된 문제를 조사하지 않도록 사용자에게 필요한 최소 정보를 묻는다.

사용자에게는 제품 의도, 제약, 예시, 성공 기준을 묻고, 구현 세부사항은 에이전트가 직접 조사한다.

기본 질문:

```markdown
1. 원하는 결과는 무엇인가요?
2. 현재 어떤 동작이 잘못되었거나, 없거나, 마음에 들지 않나요?
3. 영향을 받는 사용자나 역할은 누구인가요?
4. 제품의 어느 화면/흐름/기능에서 발생하나요?
5. 참고할 예시, 스크린샷, 링크, 로그, 이전 시도 내용이 있나요?
6. 지켜야 할 제약이나 이번 작업에서 하지 말아야 할 것이 있나요?
7. 어떤 상태가 되면 완료됐다고 판단하시겠나요?
```

이미 충분한 정보가 있으면 중복 질문하지 않는다. 대신 사용할 가정을 짧게 밝히고 research를 시작한다.

사용자가 급하게 진행하라고 하거나, 명시적으로 바로 시작하라고 하면 가정을 밝히고 research를 시작하되, 불확실한 내용은 `rpv/research.md`의 `Unknowns`에 기록한다.

## 2. `rpv/research.md`

`rpv/research.md`는 현재 시스템과 문제를 어떻게 이해했는지, 그리고 Goal을 달성하기 위해 무엇을 알아야 하는지 조사한 문서다.
다른 에이전트가 읽어도 맥락을 새로 발명하지 않고, 필요한 구현 지식과 판단 근거를 다시 조사하지 않아도 될 만큼 자세해야 한다.

포함할 내용:

```markdown
# Research

## User Intake
1. 요청한 결과:
2. 현재 불편/문제:
3. 영향을 받는 사용자/역할:
4. 관련 화면/흐름:
5. 제공된 예시/자료:
6. 제약/Non-goals:
7. 사용자의 완료 기준:

## Goal
사용자가 달성하려는 결과를 한 문장으로 정리한다.

## Current Behavior
현재 시스템이 실제로 어떻게 동작하는지 적는다.

## Relevant Surfaces
관련 화면, 라우트, 파일, API, 데이터 흐름, 상태, 권한, 외부 연동, 브라우저 동작, 배포 표면 등을 적는다.

## Codebase Analysis Method
1. Serena MCP로 확인한 symbols/참조/패턴:
2. Serena MCP로 좁힌 파일/모듈 범위:
3. 추가로 사용한 보조 명령:
4. Serena MCP를 사용할 수 없었다면 이유와 대체 확인 방법:

## Facts
코드, 실행 결과, 로그, 문서, 브라우저 관찰, 테스트, 명령 출력으로 확인한 사실만 적는다.

## Required Knowledge for Goal
Goal을 달성하기 위해 새로 알아야 하는 구현 지식, 도메인 지식, 라이브러리/API 사용법, 플랫폼 제약, 권한/보안/성능 고려사항을 적는다.
1. 예: QR 코드 기능이라면 생성/스캔/표시 중 무엇이 필요한지, 사용할 수 있는 라이브러리, React Native/iOS/Android 제약, 권한 설정, 기존 화면/상태/API와 연결되는 방식을 조사한다.
2. 예: 결제 기능이라면 결제 SDK/API, 인증/보안 요구사항, 실패/취소/환불 흐름, 서버 검증 방식, 기존 주문/구독 데이터와 연결되는 방식을 조사한다.

## Third-Party / Official Docs
Goal을 달성하기 위해 3rd party library, framework, external API, SDK, hosted service, platform capability가 필요할 수 있으면 공식 문서나 primary source를 확인한다.
확인한 문서 링크, 관련 버전, 필요한 API/설정/권한/제약/권장 패턴, 현재 코드베이스에 적용할 때의 차이를 적는다.
공식 문서 확인이 불가능하면 그 이유와 대체로 확인한 근거를 적고 `Unknowns`에도 남긴다.

## Existing Patterns
프로젝트 안에서 비슷한 기능이나 흐름이 이미 어떻게 구현되어 있는지 적는다.

## Unknowns
아직 확인하지 못한 것, 검증이 필요한 가정, 사용자에게 확인할 질문을 적는다.

## Risks
잘못 변경하면 동작, 데이터, 권한, UX, 성능, 배포에 영향을 줄 수 있는 부분을 적는다.

## Likely Change Points
수정 가능성이 높은 파일, 모듈, 흐름을 적는다. 아직 확정 구현안처럼 쓰지 않는다.

## Review Questions
plan 작성 전에 사용자에게 확인받아야 할 질문을 적는다.
답변된 질문은 답변을 `User Intake`, `Facts`, `Required Knowledge for Goal`, `Third-Party / Official Docs`, `Unknowns`, `Risks`, `Likely Change Points` 등 적절한 섹션에 반영한 뒤 제거한다.
답변 때문에 새로운 unknown, risk, likely change point가 드러나면 새 질문을 추가한다.
각 질문은 아래 형식을 따른다.

### Question N: 질문
1. 질문:
2. 의도:
3. 답변이 plan에 미치는 영향:
```

작성 후 핵심 발견, 미확인점, 위험, 수정 후보를 요약하고 사용자 검토를 요청한다.

## 3. `rpv/plan.md`

`rpv/plan.md`는 구현 계약서다.
다른 에이전트가 이 문서만 보고도 제품 의도, 데이터 동작, UI 동작, 검증 기대값을 추측하지 않고 구현할 수 있을 만큼 자세해야 한다.

`rpv/plan.md`는 반드시 실제 코드베이스 조사 결과에 근거해야 한다.
파일 경로 목록이나 추상적인 작업 목록으로 끝내지 말고, 다른 에이전트가 추가 조사 없이 구현 방향을 잡을 수 있을 정도로 관련 파일, symbol, 기존 패턴, 데이터 흐름, edge case, 검증 포인트를 상세히 적는다.
핵심 code shape, 함수 signature, type/interface, query/API 호출 형태, 상태 변경 흐름은 implementation sketch 수준으로 포함한다.
다만 사용자 승인 전 완성된 production code 전체를 작성하지 않는다.

포함할 내용:

```markdown
# Plan

## Goal
이번 구현으로 달성할 구체적 결과.

## Scope / Non-Goals
1. 포함되는 것:
2. 포함되지 않는 것:

## Approach
1. 선택한 구현 방향:
2. 기존 코드베이스 패턴과 맞는 이유:
3. Open/Closed Principle(OCP)과 backward-compatible 확장 방식:
4. 직접 수정이 필요한 부분과 이유:

## Implementation Sketch
1. 함수 signature/type/interface:
2. query/API 호출 형태:
3. route/component/service 연결:
4. 상태 변경/저장/비동기 흐름:
5. edge/error 처리:
6. React Native 앱이면 stable `testID`/accessibility id:
7. 실제 구현 단계에서 코드베이스 스타일에 맞게 조정해야 할 부분:

## Implementation Steps
각 단계마다 아래를 적는다.
각 step은 위치만 나열하지 않고, 코드베이스 근거와 구현 계획을 읽는 사람이 문서 안에서 바로 이해할 수 있게 작성한다.
코드베이스 근거에는 파일 경로만 적지 말고, 근거가 되는 기존 코드의 짧은 excerpt 또는 code shape와 그 코드가 근거인 이유를 함께 적는다.
구현 계획에는 이 step에서 코드를 어떤 순서와 형태로 작성할지 상세히 설명하고, 필요한 경우 짧은 code block 또는 pseudocode를 포함한다.
다만 사용자 승인 전 완성된 production code 전체를 작성하지 말고, 구현자가 방향을 잃지 않을 정도의 핵심 code shape만 적는다.

### Step N: 이름
1. 목표:
   이 step이 끝났을 때 달라져야 하는 상태.
2. 코드베이스 근거:
   실제 파일, symbol, component, hook, service, API, query 단위의 위치와 Serena MCP로 확인한 symbol/참조 관계를 적는다.
   위치만 적지 말고, 사용자가 문서 안에서 바로 이해할 수 있도록 관련 기존 코드의 짧은 excerpt 또는 code shape를 함께 적는다.
   따를 기존 패턴, 관련 데이터 흐름/상태/권한, 이 위치를 변경하는 이유를 적는다.
3. 구현 계획:
   이 step에서 코드를 어떤 순서와 형태로 작성할지 상세히 적는다.
   함수 signature, type/interface, query/API 호출, 상태 변경/저장/비동기 흐름, edge/error 처리, React Native 앱이면 stable `testID`/accessibility id를 실제 구현에 가까운 sketch로 포함한다.
   필요한 경우 짧은 code block 또는 pseudocode를 포함한다.
4. 확인 방법:
   이 step 직후 실행할 targeted check 또는 수동 확인.
5. 건드리지 말아야 할 것:
   scope 밖 파일, 기존 동작, 회귀 위험이 큰 부분.

## Checks and Risks
1. 실행할 targeted check:
2. 실행하지 않을 check:
3. 주요 위험:
4. fallback:

검증 작업에서 전체 `yarn lint`는 실행하지 않는다.
대신 변경한 파일/패키지/워크스페이스에 한정된 lint, typecheck, test처럼 blast radius가 작은 검증 명령을 선택한다.

## Review Questions
구현 전에 사용자에게 승인받아야 할 결정을 적는다.
답변된 질문은 답변을 `Scope / Non-Goals`, `Approach`, `Implementation Sketch`, `Implementation Steps`, `Checks and Risks` 등 적절한 섹션에 반영한 뒤 제거한다.
답변 때문에 새로운 구현 결정, scope 경계, fallback 결정이 필요해지면 새 질문을 추가한다.
각 질문은 아래 형식을 따른다.

### Question N: 질문
1. 질문:
2. 의도:
3. 답변이 구현/검증 범위에 미치는 영향:
```

작성 후 scope, non-goals, 구현 순서, 위험을 요약하고 사용자 검토를 요청한다.

## 4. `rpv/verification-scenarios.md`

시나리오를 작성하기 전에 먼저 사용자에게 원하는 검증 시나리오가 있는지 묻는다.

사용자가 시나리오를 제공하면:

- 그대로 보존한다.
- `User-requested`로 표시한다.
- AI가 제안하는 시나리오는 추가로만 작성한다.
- 사용자 시나리오를 임의로 재해석하거나 대체하지 않는다.

사용자가 별도 시나리오를 제공하지 않으면:

- `rpv/research.md`와 `rpv/plan.md`를 바탕으로 충분히 작은 검증 세트를 작성한다.
- 상태, 권한, 저장, 라우팅, 비동기 동작, 반응형 UI, 외부 서비스가 관련되면 edge scenario를 포함한다.
- 새 기능이나 수정과 연관될 수 있는 기존 기능, 기존 사용자 흐름, 공유 컴포넌트/API/상태가 계속 정상 동작하는지 확인하는 regression/adjacent scenario를 포함한다.

포함할 내용:

```markdown
# Verification Scenarios

## Verification Environment
1. 검증 도구:
2. 실행 환경:
3. 대상 플랫폼:
4. 앱 검증인 경우 선택한 플랫폼: iOS 시뮬레이터 | Android 실기기 | iOS 시뮬레이터 + Android 실기기 | N/A
5. 앱 검증인 경우 simulator/device name / OS version / app build:
6. 앱 검증인 경우 Maestro flow 파일 또는 실행 방법:
7. 인증/계정 상태:
8. 저장된 browser/auth state 또는 fresh session 사용 여부:
9. 외부 서비스, mock/stub, secret/env 의존성:
10. 현재 환경에서 검증할 수 없는 항목과 이유:

## User-Requested Scenarios
사용자가 명시적으로 요청한 검증 시나리오.

## AI-Proposed Scenarios
research와 plan을 바탕으로 AI가 추가 제안한 시나리오.
새 기능 또는 수정과 연관될 수 있는 기존 기능이 있으면, 해당 기존 동작이 깨지지 않았는지 확인하는 regression/adjacent scenario를 포함한다.

## Manual Verification Method
사용자-facing 브라우저 동작이 있는 작업은 반드시 headed Playwright로 실제 사용자처럼 클릭, 입력, 이동, 새로고침하며 수동 검증한다.
별도의 일회성 검증 스크립트로 수동 검증을 대체하지 않는다.
브라우저 표면이 없고 React Native 앱 또는 native flow 검증 대상도 아니라면 headed Playwright 검증이 적용 불가한 이유를 명시한다.
React Native 앱 또는 native flow 검증이 필요한 경우, `verification-scenarios.md` 작성 전에 사용자에게 iOS 시뮬레이터, Android 실기기, iOS 시뮬레이터 + Android 실기기 중 어떤 플랫폼에서 검증할지 묻는다.
사용자가 선택한 플랫폼만 Required 검증 범위로 보고, 선택하지 않은 플랫폼의 남은 위험은 `Excluded Scenarios` 또는 `Risks`에 명시한다.
React Native 앱의 공식 검증은 Maestro를 사용하며, iOS는 시뮬레이터, Android는 실기기를 기준으로 한다.
iOS 시뮬레이터는 iOS Required scenario의 최종 pass evidence로 사용할 수 있다.
Android 에뮬레이터는 개발 중 보조 확인에는 사용할 수 있지만, Android Required scenario의 최종 pass evidence로 사용하지 않는다.
선택된 플랫폼 검증이 불가능하면 해당 scenario는 Passed가 아니라 Blocked로 기록하고, 이유와 필요한 simulator/device 상태를 남긴다.
Maestro flow는 좌표 기반 tap/swipe보다 stable `testID`, accessibility id, id selector, 필요한 경우 text selector를 우선 사용한다.
좌표 기반 tap/swipe는 selector, `testID`, deep link, `scrollUntilVisible`로 표현할 수 없는 경우에만 마지막 fallback으로 사용하고, 사용 이유를 기록한다.
긴 navigation이나 반복 스크롤은 가능하면 deep link, seeded state, `scrollUntilVisible`로 대체한다.

## Scenario Details

### Scenario N: 이름
1. Source: User-requested | AI-proposed
2. Priority: Required | Recommended | Optional
3. Platform coverage: iOS simulator | Android real device | iOS simulator + Android real device | Web | N/A
4. Start state:
5. Browser/auth state 또는 simulator/device/auth state:
6. Viewport 또는 simulator/device:
7. Maestro flow 또는 headed Playwright 방법:
8. 사용한 selector/testID:
9. 좌표 기반 제스처 사용 여부와 이유:
10. Steps:
11. Expected result:
12. Observable evidence:
13. Console/network checks:
14. Refresh/back-navigation checks:
15. Pass/fail criteria:
16. Status: Pending | Passed | Failed | Blocked
17. Failure observed:
18. Fix attempted:
19. Re-check result:

## Excluded Scenarios
이번 범위에서 검증하지 않는 시나리오와 그 이유.

## Review Questions
검증 범위나 우선순위에 대해 사용자에게 확인할 질문.
답변된 질문은 답변을 `Verification Environment`, `Scenario Details`, `Excluded Scenarios` 등 적절한 섹션에 반영한 뒤 제거한다.
답변 때문에 새로운 검증 환경, 플랫폼 범위, regression/adjacent scenario 결정이 필요해지면 새 질문을 추가한다.
각 질문은 아래 형식을 따른다.

### Question N: 질문
1. 질문:
2. 의도:
3. 답변이 verification scenario에 미치는 영향:
```

작성 후 사용자에게 검증 시나리오 검토를 요청한다.
사용자 피드백이 반영되고 사용자가 정확히 `승인`이라고 말하기 전에는 구현하지 않는다.

## 5. Manual Verification / Session State 관리

수동 검증 도구와 환경은 `rpv/verification-scenarios.md`의 `Verification Environment`와 `Manual Verification Method`를 따른다.

반복 검증 효율을 위해 저장된 browser/auth state를 재사용한다.

1. 역할이나 계정 조건이 다르면 별도 state를 사용한다.
2. state 이름은 사람 이름이 아니라 목적 기준으로 붙인다.
3. 로그인, 온보딩, 로그아웃, 인증 전환, 권한 경계, 첫 실행 경험을 검증할 때는 fresh session을 사용한다.
4. 저장된 state가 오래되었거나 버그를 숨길 수 있으면 새로 만든다.

## 6. 구현과 검증 루프

구현은 `rpv/plan.md`의 `Implementation Steps`를 따른다.

각 의미 있는 구현 단위 이후:

1. 관련 있고 저렴한 automated check를 실행한다. 검증 작업에서 전체 `yarn lint`는 실행하지 않고, 변경 범위에 맞는 targeted lint/typecheck/test를 사용한다.
2. 영향을 받는 필수 시나리오를 `rpv/verification-scenarios.md`에 정한 도구로 수동 검증한다. 웹은 headed Playwright, React Native/native 앱은 iOS 시뮬레이터 또는 Android 실기기에서 Maestro를 사용한다.
3. 새 기능이나 수정과 연관될 수 있는 기존 기능, 기존 사용자 흐름, 공유 컴포넌트/API/상태도 계속 정상 동작하는지 검증한다.
4. `rpv/verification-scenarios.md`에 pass/fail 결과를 기록한다.
5. 필수 시나리오가 실패하면 관찰된 실패를 기록한다.
6. 실패 원인을 구현에 매핑한다.
7. 구현을 수정한다.
8. 실패한 시나리오를 다시 검증한다.
9. 수정의 영향을 받을 수 있는 기존 통과 시나리오도 다시 검증한다.
10. 모든 필수 검증 시나리오와 관련 regression/adjacent scenario가 통과할 때까지 반복한다.

구현이 끝났다는 것은 코드 작성이 끝났다는 뜻이 아니다.
모든 필수 검증 시나리오와 관련 regression/adjacent scenario가 통과하거나, 사용자가 실패 또는 차단된 시나리오를 명시적으로 수용해야 완료다.

## 7. Hotfix Handling

RPV 진행 중 검증 실패가 기존 `plan.md`의 구현 범위를 벗어난 좁은 차단 이슈를 드러내면 hotfix 흐름을 사용한다.

hotfix를 사용할 수 있는 경우:

1. 실제 검증 중 발견된 blocker.
2. 외부 API contract drift.
3. DB column/index/schema mismatch.
4. infra route, scheduler, secret, env 설정 mismatch.
5. mobile/backend/infra 간 계약 충돌.
6. 원래 RPV 목표 달성을 막는 좁은 regression.

hotfix를 사용하면 안 되는 경우:

1. 새 기능 추가.
2. 원래 scope 확장.
3. 사용자가 승인하지 않은 production 변경.
4. 단순 리팩토링.
5. "하는 김에" 고치는 주변 코드.

hotfix 문서는 같은 RPV 폴더에 `hotfix-issue-name.md` 형식으로 저장한다.

포함할 내용:

```markdown
# Hotfix: 이름

## Trigger
1. 어떤 검증, 로그, 테스트, 사용자 확인에서 발견됐는가.
2. 원래 RPV 목표를 어떻게 막고 있는가.

## Scope
1. 이번 hotfix에 포함되는 것.
2. 포함하지 않는 것.

## Root Cause
1. 확인된 원인.
2. 추측이면 추측이라고 표시한다.

## Proposed Fix
1. 수정할 파일/모듈.
2. 수정 방식.
3. 기존 plan과 다른 점.

## Safety Constraints
1. production을 건드리는지 여부.
2. DB migration 필요 여부.
3. 사용자 컨펌이 필요한 stop gate.
4. rollback 방법.

## Verification
1. hotfix 자체 검증.
2. 원래 RPV 시나리오 중 다시 돌릴 항목.
3. 통과 기준.

## Result
1. 적용 결과.
2. 재검증 결과.
3. 원래 RPV 문서에 반영해야 할 내용.
```

hotfix 규칙:

1. hotfix는 반드시 parent RPV 목표와 연결되어야 한다.
2. 변경 범위는 blocker 해소에 필요한 최소 범위로 제한한다.
3. DB, infra, auth, notification, billing, production 관련 hotfix는 사용자 컨펌 stop gate를 둔다.
4. hotfix 적용 후 `verification-scenarios.md`에 재검증 결과를 반영한다.
5. hotfix에서 얻은 결정이 원래 계약을 바꾸면 `research.md`, `plan.md`, `verification-scenarios.md` 중 영향을 받는 문서를 갱신한다.
6. 최종 보고에는 hotfix 목록과 결과를 별도 항목으로 기록한다.

## 8. 최종 보고

최종 보고에는 다음을 포함한다.

```markdown
## 변경 사항
무엇을 바꿨는지.

## 변경한 파일/모듈
어떤 파일과 모듈을 건드렸는지.

## 검증 결과
실행한 verification scenario와 pass/fail 결과.

## Browser/Device/Auth State
검증에 사용한 browser/auth state, simulator/device/auth state, fresh session 정보를 적는다.

## Automated Checks
실행한 명령과 결과.

## Hotfixes
RPV 중 추가로 발생한 hotfix가 있으면 문서, trigger, root cause, 변경 범위, 검증 결과, 원래 RPV 문서에 반영한 내용을 기록한다.

## RPV Skill 개선 추천안
이번 작업 중 사용자와 나눈 대화, 반복된 확인, 막힌 지점, 검증 과정에서 발견한 문제를 바탕으로 RPV skill에 반영하면 좋을 개선 사항을 제안한다.

1. 추천 사항:
2. 이유:
3. 기대 효과:
4. 반영 우선순위:
5. 다음 작업 전에 반영할지 여부:

## 남은 위험
남은 risk, blocked scenario, 사용자가 수용한 실패가 있다면 명시한다.
```
