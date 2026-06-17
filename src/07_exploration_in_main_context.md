# 7. 메인 대화에 탐색을 전부 넣는 습관

에이전트가 repo를 탐색할 때 모든 파일 읽기와 검색 결과를 메인 대화에 쌓으면, 이후 판단은 점점 무거워진다. 오래된 후보, 버려진 가정, 관련 없는 파일이 계속 따라다니기 때문이다.

## 탐색과 실행은 다른 종류의 컨텍스트다

탐색 단계의 정보는 대부분 일회용이다. "이 파일은 후보가 아니다", "이 경로에는 관련 코드가 없다", "검색 결과 30개 중 2개만 중요하다" 같은 정보는 결론만 남기면 된다.

실행 단계의 정보는 보존 가치가 높다.

- 수정할 파일
- 기존 패턴
- 테스트 명령
- 실패 원인
- 변경한 diff
- 남은 리스크

메인 대화에는 실행 가치가 있는 요약만 남기고, 탐색 noise는 도구나 별도 scratchpad에서 줄이는 편이 좋다.

## FastContext가 주는 힌트

Microsoft의 FastContext는 coding agent를 위한 efficient repository explorer를 훈련하는 방향을 제시한다. 핵심 아이디어는 repo 탐색을 더 잘하는 별도 explorer가 최종 응답에 필요한 근거를 찾아주는 것이다. 이 책에서 FastContext를 그대로 구현하지는 않지만, 운영 원칙은 가져올 수 있다.

- 탐색은 별도 단계로 분리한다.
- 최종 에이전트에게는 citation과 관련 파일만 넘긴다.
- repo 전체를 읽기보다 필요한 evidence를 찾는다.
- 탐색 trace 전체가 아니라 최종 근거를 보존한다.

상용 코딩 에이전트를 쓰는 개인 개발자도 이 원칙을 적용할 수 있다. 예를 들어 큰 작업을 시작할 때 "먼저 수정 후보 파일과 근거만 찾아서 보고하고, 아직 patch하지 말라"고 시킬 수 있다. 탐색 결과가 맞으면 그때 실행 단계로 넘긴다.

## 메인 컨텍스트를 가볍게 유지하는 패턴

권장 패턴은 다음과 같다.

1. 검색 단계에서는 후보 파일과 근거만 모은다.
2. 각 후보에 대해 관련성 판단을 한 줄로 남긴다.
3. 탈락 후보의 긴 출력은 보존하지 않는다.
4. 실행 단계에서는 선택한 파일 주변부만 다시 읽는다.
5. 최종 보고에는 실제 결정에 쓰인 파일만 남긴다.

이 방식은 request 수를 약간 늘릴 수 있지만, 큰 작업에서는 전체 토큰과 실패율을 줄인다. 한 번에 모든 탐색과 수정을 섞으면 잘못된 파일까지 컨텍스트에 들어와 판단을 흐린다.

## 탐색 요약 템플릿

탐색이 끝났을 때 다음 정도만 남기면 충분한 경우가 많다.

```markdown
## 탐색 요약

- 목표: 로그인 실패 시 error banner가 사라지는 문제
- 관련 파일:
  - `src/auth/LoginForm.tsx`: submit handler와 error state
  - `src/auth/useLogin.ts`: mutation error mapping
- 제외한 파일:
  - `src/components/Banner.tsx`: 표시 컴포넌트이며 상태 reset과 무관
- 검증:
  - `pnpm test LoginForm`
```

이 요약은 다음 단계의 입력으로 가치가 높고, 검색 raw output보다 훨씬 작다.

## 출처

- [microsoft/fastcontext](https://github.com/microsoft/fastcontext)
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
