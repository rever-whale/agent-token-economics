# 10. 컨텍스트 패킷과 작업 브리프

컨텍스트 패킷은 에이전트에게 넘기는 작업 단위 입력이다. 목적은 모델에게 많은 것을 알려주는 것이 아니라, 필요한 결정을 빠르게 내릴 수 있는 최소 충분 정보를 주는 것이다.

## 좋은 작업 브리프의 구성

```markdown
## Goal
무엇을 끝내야 하는가.

## Scope
수정해도 되는 영역과 제외할 영역.

## Evidence
버그 재현, 로그 핵심, 관련 파일, 스크린샷, 사용자 보고.

## Constraints
호환성, 성능, 보안, API 변경 금지, 디자인 제약.

## Verification
실행해야 할 테스트와 완료 조건.

## Reporting
최종 답변에 포함할 내용.
```

이 템플릿은 request를 줄인다. 에이전트가 "어디까지 해도 되는가", "무엇으로 성공을 판단하는가"를 묻느라 왕복하지 않아도 되기 때문이다.

## 너무 많은 컨텍스트의 역효과

사용자는 종종 불안해서 관련 있어 보이는 모든 정보를 붙인다. 하지만 컨텍스트가 많을수록 모델이 더 정확해지는 것은 아니다. 관련 없는 정보가 많으면 attention이 분산되고, 오래된 요구사항과 최신 요구사항이 충돌할 수 있다.

컨텍스트 패킷에는 다음을 넣지 않는다.

- 오래된 실패 로그 전체
- 이미 해결한 가설
- unrelated stack trace
- 변경 금지 파일의 전체 내용
- 회의록 전체
- "참고가 될 수도 있는" 긴 문서

대신 결론과 링크를 넣는다.

```markdown
- 이전 가설: API timeout 문제로 봤으나 서버 로그상 200 응답 확인. 제외.
- 관련 로그: `TypeError: Cannot read property 'id'`가 submit 직후 발생.
- 전체 로그는 필요 시 `logs/auth-2026-06-17.txt`에서 확인.
```

## 패킷은 캐시 뒤에 붙인다

컨텍스트 패킷은 작업마다 바뀌므로 캐시 가능한 repo 지시 뒤에 붙인다. 이렇게 하면 프로젝트 규칙은 cache read로 재사용하고, 작업별 정보만 새로 처리한다.

권장 순서:

1. 전역/프로젝트 지시
2. 작업 브리프
3. 관련 evidence
4. 최신 tool output

## 패킷 리뷰 질문

작업을 시작하기 전 다음 질문을 던진다.

- 이 정보가 없으면 에이전트가 잘못된 결정을 할까?
- 이 정보는 최신인가?
- raw output 대신 요약할 수 있는가?
- 같은 정보가 repo memory에 이미 있는가?
- 완료 조건이 검증 가능한가?

이 질문을 통과하지 못하는 컨텍스트는 보통 토큰을 쓰지만 판단을 돕지 않는다.

## 작업 유형별 브리프 차이

모든 작업 브리프가 같은 모양일 필요는 없다. 작업 유형마다 토큰을 줄이는 핵심 정보가 다르다.

| 작업 유형 | 꼭 필요한 정보 | 빼도 되는 정보 |
| --- | --- | --- |
| bugfix | expected/actual, 재현 단계, 실패 로그 핵심, 의심 파일 | 전체 로그, 오래된 추측 |
| feature | 사용자 흐름, 범위, 제외 범위, 검증 시나리오 | 구현 세부 지시의 과도한 나열 |
| refactor | 변경 목표, public API 유지 여부, 테스트 범위 | “깔끔하게” 같은 추상 표현 |
| review | diff 범위, 리뷰 기준, 위험 우선순위 | 전체 프로젝트 배경 |
| docs | 독자, 문서 위치, 포함/제외할 주장 | 관련 자료 전체 원문 |

작업 브리프는 에이전트가 묻지 않아도 될 질문을 미리 없애는 장치다. 혼자 일할 때도 브리프를 쓰면 다음 세션의 내가 같은 맥락을 다시 설명하지 않아도 된다. 반대로 브리프가 너무 길어져 에이전트가 핵심을 찾게 만들면 실패다.

복사용 bugfix/feature/review 브리프는 [[appendix_e_operating_templates|E. 운영 템플릿]]에 있다.

## 출처

- [Meirtz/Awesome-Context-Engineering](https://github.com/Meirtz/Awesome-Context-Engineering)
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
