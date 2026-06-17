# 14. 토큰 운영 성숙도 모델

토큰 운영은 한 번의 프롬프트 개선으로 끝나지 않는다. 개인 개발자가 에이전트를 쓰는 방식, repo memory를 유지하는 방식, 로그를 다루는 방식, 사후 리뷰를 하는 방식이 함께 성숙해야 한다. 이 장은 책 전체를 하나의 평가 모델로 묶는다.

## Level 0: 개인 감각에 의존한다

특징:

- 사용자가 매번 같은 규칙을 다시 설명한다.
- 에이전트가 package manager와 test command를 반복 탐색한다.
- 큰 파일과 테스트 로그를 자주 통째로 읽는다.
- request가 많아도 왜 많은지 알 수 없다.
- cache read/cache create를 보지 않는다.

대표 증상:

```text
request_count: 높음
cache_read_ratio: 낮음
cache_create: 매번 높음
tool_output_bytes: 통제 안 됨
retry_count: 높음
```

다음 단계로 가는 가장 작은 행동은 `AGENTS.md` 초안을 만들고, unknown output에 byte cap을 거는 것이다.

## Level 1: 출력 폭발을 통제한다

특징:

- `rg`로 위치를 좁힌 뒤 파일 주변부를 읽는다.
- 테스트 로그는 `tail -c`로 시작한다.
- generated file, build artifact, lockfile 전체 읽기를 피한다.
- 최종 답변에 긴 로그를 붙이지 않는다.

좋아지는 지표:

- `uncached_input_tokens`가 줄어든다.
- `tool_output_bytes`가 줄어든다.
- 같은 실패를 다시 해석하는 `retry_count`가 줄어든다.

이 단계의 함정은 "작게 읽기"가 "적게 이해하기"로 변하는 것이다. 필요한 맥락까지 잘라내면 잘못된 수정이 늘어난다. 출력 제한은 검색, 주변부 읽기, 실패 요약과 함께 써야 한다.

## Level 2: repo memory가 작동한다

특징:

- repo 루트에 짧고 구체적인 에이전트 지시 파일이 있다.
- package manager, 주요 명령, 수정 금지 경로, output policy가 명시되어 있다.
- 반복 정정이 생기면 `AGENTS.md`에 반영한다.
- 오래된 규칙을 삭제한다.

좋아지는 지표:

- 첫 request 이후 `cache_read_ratio`가 올라간다.
- command 탐색으로 인한 request가 줄어든다.
- 사용자의 반복 정정이 줄어든다.

이 단계에서 중요한 것은 문서 길이다. repo memory가 길어질수록 다시 토큰 비용이 된다. 좋은 repo memory는 "모르면 비싸지는 사실"만 담는다.

## Level 3: 작업 브리프가 표준화된다

특징:

- bugfix에는 expected/actual, 재현 단계, 실패 로그 핵심이 있다.
- feature에는 include/exclude scope와 기존 패턴이 있다.
- review에는 리뷰 우선순위와 출력 형식이 있다.
- 작업 브리프는 stable context 뒤에 붙는다.

좋아지는 지표:

- `request_count`가 줄어든다.
- scope late correction이 줄어든다.
- `files_read / files_changed` 비율이 개선된다.
- `complete`가 배경 설명보다 산출물에 쓰인다.

이 단계의 함정은 브리프를 또 하나의 긴 문서로 만드는 것이다. 브리프는 결정을 돕는 정보만 담아야 한다. 불안해서 붙인 전체 로그와 회의록은 브리프가 아니라 비용이다.

## Level 4: 지표 리뷰가 정책으로 이어진다

특징:

- 작업별 usage worksheet를 기록한다.
- 주간 token review를 한다.
- 가장 비싼 작업을 골라 원인을 분류한다.
- 리뷰 결과가 `AGENTS.md`, 브리프 템플릿, command policy 변경으로 이어진다.

좋아지는 지표:

- 작업 유형별 baseline이 생긴다.
- `cache_create_pressure`가 안정된다.
- retry가 많은 작업의 원인을 설명할 수 있다.
- 비싼 작업이 "어려워서 비쌌는지", "컨텍스트가 나빠서 비쌌는지" 구분된다.

이 단계부터 토큰 운영은 즉흥적인 개인 기량이 아니라 개인 운영 체계가 된다. 중요한 산출물은 그래프가 아니라 다음 작업 전에 바뀐 정책이다.

## Level 5: 컨텍스트 아키텍처를 설계한다

특징:

- stable context와 dynamic context가 분리되어 있다.
- long-horizon 작업에서는 compaction과 note-taking을 쓴다.
- 탐색은 필요한 evidence만 메인 컨텍스트에 남긴다.
- 큰 작업은 sub-agent나 별도 탐색 단계로 분리한다.
- 사내 harness나 telemetry가 공통 schema로 지표를 수집한다.

좋아지는 지표:

- 반복 작업에서 `cache_read_ratio`가 높다.
- `steady_cache_create_avg`가 낮다.
- 큰 작업에서도 request가 milestone 단위로 설명된다.
- 탐색 noise가 메인 대화에 오래 남지 않는다.

이 단계의 목표는 토큰을 최소화하는 것이 아니다. 고난도 작업에 충분한 컨텍스트를 쓰되, 버려야 할 컨텍스트를 오래 끌고 다니지 않는 것이다.

## 성숙도 진단표

| 질문 | 아니오라면 |
| --- | --- |
| repo에 짧은 에이전트 지시 파일이 있는가? | Level 1에서 AGENTS.md를 만든다 |
| unknown output과 test log에 cap이 있는가? | Level 1 output policy를 먼저 도입한다 |
| bugfix/feature/review 브리프가 구분되는가? | Level 3 템플릿을 쓴다 |
| 작업별 request/cache read/cache create/complete를 기록하는가? | Level 4 worksheet부터 시작한다 |
| 비싼 작업 리뷰가 정책 변경으로 끝나는가? | 리뷰 템플릿의 Action Items를 강제한다 |
| stable/dynamic context 순서가 고정되어 있는가? | Level 5 cache-friendly prompt skeleton을 적용한다 |

## 다음 단계 선택법

어디부터 시작해야 할지 모르겠다면 다음 순서로 간다.

1. 로그와 명령 출력 cap을 만든다.
2. AGENTS.md 첫 버전을 만든다.
3. bugfix 브리프 템플릿을 도입한다.
4. 10개 작업의 usage worksheet를 작성한다.
5. 가장 비싼 작업 3개를 리뷰한다.
6. cache read ratio와 cache create pressure를 본다.
7. stable/dynamic context 순서를 고정한다.

이 순서는 일부러 단순하다. 토큰 운영은 복잡한 대시보드에서 시작하지 않는다. 매번 같은 실수를 줄이는 작고 구체적인 정책에서 시작한다.

## 출처

- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Austin1serb/agents-md](https://github.com/Austin1serb/agents-md)
- [microsoft/fastcontext](https://github.com/microsoft/fastcontext)
