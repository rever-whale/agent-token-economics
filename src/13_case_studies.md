# 13. 세 가지 작업으로 보는 토큰 운영

이 장은 앞에서 만든 지표와 템플릿을 실제 작업 흐름에 연결한다. 숫자는 특정 vendor의 청구 예시가 아니라 개인 작업 리뷰를 위한 가상 trace다. 목적은 "얼마가 정답인가"가 아니라 "어떤 패턴이면 어디를 고쳐야 하는가"를 읽는 것이다.

## 사례 1: 작은 Bugfix가 비싸지는 순간

상황:

- 로그인 실패 후 error banner가 다음 submit에서 사라지지 않는다.
- 실제 수정은 `LoginForm.tsx`와 `useLogin.ts` 두 파일이면 충분하다.
- 하지만 초기 브리프에는 재현 단계와 expected/actual이 없다.

나쁜 진행:

| 단계 | 행동 | 지표 변화 |
| --- | --- | --- |
| 1 | "로그인 에러 고쳐줘"만 전달 | request 증가 가능성 시작 |
| 2 | 에이전트가 auth 폴더 전체와 shared UI를 넓게 탐색 | uncached input, files read 증가 |
| 3 | 전체 테스트 로그를 그대로 읽음 | tool output bytes와 uncached input 급증 |
| 4 | 실패 원인을 API timeout으로 오해 | retry 증가 |
| 5 | 사용자가 expected/actual을 뒤늦게 설명 | request와 complete 추가 발생 |

나쁜 trace:

```text
request_count: 9
cache_read_tokens: 24,000
cache_create_tokens: 76,000
uncached_input_tokens: 91,000
output_tokens: 17,000
tool_output_bytes: 180,000
files_read: 21
files_changed: 2
retry_count: 4
```

핵심 문제는 모델 성능이 아니다. 작업 브리프가 약했고, 로그 출력이 커졌고, 실패 요약이 다음 request에 압축되지 않았다.

좋은 진행:

```markdown
## Goal
로그인 실패 후 다음 submit에서도 error banner가 남는 문제를 수정한다.

## Expected / Actual
- Expected: 새 submit이 시작되면 이전 error banner가 reset된다.
- Actual: 이전 error banner가 남아 사용자가 새 시도를 구분하기 어렵다.

## Evidence
- Error state는 `LoginForm.tsx` submit handler에서 관리된다.
- Login mutation은 `useLogin.ts`를 통해 호출된다.

## Verification
- `pnpm test LoginForm`
```

좋은 trace:

```text
request_count: 4
cache_read_tokens: 68,000
cache_create_tokens: 18,000
uncached_input_tokens: 7,800
output_tokens: 5,900
tool_output_bytes: 14,000
files_read: 5
files_changed: 2
retry_count: 1
```

줄어든 값:

- `request`: expected/actual과 검증 명령이 있어 확인 왕복이 줄었다.
- `uncached_input`: auth 폴더 전체 대신 의심 파일 주변부만 읽었다.
- `cache_create`: 동적 로그가 stable prefix를 깨지 않았다.
- `complete`: 모델이 긴 가능성 목록 대신 patch와 검증 보고에 출력 토큰을 썼다.

## 사례 2: Feature 작업에서 scope가 흔들릴 때

상황:

- product table에 상태 필터를 추가한다.
- 기존 table component와 URL query sync 패턴이 이미 있다.
- 사용자는 "필터 추가"만 요청하고, URL sync 여부와 접근성 요구를 말하지 않았다.

나쁜 진행은 보통 이런 모양이다.

- 에이전트가 table state, route state, API query, design component를 모두 탐색한다.
- 구현 후 사용자가 "URL에 남아야 한다"고 정정한다.
- 다시 route hook을 읽고 상태 구조를 바꾼다.
- 테스트가 깨지고, snapshot diff를 크게 읽는다.

나쁜 trace:

```text
request_count: 16
cache_read_tokens: 110,000
cache_create_tokens: 64,000
uncached_input_tokens: 72,000
output_tokens: 31,000
files_read: 29
files_changed: 8
test_runs: 6
retry_count: 3
```

이 경우 줄여야 할 것은 단순히 output이 아니다. feature 작업은 원래 output이 크다. 문제는 scope가 늦게 확정되어 files read와 retry가 커진 것이다.

좋은 feature 브리프:

```markdown
## Goal
Product table에 status 필터를 추가한다.

## Scope
- Include: UI segmented control, URL query sync, table filtering.
- Exclude: server API 변경, saved filter preset.

## Existing Patterns
- Similar URL sync: `useProductSearchParams`
- Similar filter UI: `OrderStatusFilter`
- Similar test: `ProductTable.filters.test.tsx`

## Verification
- `pnpm test ProductTable`
- URL query가 reload 후 유지되는지 확인.
```

좋은 trace:

```text
request_count: 9
cache_read_tokens: 142,000
cache_create_tokens: 30,000
uncached_input_tokens: 31,000
output_tokens: 24,000
files_read: 13
files_changed: 6
test_runs: 4
retry_count: 1
```

줄어든 값:

- `request`: late correction이 줄었다.
- `files_read`: similar pattern을 알려줘 탐색 후보가 좁아졌다.
- `cache_create`: feature scope가 stable하게 유지되었다.
- `retry_count`: URL sync 요구가 처음부터 반영되었다.

## 사례 3: Review 작업에서 complete가 낭비될 때

상황:

- PR diff를 리뷰한다.
- 사용자는 bug risk와 missing test를 원한다.
- 에이전트가 diff 전체를 장황하게 요약한다.

나쁜 리뷰 출력은 다음과 같다.

- 변경 요약이 길다.
- 코드 스타일 칭찬이 많다.
- findings보다 배경 설명이 먼저 나온다.
- 같은 위험을 여러 문단으로 반복한다.
- line reference가 없다.

나쁜 trace:

```text
request_count: 5
cache_read_tokens: 35,000
cache_create_tokens: 18,000
uncached_input_tokens: 42,000
output_tokens: 22,000
files_read: 12
findings: 2
```

이 작업에서 `output_tokens`가 높은 것은 코드 생성 때문이 아니다. 리뷰 산출물과 무관한 요약이 complete를 태운 것이다.

좋은 review brief:

```markdown
## Review Priorities
1. Correctness
2. Regression risk
3. Missing tests

## Output Format
- Findings first.
- Include file/line references.
- No broad summary unless there are no findings.
- Mention test gaps separately.
```

좋은 trace:

```text
request_count: 3
cache_read_tokens: 41,000
cache_create_tokens: 9,000
uncached_input_tokens: 18,000
output_tokens: 7,500
files_read: 7
findings: 2
```

줄어든 값:

- `complete`: output policy가 findings 중심으로 고정되었다.
- `uncached_input`: diff 전체 반복 읽기 대신 관련 파일과 hunk만 읽었다.
- `request`: 리뷰 기준을 다시 묻지 않았다.

## 세 사례에서 공통으로 보이는 것

토큰 낭비는 보통 한 지표만 커지는 형태가 아니다. 하나가 흔들리면 다른 지표도 따라 흔들린다.

| 원인 | 먼저 커지는 값 | 뒤따라 커지는 값 |
| --- | --- | --- |
| 작업 브리프 부족 | request | complete, files read |
| 전체 로그 읽기 | uncached input | cache create, retry |
| scope late correction | request | files changed, test runs |
| output policy 부재 | complete | request |
| cache-breaking prompt | cache create | latency, cost |

따라서 운영 개선도 한 줄 규칙이 아니라 연결된 정책이어야 한다.

- AGENTS.md는 반복 지시를 줄인다.
- 작업 브리프는 request와 retry를 줄인다.
- output cap은 uncached input을 줄인다.
- stable/dynamic context 분리는 cache create를 줄인다.
- review output policy는 complete를 줄인다.

## 사례 리뷰 템플릿

비싼 작업 하나를 골라 다음 순서로 리뷰한다.

```markdown
## Case Review

Task:
Task type:
Result:

### What got expensive?

- request:
- cache read:
- cache create:
- uncached input:
- complete:
- tool output:

### Why?

- Missing brief:
- Missing repo memory:
- Large output:
- Late correction:
- Test failure:
- Output policy:

### Next Policy

- AGENTS.md update:
- Brief template update:
- Command output policy:
- Cache layout change:
- Dashboard metric:
```

이 템플릿의 마지막 줄이 중요하다. 사례 리뷰는 반성이 아니라 정책 변경으로 끝나야 한다.

## 출처

- [Austin1serb/agents-md](https://github.com/Austin1serb/agents-md)
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
