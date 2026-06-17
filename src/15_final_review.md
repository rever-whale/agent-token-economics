# 15. 최종 리뷰와 7일 실행 계획

이 책의 결론은 "토큰을 덜 쓰자"가 아니다. 결론은 더 정확하다. 개인 개발자는 에이전트가 읽어야 할 것, 읽지 말아야 할 것, 오래 기억해야 할 것, 이번 작업 뒤 버려야 할 것을 구분해야 한다. 그 구분이 생기면 request, cache create, uncached input, complete가 자연스럽게 안정된다.

## 책 전체를 관통하는 다섯 문장

1. 토큰 제한은 프롬프트 길이 문제가 아니라 작업 운영 문제다.
2. 캐시는 정적인 컨텍스트를 앞에 두고 동적인 컨텍스트를 뒤에 둘 때 살아난다.
3. 도구 출력은 무료가 아니다. 모델이 읽는 순간 입력 토큰이 된다.
4. 좋은 작업 브리프는 에이전트가 물어볼 질문을 줄인다.
5. 비싼 작업 리뷰는 반성이 아니라 정책 변경으로 끝나야 한다.

이 다섯 문장만 개인 작업 규칙으로 남겨도 많은 낭비가 줄어든다.

## 최종 원고 리뷰 체크

책을 개인 작업 매뉴얼이나 repo 운영 문서로 옮기기 전에 다음을 확인한다.

- 지표 용어가 하나의 schema로 정리되어 있는가?
- `complete`와 `output_tokens`처럼 도구별 표현 차이를 설명했는가?
- 좋은 비율과 나쁜 비율이 절대 규칙이 아니라 baseline 시작점으로 설명되는가?
- output cap 규칙이 line cap이 아니라 byte cap 중심으로 되어 있는가?
- AGENTS.md, 작업 브리프, usage worksheet, token review 템플릿이 모두 있는가?
- 사례 연구가 request/cache create/uncached input/complete의 연결을 보여주는가?
- 성숙도 모델이 다음 행동으로 이어지는가?
- 모든 장이 출처와 연결되는가?

이 체크는 문장 교정이 아니라 운영 가능성 점검이다. 읽고 고개를 끄덕이는 책보다, 다음 월요일에 정책 하나를 바꾸게 만드는 책이 더 낫다.

## 7일 실행 계획

전체 작업 방식이 한 번에 바뀌는 것이 부담스럽다면 한 repo에서 7일만 시도한다.

| 날짜 | 할 일 | 산출물 |
| --- | --- | --- |
| 1일차 | 최근 에이전트 작업 5개를 되짚는다 | 비싼 작업 후보 목록 |
| 2일차 | `AGENTS.md` 첫 버전을 만든다 | package manager, 명령, output policy |
| 3일차 | bugfix 브리프 템플릿을 적용한다 | expected/actual, evidence, verification |
| 4일차 | unknown output과 test log cap을 적용한다 | command output policy |
| 5일차 | usage worksheet를 3개 작업에 작성한다 | request/cache/output 기록 |
| 6일차 | 가장 비싼 작업 1개를 리뷰한다 | 원인과 정책 변경 |
| 7일차 | 규칙을 줄이고 고정한다 | AGENTS.md와 브리프 템플릿 개정 |

이 7일 계획의 성공 기준은 토큰 총량이 즉시 줄었는지가 아니다. 성공 기준은 내가 "왜 비쌌는지"를 한 문장으로 설명하고, 다음 작업 전에 바꿀 정책을 하나 찾았는지다.

## 운영 시작 전 최소 세트

바로 시작하려면 다음 네 개만 있으면 된다.

- `AGENTS.md`: repo 규칙, 명령, output policy
- 작업 브리프: bugfix/feature/review 중 하나
- usage worksheet: request, cache read, cache create, uncached input, output 기록
- 주간 리뷰: 가장 비싼 작업 3개와 정책 변경

이 최소 세트는 가볍지만 강하다. 에이전트가 매번 다시 배우는 비용을 줄이고, 사용자가 매번 다시 설명하는 피로를 줄인다.

## 마지막 판단 기준

토큰 운영이 좋아졌는지 보려면 다음 질문을 던진다.

- 같은 종류의 작업에서 request가 줄었는가?
- 첫 request 이후 cache read가 늘었는가?
- cache create가 매번 반복되지 않는가?
- 테스트 실패 로그가 전체 입력을 지배하지 않는가?
- complete가 실제 산출물에 쓰이는가?
- 비싼 작업 리뷰가 다음 정책 변경으로 이어지는가?

이 질문에 답할 수 있다면 이미 중요한 전환을 이룬 것이다. 더 이상 "에이전트가 이번에는 왜 많이 썼지?"라고 막연히 묻지 않는다. 대신 "이번 작업은 브리프가 약했고, 로그가 커졌고, cacheable prefix가 깨졌다"처럼 고칠 수 있는 언어로 말하게 된다.

그 언어가 생기는 순간, 토큰은 불편한 제한이 아니라 작업 시스템을 개선하는 계측값이 된다.

## 출처

- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [Austin1serb/agents-md](https://github.com/Austin1serb/agents-md)
