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
ComfySims에 **통로 여유 검사(clearance check)**를 넣어 줘. 단순히 "가구 사이 거리"가 아니라, 사람이 실제로 지나갈 수 있는지를 판정하는 기능이어야 해.

**정의.** 통로는 방의 각 문에서 다른 문과 각 가구의 사용 위치(의자 빼는 공간, 침대 옆, 옷장·서랍 여는 깊이 등)까지 가는 경로야. 판정은 그 경로의 **가장 좁은 곳(병목) 폭**으로 해. 바닥에 놓인 가구는 높이와 상관없이 장애물이야(러그만 예외). 그러니 `walkObstacles`의 50 cm 규칙을 그대로 쓰지 말고, 문이 열리는 호와 가구 사용 공간도 장애물에 넣어.

**원자료부터.** 블로그나 요약 말고 원문을 가져와 조항 번호와 함께 인용해. 대상은 2010 ADA Standards §403.5.1·§304·§305·§404.2.4, ICC A117.1, 한국 「장애인·노인·임산부 등의 편의증진 보장에 관한 법률 시행규칙」 [별표 1], Size Korea 인체치수조사(어깨너비·몸 두께 백분위), Panero & Zelnik *Human Dimension & Interior Space*, NKBA 주방 가이드라인이야. 원문을 확인할 수 없는 수치는 "미확인"이라고 표시하고 기본값으로 쓰지 마. 이 수치로 프로필(혼자 통과 / 두 사람 엇갈림 / 휠체어 통과·회전)을 만들고, 기본 프로필을 무엇으로 했는지와 그 이유는 ADR로 남겨.

**방법.** `packages/placement`에 순수 함수로 구현해. 단위는 cm이고 결과는 데이터로 반환해. 병목 폭은 독립된 두 방식으로 구해.
(a) 장애물을 r만큼 Minkowski 확장(Clipper2)하고, 연결이 유지되는 최대 r을 이분 탐색으로 찾아 병목 폭 = 2r.
(b) 1 cm 격자에 정확한 EDT를 적용하고 widest-path로 찾기.
결과는 경로마다 병목 폭, 위치, 원인이 된 두 물체 id, 프로필별 등급이고, "X를 N cm 옮기면 통과" 같은 수정 제안도 넣어(`findSpot` 재사용 검토). `window.roomfit` API, MCP 도구, 에디터 오버레이까지 연결해.

**스펙 먼저.** `spec/TEMPLATE.md`로 `spec/M19-walkway-clearance.md`를 쓰고, AC 표·ADR·journal 항목을 만들고, AC id로 시작하는 `acceptance/clearance.test.ts`를 작성한 뒤 구현해.

**맞다는 증거.** 너나 내 판단 말고 다음으로 보여 줘.
1. 손으로 답을 계산할 수 있는 fixture(두 상자 80 cm → 80, L자 복도, 문 폭 하나만 열린 벽)
2. (a)와 (b)가 무작위 배치 수천 개에서 ±1 cm 안에서 일치
3. ADA 그림 403.5.1·304.3 치수로 만든 fixture가 표준과 같은 판정
4. fast-check 성질: 평행이동·회전 불변, 가구를 떨어뜨리거나 지우면 병목이 줄지 않음
5. Stryker mutation testing으로 계산을 틀리게 바꿨을 때 테스트가 잡아내는지
6. `pnpm see`로 만든 오버레이 PNG에서 병목 표시가 실제 좁은 곳에 있는지
7. 실제 방을 줄자로 잰 값과 비교하는 `manual` AC(측정은 내가 할게)

끝내기 전에 `pnpm test`, `pnpm typecheck`, `pnpm verify`를 돌리고, 통과·실패를 출력 그대로 보고해. 어느 기준 수치를 어느 조항에서 가져왔는지 표로 정리해 줘.
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
