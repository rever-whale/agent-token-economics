# 12. 실전 운영 플레이북

이 장은 앞의 내용을 개인 개발자의 실제 운영 절차로 묶는다.

## 1단계: 현재 비용을 보이게 한다

먼저 10~20개 작업만 기록한다. 자동화가 없어도 된다. 작업별 request, cache read, cache create, complete, retry count, 가장 큰 도구 출력만 적는다.

이 단계의 목표는 최적화가 아니라 패턴 발견이다. 대부분의 개인 개발자는 다음 중 하나를 발견한다.

- 특정 작업 유형에서 request가 비정상적으로 많다.
- 테스트 로그가 대부분의 토큰을 먹는다.
- cache read가 거의 없다.
- cache create가 매번 높다.
- 사용자가 같은 repo 규칙을 반복 설명한다.

## 2단계: 출력 폭발을 먼저 막는다

가장 빠른 개선은 command output policy다.

```markdown
## Command Output Policy

- Unknown command output must be capped with `head -c 4000`.
- Test failures should start with `tail -c 12000`.
- Do not read generated files, lockfiles, or build artifacts in full unless necessary.
- Prefer `rg` before opening large files.
```

이 정책은 구현 비용이 낮고 효과가 빠르다.

## 3단계: AGENTS.md를 만든다

다음으로 repo memory를 만든다. 첫 버전은 100~200줄을 넘기지 않는다.

- package manager
- 주요 경로
- 개발/테스트 명령
- 수정 금지 영역
- output policy
- verification policy
- final response policy

반복 정정이 생길 때마다 문서에 반영한다.

## 4단계: 작업 브리프 템플릿을 쓴다

모든 작업에 긴 specification이 필요한 것은 아니다. 하지만 bugfix와 feature는 최소 템플릿을 쓰는 편이 좋다.

```markdown
Goal:
Scope:
Evidence:
Constraints:
Verification:
```

이 템플릿은 request와 retry를 줄이는 데 직접적이다.

## 5단계: 캐시 친화적으로 재배치한다

프롬프트와 지시 순서를 점검한다.

- 정적 지시는 앞에 둔다.
- 동적 작업 정보는 뒤에 둔다.
- timestamp와 로그는 cache breakpoint 앞에 두지 않는다.
- 긴 예제와 평가 기준은 안정된 block으로 유지한다.

cache read ratio가 낮으면 먼저 순서를 의심한다.

## 6단계: 리뷰로 정책을 업데이트한다

주기적으로 비싼 작업을 3개 골라 사후 리뷰한다.

- 어떤 출력이 가장 컸는가?
- 그 출력은 실제로 필요했는가?
- 어떤 지시가 없어서 재시도했는가?
- 어떤 정보를 cacheable context로 올릴 수 있는가?
- 다음번에 request를 줄이는 가장 작은 정책은 무엇인가?

정책은 작은 문장으로 끝나야 한다. "더 조심하자"는 정책이 아니다. "unknown output은 `head -c 4000`"처럼 실행 가능한 문장이어야 한다.

## 최종 원칙

토큰을 아끼는 개인 개발자는 에이전트에게 적게 알려주는 사람이 아니다. 무엇을 오래 기억할지, 무엇을 이번 작업에만 쓸지, 무엇을 읽지 말아야 할지 잘 나누는 사람이다.

## 30일 도입 계획

작게 시작하려면 다음 순서를 쓴다.

| 기간 | 목표 | 산출물 |
| --- | --- | --- |
| 1주차 | 현재 비용을 관찰한다 | 10개 작업 usage worksheet |
| 2주차 | 출력 폭발을 줄인다 | command output policy가 들어간 AGENTS.md |
| 3주차 | 작업 입력을 표준화한다 | bugfix/feature/review 브리프 템플릿 |
| 4주차 | 리뷰 루프를 만든다 | 주간 token review와 정책 변경 1개 |

이 계획의 핵심은 자동화보다 습관이다. 자동 수집이 없어도, 가장 큰 tool output과 retry 원인을 기록하기 시작하면 낭비가 보인다. 낭비가 보이면 개인 규칙으로 바꿀 수 있다.

도입에 필요한 복사용 템플릿은 [[appendix_d_usage_worksheet|D. 지표 워크시트와 예시 Trace]]와 [[appendix_e_operating_templates|E. 운영 템플릿]]에 있다.

## 출처

- [Austin1serb/agents-md](https://github.com/Austin1serb/agents-md)
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [microsoft/fastcontext](https://github.com/microsoft/fastcontext)
