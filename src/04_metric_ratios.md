# 4. 좋은 비율과 나쁜 비율

토큰 지표는 절대값보다 비율로 볼 때 유용하다. 프로젝트 크기, 모델, 도구, 작업 유형에 따라 총량은 달라지지만, 비율은 작업 방식의 건강 상태를 드러낸다.

이 장의 숫자는 고정 규칙이 아니라 시작점이다. 개인 개발자는 2~4주 정도 자신의 실제 작업 trace를 모아 baseline을 만들어야 한다.

## Cache Read Ratio

```text
cache_read_ratio = cache_read / (cache_read + cache_create + uncached_input)
```

좋은 반복 작업에서는 이 비율이 높아야 한다. 같은 repo에서 비슷한 작업을 여러 번 수행하는데 cache read ratio가 낮다면 프로젝트 지시나 공통 컨텍스트가 안정적으로 재사용되지 않는다는 뜻이다.

권장 해석:

- 0~20%: 캐시를 거의 못 쓰는 상태. 지시 위치, breakpoint, 동적 입력 위치를 점검한다.
- 20~60%: 혼합 상태. 초기 탐색이나 작업별 컨텍스트가 큰 경우 가능하다.
- 60~90%: 반복 작업에 건강한 상태. 정적 prefix가 잘 유지되고 있다.
- 90% 이상: 매우 좋은 상태일 수 있지만, 동적 요구사항을 충분히 반영하지 못하는지 확인한다.

## Cache Create Pressure

```text
cache_create_pressure = cache_create / request
```

첫 request의 cache create는 투자다. 하지만 request마다 create가 높으면 비용이 새고 있다. 특히 같은 세션의 2번째, 3번째 요청에서도 create가 계속 크면 캐시를 깨는 동적 prefix를 의심한다.

실무에서는 첫 request와 이후 request를 분리해서 본다.

```text
initial_cache_create = request_1.cache_create
steady_cache_create_avg = avg(request_2..n.cache_create)
```

건강한 패턴은 `initial_cache_create`가 높고 `steady_cache_create_avg`가 낮은 것이다. 나쁜 패턴은 둘 다 높거나, request가 늘수록 create가 증가하는 것이다.

## Completion Share

```text
completion_share = complete / (cache_read + cache_create + uncached_input + complete)
```

completion share가 높다는 것은 모델이 많이 쓰고 있다는 뜻이다. 코드 생성이 많은 작업에서는 정상이다. 하지만 수정량이 작고 설명이 많은 작업에서 높다면 보고 정책이 장황하거나, 에이전트가 불확실성을 말로만 처리하고 있을 수 있다.

권장 해석:

- 작은 수정/질문 답변: 10~30%
- 중간 규모 코드 변경: 20~50%
- 문서 생성/리팩터링 설명: 40% 이상 가능

중요한 것은 complete 자체가 아니라 산출물 밀도다. `complete per changed line`, `complete per passing test`, `complete per resolved issue`처럼 성과와 묶어야 한다.

## Request-to-Outcome Ratio

```text
request_to_outcome = request / completed_task
```

작업 하나를 완료하는 데 request가 얼마나 드는지 보는 지표다. request가 늘어난다고 항상 나쁜 것은 아니다. 위험한 마이그레이션은 여러 단계 확인이 필요하다. 그러나 비슷한 bug fix의 request가 계속 늘어난다면 작업 브리프, repo memory, 테스트 명령, 로그 캡 규칙 중 하나가 부족하다는 뜻이다.

개인 baseline 예시:

- 단순 문서 수정: 1~2 request
- 작은 버그 수정: 3~8 request
- 중간 기능 추가: 8~20 request
- 대형 마이그레이션: milestone 단위로 분리해서 측정

## Tool Output Waste

```text
tool_output_waste = irrelevant_tool_output_tokens / total_tool_output_tokens
```

이 지표는 자동 계산하기 어렵지만 리뷰에서 가장 강력하다. 작업에 필요 없던 로그, 파일 전체, 중복 검색 결과를 표시해보면 낭비가 보인다. 정량화가 어렵다면 간단히 "이번 작업에서 가장 큰 도구 출력 3개가 실제 결정에 쓰였는가"를 리뷰한다.

## 좋은 상태의 조합

반복되는 repo 작업에서 좋은 조합은 보통 다음과 같다.

- request는 작업 난이도에 비례해서 증가한다.
- cache read ratio는 세션이 진행될수록 높아진다.
- cache create는 초반에 높고 이후 낮아진다.
- complete는 코드 diff와 검증 보고에 집중된다.
- 도구 출력은 byte cap 또는 주변부 읽기로 제한된다.
- retry count가 늘 때 로그 크기는 통제된다.

나쁜 조합은 이렇다.

- request가 많고 cache read는 낮다.
- cache create가 매번 높다.
- complete가 길지만 diff는 작다.
- 테스트 실패 로그가 매번 크게 들어온다.
- 사용자가 같은 제약을 반복해서 알려준다.

## 두 개의 예시 Trace

다음 숫자는 특정 vendor 청구 방식이 아니라 개인 작업 리뷰를 위한 가상 trace다.

좋은 반복 bugfix 세션:

| request | cache read | cache create | uncached input | complete | 해석 |
| --- | ---: | ---: | ---: | ---: | --- |
| 1 | 0 | 18,000 | 2,500 | 1,200 | repo 규칙과 작업 브리프를 처음 캐시에 올림 |
| 2 | 17,800 | 600 | 1,400 | 900 | 관련 파일 주변부만 읽고 수정 계획 확정 |
| 3 | 18,200 | 300 | 2,200 | 1,600 | patch 생성과 테스트 실행 |
| 4 | 18,400 | 200 | 1,000 | 700 | 최종 검증과 보고 |

이 세션은 첫 request의 cache create가 크지만 이후 request에서 cache read가 안정적으로 올라간다. complete도 대부분 계획, patch, 검증 보고에 쓰인다. 도구 출력이 통제되어 uncached input이 크게 튀지 않는다.

나쁜 로그 폭발 세션:

| request | cache read | cache create | uncached input | complete | 해석 |
| --- | ---: | ---: | ---: | ---: | --- |
| 1 | 0 | 16,000 | 3,000 | 1,000 | 시작은 정상 |
| 2 | 2,000 | 15,500 | 48,000 | 2,600 | 전체 테스트 로그가 앞쪽 context를 오염 |
| 3 | 1,800 | 16,200 | 42,000 | 3,200 | 같은 로그를 다시 읽고 다른 가설로 이동 |
| 4 | 1,500 | 15,900 | 39,000 | 2,900 | cache read가 낮고 create가 반복됨 |

이 세션은 cache read가 계속 낮고 cache create가 반복된다. 로그가 동적 prefix 근처에 들어가 캐시 효율을 깨고, 모델이 같은 실패를 여러 방식으로 재해석하느라 complete도 커진다. 이때의 처방은 "모델에게 더 자세히 설명하라"가 아니라 로그를 tail과 keyword search로 줄이고, 실패 요약만 다음 request에 남기는 것이다.

## 출처

- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [Austin1serb/agents-md](https://github.com/Austin1serb/agents-md)
