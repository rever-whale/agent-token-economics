# 3. Request, Cache Read, Cache Create, Complete 읽는 법

코딩 에이전트의 토큰 사용량을 볼 때 가장 먼저 해야 할 일은 지표를 역할별로 분리하는 것이다. 총합만 보면 "많이 썼다"는 사실만 알 수 있다. 하지만 무엇이 늘었는지 모르면 개선책이 반대로 간다.

## Request

`request`는 모델 호출 횟수다. request가 많다는 것은 대체로 다음 중 하나다.

- 작업이 너무 크게 시작되어 중간 재계획이 많았다.
- 요구사항이 불명확해서 확인 질문과 정정이 반복되었다.
- 에이전트가 필요한 파일을 찾지 못하고 탐색을 반복했다.
- 테스트 실패를 한 번에 좁히지 못했다.
- 사용자에게 너무 자주 장황하게 보고했다.

request를 줄이는 가장 좋은 방법은 한 번에 모든 것을 시키는 것이 아니다. 작업 브리프를 명확히 하고, 탐색 범위를 정하고, 검증 명령을 알려주고, 완료 조건을 분명히 하는 것이다. 큰 작업은 오히려 request를 늘릴 수 있다. 모델이 한 요청 안에서 많은 결정을 해야 하고, 실패했을 때 되돌릴 범위도 커지기 때문이다.

## Cache Read

`cache read`는 이미 캐시된 prefix를 재사용한 입력 토큰이다. 이 값이 높다는 것은 정적 컨텍스트가 안정적으로 앞쪽에 있고, 이전 request와 같은 prefix를 공유한다는 뜻이다.

좋은 cache read 후보는 다음과 같다.

- 시스템 지시
- 프로젝트 규칙
- 도구 정의
- repo 구조 요약
- coding convention
- 반복 작업의 평가 기준
- 긴 예제와 템플릿

cache read가 높으면 비용과 latency가 줄어드는 경우가 많다. Anthropic의 prompt caching 문서는 cache read token을 base input token의 0.1배 가격으로 설명하고, cache write는 5분 cache 기준 1.25배, 1시간 cache 기준 2배로 설명한다. 가격표는 공급자와 모델마다 바뀔 수 있으므로, 이 책에서는 구체 가격보다 비율 해석을 우선한다.

## Cache Create

`cache create`는 새 cache entry를 만들기 위해 쓴 입력 토큰이다. 첫 요청에서 큰 cache create가 나오는 것은 정상일 수 있다. 프로젝트 지시, 긴 specification, 공통 예제를 캐시에 올리는 과정이기 때문이다.

문제는 매 request마다 cache create가 계속 큰 경우다. 이때는 보통 cache breakpoint 앞쪽에 매번 바뀌는 값이 들어가 있다.

- timestamp
- 현재 사용자 질문
- 매번 달라지는 로그
- tool output
- random id
- "이번에는..."으로 시작하는 동적 지시

캐시는 prefix가 같을 때 유리하다. 앞쪽에 바뀌는 조각을 넣으면 뒤쪽이 아무리 같아도 prefix hash가 달라진다. 따라서 정적 지시는 앞에, 동적 입력은 뒤에 둔다.

## Complete

`complete`는 모델이 생성한 토큰이다. 일부 도구에서는 `completion_tokens`, `output_tokens`처럼 표시된다. 코딩 에이전트에서는 complete가 다음 세 가지로 쓰인다.

- 실제 산출물: 코드, 테스트, 문서, diff 설명
- 판단 과정: 계획, 가정, 비교, 원인 분석
- 커뮤니케이션: 중간 업데이트, 최종 보고, 사용자 질문

complete를 무조건 낮추면 품질이 떨어질 수 있다. 특히 위험한 변경에서는 모델이 충분히 설명하고 검증 경로를 남겨야 한다. 하지만 반복적인 자기 설명이나 과도한 요약은 줄여야 한다. 개인 repo의 `AGENTS.md`나 작업 템플릿에는 "최종 답변은 변경 파일, 검증, 남은 리스크만 보고한다"처럼 출력 정책을 넣는 것이 좋다.

## 총 입력 토큰 계산

캐시를 쓰는 환경에서는 `input_tokens`가 전체 입력을 뜻하지 않을 수 있다. Anthropic 문서는 total input을 다음처럼 계산한다.

```text
total_input_tokens = cache_read_input_tokens + cache_creation_input_tokens + input_tokens
```

이 관점은 다른 공급자에도 유용하다. 핵심은 "읽은 총량"과 "새로 처리한 비용"을 분리하는 것이다. 같은 100,000 token 입력이라도 대부분 cache read라면 운영상 의미가 다르다. 반대로 매번 100,000 token을 새로 create하고 있다면 구조가 잘못되었을 가능성이 높다.

## 지표 이름 정규화하기

도구와 API마다 usage field 이름은 다르다. 개인 작업 로그에서도 공급자별 원본 이름을 그대로 쓰기보다, 먼저 공통 schema로 정규화하는 편이 좋다.

| 공통 이름 | 의미 | 흔한 원본 이름 |
| --- | --- | --- |
| `request_count` | 모델 호출 횟수 | request, run, response, message call |
| `cache_read_tokens` | 캐시에서 읽은 입력 토큰 | cache read, cached input, `cache_read_input_tokens`, cached tokens |
| `cache_create_tokens` | 새 cache entry 생성에 쓴 입력 토큰 | cache create, cache write, `cache_creation_input_tokens` |
| `uncached_input_tokens` | 캐시 처리 밖의 일반 입력 토큰 | input, prompt, non-cached input |
| `output_tokens` | 모델이 생성한 토큰 | complete, completion, output |
| `tool_output_bytes` | 모델에 들어가기 전 도구 출력 크기 | command output bytes, log bytes, observation size |
| `retry_count` | 실패 후 재시도 횟수 | retries, repair loop count, failed attempts |

이 정규화는 숫자를 예쁘게 보이기 위한 작업이 아니다. 같은 작업을 Codex, Claude Code, Cursor, 사내 harness에서 비교하려면 "이 도구의 `complete`가 저 도구의 `output_tokens`와 같은 층인가"를 먼저 맞춰야 한다.

정규화할 때 주의할 점은 세 가지다.

- 캐시된 입력도 total context에는 포함된다. 비용은 낮을 수 있지만 attention budget 관점에서는 여전히 읽히는 정보다.
- `tool_output_bytes`는 토큰이 아니지만, 다음 request에 들어가면 입력 토큰이 된다.
- provider billable token과 agent trace token은 다를 수 있다. 개인 작업 로그에서도 둘을 분리해서 보관한다.

## 출처

- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
