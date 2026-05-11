# RPV Skill

RPV는 Research, Plan, Verify workflow를 위한 Codex skill입니다.

이 스킬은 AI가 바로 구현부터 시작하지 않고, 먼저 조사 문서, 구현 계획, 검증 시나리오를 작성한 뒤 사용자 검토를 거쳐 구현하도록 돕습니다.

## 포함 파일

1. `SKILL.md`: RPV skill 본문
2. `agents/openai.yaml`: Codex UI 표시 및 invocation metadata

## 설치

이 저장소를 Codex skills 폴더에 clone합니다.

```bash
git clone <repo-url> ~/.codex/skills/rpv
```

이미 설치되어 있다면 기존 폴더를 백업하거나 제거한 뒤 clone합니다.

## 업데이트

```bash
git -C ~/.codex/skills/rpv pull
```

## 사용

Codex에서 작업을 요청할 때 `/rpv`를 호출합니다.

```text
/rpv 이 기능 구현 전에 research, plan, verification scenarios를 먼저 작성해줘.
```

## 적합한 작업

1. 사용자-facing 기능
2. 상태, 인증, 세션, 권한, 저장, 라우팅이 얽힌 작업
3. 여러 파일이나 모듈을 건드리는 작업
4. 원인이 불확실한 버그
5. 구현 전에 제품 의도와 범위를 먼저 합의해야 하는 작업

## 핵심 흐름

1. Research
2. Plan
3. Verification Scenarios
4. Implement
5. Verify
6. Fix and Re-verify
7. Final Report
