# C. 용어집

## request

모델 호출 1회를 뜻한다. 코딩 에이전트에서는 사용자 메시지뿐 아니라 도구 결과를 읽고 다음 행동을 결정하는 내부 호출도 비용에 포함될 수 있다.

## input tokens

모델에 들어간 입력 토큰이다. 캐시를 쓰는 API에서는 uncached input만 별도 `input_tokens`로 표시될 수 있으므로, 공급자별 정의를 확인해야 한다.

## cache read

이미 캐시된 prompt prefix를 재사용한 토큰이다. 보통 새로 처리하는 입력보다 비용과 latency가 낮다.

## cache create

새로운 cache entry를 만들기 위해 처리한 토큰이다. 초기 투자로는 정상이나, 매 request마다 높으면 cache invalidation을 의심한다.

## complete

모델이 생성한 출력 토큰이다. `completion_tokens`, `output_tokens` 등으로 표시되기도 한다.

이 책에서는 `complete`와 `output_tokens`를 같은 층의 말로 다룬다. 도구 UI가 `complete`라고 부르면 그대로 쓰고, CSV나 telemetry schema에서는 `output_tokens`로 정규화한다.

## uncached input

캐시 재사용 없이 새로 처리되는 입력 토큰이다. 도구 출력, 최신 사용자 메시지, 현재 작업 브리프, 최근 로그가 여기에 들어가기 쉽다.

## stable context

여러 request에서 거의 바뀌지 않는 컨텍스트다. repo 규칙, coding convention, 테스트 명령, 반복 rubric처럼 cache read로 재사용되기 좋은 정보다.

## dynamic context

현재 작업마다 바뀌는 컨텍스트다. 사용자 최신 요구, 테스트 로그, tool output, git diff, timestamp가 여기에 속한다. cacheable prefix 앞쪽에 두면 cache create가 반복될 수 있다.

## high-signal context

결정에 직접 영향을 주는 작은 컨텍스트다. 전체 로그보다 첫 actionable failure, 전체 파일보다 관련 함수 주변부, 회의록 전체보다 결정된 scope가 high-signal context에 가깝다.

## prefix

프롬프트의 앞부분이다. prompt caching은 대개 동일한 prefix 재사용에 의존하므로, 정적인 정보를 앞쪽에 배치하는 것이 중요하다.

## breakpoint

캐시할 prefix의 끝 지점이다. breakpoint 앞쪽에 바뀌는 값이 들어가면 cache hit가 어려워진다.

## context packet

이번 작업에 필요한 목표, 범위, evidence, 제약, 검증 조건을 묶은 입력 단위다.

## AGENTS.md

코딩 에이전트에게 repo별 작업 규칙을 알려주는 markdown 지시 파일 패턴이다. 도구마다 파일명과 지원 방식은 다를 수 있다.

## tool output

셸 명령, 파일 읽기, 검색, 테스트 실행, 브라우저 검사 등 도구가 반환한 출력이다. 모델이 읽는 순간 입력 토큰 비용이 된다.

## retry count

실패 후 재시도한 횟수다. 토큰 비용뿐 아니라 요구사항 명확성, 검증 품질, 탐색 효율을 보여주는 신호다.

## 출처

- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Austin1serb/agents-md](https://github.com/Austin1serb/agents-md)
