# 9. AGENTS.md와 프로젝트 메모리 설계

`AGENTS.md`는 코딩 에이전트에게 repo별 작업 규칙을 알려주는 가벼운 인터페이스다. 이름이나 지원 방식은 도구마다 다르지만, 핵심은 같다. 프로젝트에서 반복되는 판단을 대화창이 아니라 version-controlled file에 둔다.

## AGENTS.md가 줄이는 비용

좋은 프로젝트 메모리는 다음 비용을 줄인다.

- package manager와 script 탐색 비용
- 잘못된 파일 수정 후 되돌리는 비용
- 테스트 명령을 찾는 비용
- 코드 스타일 정정 비용
- 최종 보고 형식 재설명 비용
- 위험한 명령 승인/거절 비용

AGENTS.md는 만능 지식베이스가 아니다. 너무 길면 매 request 입력을 무겁게 만들고, 오래된 규칙은 오히려 실패를 만든다. 목적은 "에이전트가 첫 5분 안에 잘못된 길로 가지 않게 하는 것"이다.

## 권장 구조

```markdown
# AGENTS.md

## Project Snapshot
- Runtime:
- Package manager:
- Main app paths:

## Common Commands
- Install:
- Typecheck:
- Unit test:
- Build:

## Editing Rules
- Do:
- Avoid:
- Generated files:

## Command Output Policy
- Unknown output:
- Test failures:
- Large files:

## Verification Policy
- Required checks:
- When checks cannot run:

## Final Response Policy
- Mention:
- Do not include:
```

이 정도 구조면 에이전트가 작업 시작 전에 필요한 결정을 빠르게 내릴 수 있다.

## 메모리는 짧고 구체적이어야 한다

나쁜 메모리:

```text
좋은 코드를 작성하라. 테스트를 잘 하라. 프로젝트 스타일을 지켜라.
```

좋은 메모리:

```text
Use `pnpm --filter web test` for web unit tests.
Do not edit files under `src/generated/`.
For unknown command output, cap with `head -c 4000` or `tail -c 12000`.
Final response must include changed files, checks run, and remaining risks.
```

추상적 원칙은 모델이 이미 안다. repo memory에는 모델이 모르는 구체 사실을 둔다.

## 유지보수 규칙

프로젝트 메모리는 코드와 함께 늙는다. 다음 규칙이 필요하다.

- command가 바뀌면 같은 PR에서 AGENTS.md도 수정한다.
- 3개월 이상 쓰이지 않은 규칙은 삭제 후보로 표시한다.
- 반복 정정이 2번 이상 나오면 메모리에 반영한다.
- 한 달 뒤의 내가 읽어도 2분 안에 이해되어야 한다.
- 장황한 배경 설명은 별도 문서로 빼고 링크만 둔다.

좋은 AGENTS.md는 토큰을 아낄 뿐 아니라 개인 작업의 암묵지를 드러낸다. 에이전트가 잘못하는 지점은 대개 한 달 뒤의 나도 다시 헷갈릴 지점이다.

## 첫 버전을 만드는 순서

처음부터 완벽한 AGENTS.md를 만들려고 하면 문서가 길어진다. 다음 순서로 만든다.

1. 최근 10개 에이전트 작업에서 사용자가 반복 정정한 문장을 모은다.
2. package manager, test command, build command처럼 사실 정보만 먼저 쓴다.
3. 출력 폭발을 막는 command output policy를 넣는다.
4. 수정 금지 영역과 generated file 위치를 넣는다.
5. 최종 답변 형식을 3~5개 항목으로 제한한다.
6. 한 달 뒤 실제로 도움이 되지 않은 규칙을 삭제한다.

중요한 것은 “에이전트에게 알려주고 싶은 모든 것”이 아니라 “모르면 비용이 커지는 것”을 넣는 것이다. 예를 들어 개발 철학보다 테스트 명령이 먼저다. naming preference보다 generated file 금지가 먼저다.

복사용 AGENTS.md 템플릿은 [[appendix_e_operating_templates|E. 운영 템플릿]]에 있다.

## 출처

- [Austin1serb/agents-md](https://github.com/Austin1serb/agents-md)
- [Context Engineering for AI Agents in Open-Source Software](https://arxiv.org/abs/2510.21413)
