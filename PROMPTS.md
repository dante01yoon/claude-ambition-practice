# 영상 속 프롬프트 원문

영상 「Anthropic 엔지니어가 Claude Code를 쓰는 법」에서 실제로 입력한 프롬프트예요. 시연 프로젝트는 [ComfySims](https://github.com/dante01yoon/ComfySims)입니다.
원래 글: Felix Rieseberg, [How to be more ambitious: A tutorial for humans with models](https://felixrieseberg.com/how-to-be-more-ambitious-a-tutorial-for-humans-with-models/)

## 1. 고도 — 할 일만

```text
편집기 오른쪽 패널에 지금 배치를 요약하는 패널을 추가해 줘. fixtures/layouts/demo-house.json으로 확인해 줘.
```

## 2. 고도 — 할 일 + 목표

```text
편집기 오른쪽 패널에 지금 배치를 요약하는 패널을 추가해 줘. fixtures/layouts/demo-house.json으로 확인해 줘. 이 패널의 목적은 사용자가 가구를 사기 전에, 정말 우리 집에 들어가고 지낼 만한지 확신하게 하는 거야. 숫자만 늘어놓지 말고, 지금 무엇이 그 확신을 막고 있는지와 사기 전에 내려야 할 결정 세 가지를 근거와 함께 보여 줘.
```

## 3. 깊이 — 깊이 프롬프트로 다시 쓰기

```text
가구 사이 통로가 사람이 지나다니기에 충분한지 확인하는 기능을 넣어 줘.

이 일의 더 큰 버전 말고 더 깊은 버전을 원해. 다시 쓰기 전에 (1) 이 분야 최고 전문가라면 내가 요청하지 않은 무엇을 할지 (2) 이미 정리된 자료 말고 어떤 가장 원래의 자료에서 시작할지 (3) 어떤 구체적인 방법과 도구를 쓸지 (4) 너나 나 말고 무엇이 결과가 맞다는 걸 보여 줄지 말해 줘. 그다음 정확히 그걸 요청하도록 내 프롬프트를 다시 써 줘.
```

이어서:

```text
좋아. 다시 쓴 프롬프트만 deep-prompt.md 파일로 저장해 줘.
```

새 세션에서:

```bash
claude "$(cat deep-prompt.md)"
```

작업이 길어 끊겼을 때 이어서(`claude --continue`):

```text
좋아. 하던 구현을 끝까지 마무리해 줘. 스펙·테스트·화면 오버레이까지 다 하고, 직접 확인한 결과를 짧게 알려 줘.
```

클로드가 다시 쓴 프롬프트(`deep-prompt.md`):

```text
가구 사이 통로가 사람이 지나다니기에 충분한지 판정하는 기능을 만들어 줘. 범위를 넓히지 말고 판정의 정확성과 근거를 깊게 파 줘. 자동 재배치나 최적화는 하지 마.

**정의.** 통로 폭은 경로의 성질로 다뤄. 방의 문↔문, 그리고 문↔가구를 쓰는 면(침대 양옆, 소파 앞, 옷장 앞 등) 경로마다 "가장 좁은 곳의 폭, 그 위치, 원인이 된 두 물체"를 결과 데이터로 돌려줘. 빈 공간 = 방 내부 − 벽(문 열림 구간 제외) − 가구의 listed-size 바닥 면적이야. 폭 w로 지나갈 수 있다는 건 빈 공간을 w/2만큼 깎아도 경로가 이어져 있다는 뜻이고, 최대 통과 폭은 병목 경로 문제로 구해. 회전된 가구의 대각선 틈은 유클리드 거리로 재.

**기존 규칙과의 관계.** M15의 walker(반지름 25 cm, 높이 50 cm 미만 가구는 무시)는 걷기 화면용이야. 통로 판정에서는 낮은 가구도 막는 것으로 쳐. 이 차이와 이유를 ADR로 남기고, walker 규칙 자체는 바꾸지 마.

**기준.** 최소 / 편안함 / 휠체어 프로필을 두고, 숫자마다 원문 출처와 조항을 붙여. 출처는 ADA 2010 §403.5.1·§304, ICC A117.1, 장애인등편의법 시행규칙 별표 1, NKBA Planning Guidelines, Size Korea 인체치수 원자료, ISO 7250-1이야. 요약 블로그 숫자는 쓰지 말고, 원문에서 확인 못 한 값은 확인 못 했다고 적어.

**구현.** `packages/placement`에 순수 함수로 넣어. 기존 `nearestOn` 거리로 1 cm 격자 거리장을 만들고, union-find로 병목을 구해. 격자 오차 상한을 명시해. 결과는 throw 말고 데이터로, 단위는 cm야. `window.roomfit` API와 MCP 도구로 공개하고, 편집기에서 병목을 표시하고, `pnpm see` PNG에서도 보이게 해. 스펙 `spec/M19-…`(AC 표), ADR, journal, `acceptance/` 테스트를 함께 써.

**검증. 나나 네 말이 아니라 이것들이 증명하게 해.**

1. 정답이 손으로 계산되는 fixture: 평행한 틈 80 cm, 37° 회전된 모서리끼리의 대각선 틈, 경로가 없는 방
2. 판정이 "w 통과"면 반지름 w/2인 `walkerStep`이 실제로 통과하고, "불가"면 막히는지
3. 무작위 배치 수백 개에서 Python Shapely(`buffer(-r)` + 연결성) 독립 구현과 오차 상한 안에서 일치하는지
4. fast-check 성질 테스트: 평행이동·회전해도 값이 같고, 가구를 떼거나 지워도 폭이 줄지 않는지
5. `fixtures/layouts/demo-house.json`의 병목 리포트를 PNG로 확인

`pnpm test`, `pnpm typecheck`, `pnpm see`로 확인하고, 각 검증 결과를 실제 출력으로 보고해 줘.
```

## 4. 바깥 심판 — Codex로 검토받기

```text
방금 만든 통로 확인 기능을 설치된 Codex CLI(codex exec)로 깊게 검토받아 줘. 만든 너 말고 다른 모델이 보게 하는 거야. 나온 지적은 근거를 확인해서 맞는 것만 고쳐 줘.
```

## 5. 범위 — 명사를 동사로, 분야마다 서브에이전트

```text
ComfySims는 '3D 집 편집기'가 아니야. 하는 일은 '사려는 가구가 우리 집에 맞고 그 집에서 지낼 만한지, 사기 전에 믿고 결정하게 하는 것'이야. 이 일을 가장 잘하는 분야를 가능한 한 많이 골라서, 분야마다 서브에이전트를 하나씩 띄워 줘. 그 분야 사람처럼 말하게 하지 말고 그 사람이 실제로 하는 일을 하게 해. 한 명씩 차례로 편집기를 보고 고친 다음 다음 사람에게 넘겨 줘.
```

이어서(같은 세션, `claude --continue`):

```text
4번 분야까지만 마무리하고 멈춰 줘. 그다음 지금까지 끝난 분야마다 무엇을 보고 무엇을 고쳤는지 표로 정리해 줘. 남은 분야는 다음에 이어서 할게.
```

## 펠릭스의 깊이 프롬프트 원문(영어)

```text
I want a deeper version of this task, not a bigger one. Before rewriting it: (1) say what a top professional in this field would do here that I'm not asking for; (2) name the rawest material they would start from instead of what has already been digested for me; (3) name the specific methods and tools they would use; (4) say what, other than you or me, would show the result is right. Then rewrite my prompt to ask for exactly that.
```
