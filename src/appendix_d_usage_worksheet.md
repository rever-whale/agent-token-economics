# D. 지표 워크시트와 예시 Trace

이 부록은 개인 개발자가 바로 복사해서 쓸 수 있는 기록 양식이다. 처음부터 자동 수집을 만들 필요는 없다. 10개 작업만 수동으로 기록해도 어디서 토큰이 새는지 보인다.

## 작업 단위 기록지

```markdown
## Task Usage Record

- Task ID:
- Date:
- Repository:
- Agent:
- Task type:
- Result:

### Usage

- Request count:
- Cache read tokens:
- Cache create tokens:
- Uncached input tokens:
- Output/complete tokens:
- Largest tool output:
- Tool output bytes:

### Work Shape

- Files read:
- Files changed:
- Test runs:
- Retry count:
- Blocked by:

### Review

- Was the largest tool output necessary:
- Did cache read increase after request 1:
- Did cache create stay high after request 1:
- Was completion mostly code/diff/verification:
- What should be added to AGENTS.md:
- What should be added to the task brief template:
```

## 작업 유형별 기대 패턴

| 작업 유형 | 좋은 패턴 | 경고 패턴 |
| --- | --- | --- |
| docs | request가 적고 output share가 높음 | 파일 탐색이 넓고 cache create가 반복됨 |
| small bugfix | files read가 작고 retry가 낮음 | 테스트 로그가 uncached input을 지배함 |
| feature | request와 output이 중간 이상, 검증이 명확함 | scope가 흔들려 files read와 retry가 계속 증가함 |
| refactor | files changed가 많고 test_runs가 많음 | complete가 설명에 치우치고 검증이 부족함 |
| review | output이 findings에 집중됨 | diff 전체를 반복해서 읽고 같은 내용을 요약함 |

## 예시 1: 건강한 Bugfix

```text
task_id: AUTH-142
task_type: bugfix
request_count: 5
cache_read_tokens: 72,000
cache_create_tokens: 19,000
uncached_input_tokens: 8,500
output_tokens: 6,200
tool_output_bytes: 18,000
files_read: 6
files_changed: 2
test_runs: 3
retry_count: 1
result: success
```

해석:

- 첫 request에서 repo 규칙과 작업 맥락을 캐시에 올린 뒤 cache read가 잘 유지되었다.
- files read 대비 files changed가 3:1로 나쁘지 않다.
- retry가 1회 있었지만 test output이 과도하게 커지지 않았다.
- 다음 작업을 위해 AGENTS.md에 추가할 내용은 없다. 작업 브리프가 충분했다.

## 예시 2: 캐시가 깨진 Bugfix

```text
task_id: BILLING-209
task_type: bugfix
request_count: 9
cache_read_tokens: 21,000
cache_create_tokens: 88,000
uncached_input_tokens: 114,000
output_tokens: 19,000
tool_output_bytes: 220,000
files_read: 24
files_changed: 1
test_runs: 5
retry_count: 4
result: success
```

해석:

- cache create가 request마다 높다. 동적 로그나 timestamp가 cacheable prefix 앞쪽에 있었을 가능성이 높다.
- files read가 24인데 changed는 1이다. 탐색 범위가 넓었거나 초기 브리프가 약했다.
- tool output bytes가 크다. 테스트 로그를 전체로 읽었을 가능성이 있다.
- 처방은 로그 tail cap, 관련 파일 후보 요청, 실패 요약 유지다.

## 예시 3: Output이 비싼 문서 작업

```text
task_id: DOC-031
task_type: docs
request_count: 2
cache_read_tokens: 18,000
cache_create_tokens: 6,000
uncached_input_tokens: 1,200
output_tokens: 4,200
tool_output_bytes: 2,000
files_read: 3
files_changed: 1
test_runs: 0
retry_count: 0
result: success
```

해석:

- output share가 높지만 문서 작업에서는 정상이다.
- request와 tool output이 낮고, 변경 파일이 명확하다.
- 이 작업은 토큰 낭비 사례가 아니다. complete가 실제 산출물로 쓰였기 때문이다.

## 회고 질문

- 비싼 작업은 정말 어려웠는가, 아니면 컨텍스트가 지저분했는가?
- cache read가 낮은 작업은 정적 지시를 재사용하지 못했는가?
- cache create가 높은 작업은 동적 입력을 앞쪽에 넣었는가?
- 가장 큰 tool output은 다음 결정을 바꾸었는가?
- 다음번에는 작업 브리프, AGENTS.md, command policy 중 무엇을 바꾸면 되는가?

## 출처

- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Austin1serb/agents-md](https://github.com/Austin1serb/agents-md)
