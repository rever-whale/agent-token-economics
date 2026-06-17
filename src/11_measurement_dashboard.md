# 11. 측정 대시보드와 리뷰 루프

토큰 운영은 측정하지 않으면 감으로 흐른다. "이번 작업은 비쌌다"는 느낌보다 어떤 지표가 왜 늘었는지 보는 대시보드가 필요하다.

## 최소 수집 항목

작업 단위로 다음을 기록한다.

- task id
- task type: bugfix, feature, refactor, docs, review
- request count
- cache read tokens
- cache create tokens
- uncached input tokens
- complete/output tokens
- wall-clock time
- files read
- files changed
- largest tool output
- test runs
- retry count
- result: success, partial, blocked

처음부터 완벽한 telemetry를 만들 필요는 없다. 수동 표로 20개 작업만 모아도 병목이 보인다.

## 파생 지표

기본 지표에서 다음을 계산한다.

```text
cache_read_ratio = cache_read / total_input
cache_create_pressure = cache_create / request
completion_share = complete / (total_input + complete)
retry_rate = retry_count / request
changed_file_precision = files_changed / files_read
```

`changed_file_precision`은 탐색 효율을 보는 간단한 지표다. 파일을 40개 읽고 1개만 바꿨다면 탐색이 넓었거나 작업이 모호했을 수 있다. 물론 큰 리팩터링에서는 정상일 수 있으므로 task type별로 봐야 한다.

## 리뷰 루프

매주 또는 sprint마다 다음 질문으로 리뷰한다.

- cache read ratio가 낮은 작업 유형은 무엇인가?
- cache create가 매 request마다 높은 세션은 어떤 구조였는가?
- 가장 큰 tool output 10개 중 실제 결정에 쓰인 것은 몇 개인가?
- retry가 많은 작업은 요구사항 문제였나, 테스트 문제였나, 탐색 문제였나?
- AGENTS.md에 추가하면 다음번에 줄일 수 있는 반복 정정은 무엇인가?
- 최종 답변이 너무 길거나 너무 짧아 재작업을 만든 사례는 무엇인가?

리뷰 결과는 정책으로 바뀌어야 한다. 예를 들어 "테스트 로그는 tail-c 12000부터"라는 규칙, "모든 bugfix 브리프에는 재현 단계와 expected/actual 포함" 같은 규칙이 생긴다.

## 목표치 설정

처음부터 산업 표준 목표치를 찾으려 하지 말고 개인 baseline을 만든다.

예시:

- bugfix 평균 request를 8에서 5로 낮춘다.
- 반복 작업 cache read ratio를 60% 이상으로 유지한다.
- steady-state cache create를 첫 request 대비 20% 이하로 유지한다.
- retry 2회 이상 작업은 사후 리뷰한다.
- largest tool output이 50KB를 넘으면 이유를 남긴다.

목표는 자신의 작업 품질을 개선하기 위한 것이지 에이전트를 억지로 침묵시키기 위한 것이 아니다. 토큰을 덜 쓰고도 실패가 늘면 잘못된 최적화다.

## 수동 대시보드로 시작하기

처음에는 CSV 하나면 충분하다.

```csv
task_id,date,task_type,agent,request_count,cache_read_tokens,cache_create_tokens,uncached_input_tokens,output_tokens,tool_output_bytes,files_read,files_changed,test_runs,retry_count,result,notes
AUTH-142,2026-06-17,bugfix,codex,5,72000,19000,8500,6200,18000,6,2,3,1,success,"login error reset"
UI-088,2026-06-17,feature,claude-code,14,210000,42000,39000,28000,64000,18,7,5,2,success,"table filter"
DOC-031,2026-06-17,docs,cursor,2,18000,6000,1200,4200,2000,3,1,0,0,success,"README update"
```

이 CSV는 세 가지 질문에 답할 수 있어야 한다.

- 어떤 task type이 request를 많이 쓰는가?
- 어떤 agent 또는 repo에서 cache read가 낮은가?
- 어떤 작업이 files read 대비 files changed가 낮은가?

자동화는 나중 문제다. 먼저 사람이 읽는 표로 시작해야 숫자의 의미를 배운다.

## 주간 리뷰 템플릿

```markdown
## Agent Token Review

Period:
Repositories:
Tasks reviewed:

### Biggest Spenders
- Task:
- Why expensive:
- Avoidable:
- Policy change:

### Cache Health
- Best cache read ratio:
- Worst cache read ratio:
- Repeated cache create cause:

### Tool Output
- Largest output:
- Was it needed:
- New output cap rule:

### Action Items
- AGENTS.md update:
- Brief template update:
- Test/log command update:
```

리뷰의 산출물은 그래프가 아니라 정책 변경이어야 한다. 숫자를 보고도 `AGENTS.md`, 작업 브리프, 검증 명령, 로그 캡 규칙이 바뀌지 않으면 다음 주에도 같은 비용을 낸다.

## 출처

- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [microsoft/fastcontext](https://github.com/microsoft/fastcontext)
