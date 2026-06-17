# 16. work_vault 사례 분석

이 장은 실제 로컬 vault인 `work_vault`를 이 책의 기준으로 분석한 사례다. 분석 대상은 제품 코드 저장소가 아니라, 여러 프로젝트의 AI 작업 맥락, 하네스 문서, 토픽, 세션 기록을 관리하는 Markdown vault다.

분석 대상 경로:

```text
~/Desktop/workspace/public/work_vault
```

이 분석은 읽기 전용으로 수행했다. 실제 `work_vault` 파일은 수정하지 않았다.

## 1. 구조 요약

`work_vault`는 다음 구조를 가진다.

```text
work_vault/
  AGENTS.md
  CONTINUATION_INDEX.md
  README.md
  projects/
    <project-slug>/
      AGENTS.md
      PROJECT.md
      ai-harness/
      topics/
        <topic-slug>/
          AGENTS.md
          TOPIC.md
          sessions/
  templates/
```

확인한 규모는 다음과 같다.

| 항목 | 관찰값 |
| --- | ---: |
| Markdown 파일 | 약 245개 |
| 세션 파일 | 35개 |
| `Usage And Cost` 관련 기록/템플릿 | 30개 파일 |
| 토큰값이 `unknown`으로 남은 세션/문서 | 최소 24개 |
| 루트 `AGENTS.md` 크기 | 약 10.6KB |
| 가장 큰 세션 파일 | 약 21.8KB |
| 큰 하네스 문서 예 | `FEATURE_LIST.md` 약 20KB |

이 vault는 단순 메모장이 아니라, 개인 개발자가 여러 작업을 이어가기 위한 작은 agent memory system에 가깝다.

## 2. 이미 잘 설계된 부분

### Continuation Index가 얇다

루트 `CONTINUATION_INDEX.md`는 세션 상세를 중복하지 않고 프로젝트/토픽 수준의 active/paused 후보만 담는다. 이어하기 프로토콜도 2단계다.

1. 루트 인덱스만 읽고 프로젝트와 토픽을 고른다.
2. 선택된 토픽의 `TOPIC.md`와 필요한 하네스 문서만 읽는다.
3. 기존 세션을 선택한 경우에만 해당 session file을 읽는다.

이 구조는 이 책의 관점에서 매우 좋은 설계다. 메인 대화에 모든 과거 세션을 올리지 않고, 필요한 토픽만 지연 로드하기 때문이다.

줄이는 값:

- `uncached_input_tokens`
- `request_count`
- 탐색 단계의 `files_read`

### 실제 대상 repo와 기록 repo를 분리한다

루트 `AGENTS.md`는 실제 제품 코드와 `work_vault` 기록을 분리한다. 브랜치 생성, 자동 커밋, 기본 브랜치 머지는 `work_vault` 기록에만 적용하고, 실제 대상 repo는 사용자가 명시적으로 요청하지 않으면 자동 커밋/머지하지 않도록 되어 있다.

이 규칙은 토큰 절약과 직접 연결된다. 에이전트가 "지금 내가 수정해야 하는 repo가 무엇인가"를 매번 재추론하지 않게 하고, 변경 권한의 경계를 분명히 한다.

줄이는 값:

- 확인 질문으로 인한 `request_count`
- 잘못된 repo를 탐색하는 `uncached_input_tokens`
- 잘못된 변경 후 복구 비용

### 세션 기록의 필수 항목이 좋다

세션 기록에는 날짜, 프로젝트와 토픽, 실제 대상 경로, 작업 목표, 읽은 주요 파일, 변경 파일, 실행한 검증, 토큰 사용량과 비용 추정, 남은 위험이 남는다. 최신 세션에는 `Work Metrics`도 추가되어 있었다.

```text
request_count
cache_read_tokens
cache_create_tokens
uncached_input_tokens
tool_output_bytes
retry_count
읽은 파일 수
변경 파일 수
테스트 실행 수
가장 큰 도구 출력
```

이 필드는 이 책에서 제안한 usage worksheet와 거의 같은 방향이다.

## 3. 토큰 비용 관점의 위험

### 루트 AGENTS.md가 커지고 있다

루트 `AGENTS.md`는 약 10.6KB다. 내용은 유용하지만, 모든 작업의 stable prefix로 매번 들어간다면 비용이 될 수 있다. 캐시가 잘 작동하면 괜찮지만, 동적 입력이 앞쪽에 섞이거나 cache breakpoint가 흔들리면 `cache_create`가 반복될 수 있다.

개선안:

- `AGENTS.md` 상단에 30~50줄짜리 Quick Start를 둔다.
- 상세 규칙은 아래쪽 또는 별도 문서로 분리한다.
- 이어하기 시 필요한 규칙만 선택적으로 로드하도록 "규칙 라우팅" 섹션을 둔다.

예시:

```markdown
## Quick Start

1. Always read `CONTINUATION_INDEX.md` first.
2. Load only the selected project/topic.
3. Do not read session files unless continuing a specific session.
4. Record Usage And Cost; use `unknown` if metadata is unavailable.
5. Do not auto-commit target repos unless explicitly requested.
```

기대 효과:

- 첫 request의 이해 비용은 유지한다.
- 반복 request의 stable context 크기를 낮춘다.
- `cache_create_pressure`가 줄어든다.

### Usage And Cost가 많지만 실제 값은 대부분 unknown이다

35개 세션 중 `Usage And Cost` 관련 기록은 널리 들어가 있지만, 실제 `input_tokens`, `output_tokens`, `total_tokens`, 비용은 대부분 `unknown`이다. 이것은 정직한 기록이라는 점에서 좋다. 임의 계산하지 않는 원칙은 안전하다.

다만 이 상태에서는 지표 개선이 어렵다.

현재 할 수 있는 분석:

- 읽은 파일 수
- 변경 파일 수
- 테스트 실행 수
- retry count
- 가장 큰 도구 출력 이름
- 세션 파일 크기

아직 어려운 분석:

- `cache_read_ratio`
- `cache_create_pressure`
- `completion_share`
- request당 비용
- 작업 유형별 token baseline

개선안:

- 사용량 메타데이터를 확인할 수 없는 세션에도 proxy metric을 필수화한다.
- 최소한 `request_count`, `files_read`, `files_changed`, `test_runs`, `retry_count`, `largest_tool_output`은 채운다.
- 실제 token value가 없을 때는 `Token Usage: unavailable`, `Operational Metrics: available`처럼 층을 나눈다.

### 큰 세션 파일이 다음 세션의 입력을 키울 수 있다

가장 큰 세션 파일은 약 21.8KB였고, 10KB가 넘는 세션도 여러 개 있다. 세션 기록이 자세한 것은 장점이지만, 이어하기 때 전체 세션을 다시 읽으면 비용이 커진다.

개선안:

- 각 세션 상단에 `Continuation Summary`를 10~15줄로 둔다.
- 긴 구현 상세는 아래로 내린다.
- `TOPIC.md`에는 최신 이어하기 요약만 유지하고, 전체 세션은 필요할 때만 읽는다.

예시:

```markdown
## Continuation Summary

- Current status:
- Last target branch:
- Files most likely needed next:
- Last verification:
- Open risks:
- Do not reread unless needed:
```

기대 효과:

- 기존 세션 이어가기의 `uncached_input_tokens` 감소
- `files_read`와 `request_count` 감소

## 4. 성숙도 평가

이 책의 성숙도 모델로 보면 `work_vault`는 이미 Level 3~4 사이에 있다.

| 성숙도 항목 | 평가 |
| --- | --- |
| 출력 폭발 통제 | 일부 세션에서 capped output 언급이 있고, 검증 로그를 요약한다 |
| repo memory | 루트/프로젝트/토픽 `AGENTS.md` 상속 구조가 있다 |
| 작업 브리프 | 상황/목표/제약/진행 방식 형태가 루트 계약에 있다 |
| usage worksheet | `Usage And Cost`는 널리 있으나 실제 token value는 대부분 unknown |
| 리뷰 루프 | 세션마다 검증/위험은 남지만, 주기적 token review는 별도 구조가 약하다 |
| context architecture | continuation index와 topic lazy loading이 강하다 |

판정:

```text
Current level: Level 3.5
Reason: 작업 브리프, repo memory, continuation architecture는 강하지만,
        실제 token metric 수집과 주기적 리뷰 루프는 아직 약하다.
```

## 5. 가장 좋은 설계 결정

가장 좋은 설계는 루트 인덱스가 세션 상세를 중복하지 않는다는 점이다.

많은 작업 vault는 "이어하기 쉽게" 만들려다가 모든 세션 요약을 루트에 계속 붙인다. 그러면 처음에는 편하지만, 시간이 지날수록 루트 문서가 커지고 매번 읽어야 할 컨텍스트가 된다. `work_vault`는 이 위험을 피하고 있다.

`CONTINUATION_INDEX.md`는 얇은 입구이고, 세부 맥락은 선택된 토픽 내부에서만 로드한다. 이것은 agentic search와 just-in-time retrieval에 가까운 구조다.

## 6. 우선순위 높은 개선안

### 1순위: 세션 상단 Continuation Summary

긴 세션 파일마다 상단에 짧은 이어하기 요약을 둔다.

효과:

- 이어하기 request의 입력 크기 감소
- 이전 작업 재탐색 감소
- 불필요한 세션 전체 읽기 감소

### 2순위: proxy metric 필수화

실제 token metadata를 모를 때도 다음은 기록한다.

```text
request_count
files_read
files_changed
test_runs
retry_count
largest_tool_output
largest_tool_output_used: yes/no
```

효과:

- token value가 없어도 비싼 작업을 비교할 수 있다.
- `unknown`이 기록의 끝이 아니라 운영 분석의 시작점이 된다.

### 3순위: 루트 AGENTS.md Quick Start 분리

루트 규칙은 유지하되, 상단 quick start를 stable prefix로 쓰기 쉽게 만든다.

효과:

- 첫 로드에서 핵심 행동을 빠르게 고정한다.
- 상세 규칙은 필요할 때만 읽을 수 있다.

### 4순위: 10세션 단위 Token Review

개인 개발자라면 주간 리뷰가 부담스러울 수 있다. 대신 10세션마다 한 번 다음을 리뷰한다.

```markdown
## 10-Session Token Review

- 가장 큰 세션 파일:
- 가장 많은 파일을 읽은 세션:
- retry가 가장 많았던 세션:
- verification이 실패한 세션:
- 다음 AGENTS.md 변경:
- 다음 브리프 템플릿 변경:
```

효과:

- 세션 기록이 쌓이는 것에서 끝나지 않고 개인 운영 정책으로 환류된다.

## 7. 이 책에 주는 교훈

`work_vault` 사례는 이 책의 주장을 잘 보여준다.

- 토큰 절약은 짧은 프롬프트가 아니라 구조화된 기억에서 나온다.
- 이어하기 비용은 세션 기록의 양보다 로드 순서가 결정한다.
- `unknown`을 정직하게 남기는 것은 좋지만, proxy metric이 없으면 개선이 막힌다.
- 개인 개발자도 충분히 agent memory architecture를 만들 수 있다.
- 가장 중요한 문서는 "모든 정보가 있는 문서"가 아니라 "다음에 무엇을 읽을지 알려주는 문서"다.

## 출처

- 로컬 사례: `/Users/cayde/Desktop/workspace/public/work_vault/README.md`
- 로컬 사례: `/Users/cayde/Desktop/workspace/public/work_vault/AGENTS.md`
- 로컬 사례: `/Users/cayde/Desktop/workspace/public/work_vault/CONTINUATION_INDEX.md`
- 로컬 사례: `/Users/cayde/Desktop/workspace/public/work_vault/projects/`
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Austin1serb/agents-md](https://github.com/Austin1serb/agents-md)
