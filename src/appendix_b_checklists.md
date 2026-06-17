# B. 체크리스트

## 작업 시작 전

- [ ] 목표가 한 문장으로 쓰였는가?
- [ ] 수정 범위와 제외 범위가 있는가?
- [ ] 관련 파일 후보가 있거나 찾는 방법이 있는가?
- [ ] 검증 명령이 명시되어 있는가?
- [ ] 완료 조건이 관찰 가능한가?
- [ ] repo 공통 규칙은 대화가 아니라 파일에 있는가?

## 도구 출력

- [ ] 처음 보는 출력에 byte cap을 걸었는가?
- [ ] 테스트 실패는 tail부터 봤는가?
- [ ] 큰 파일은 `rg`로 위치를 좁힌 뒤 읽었는가?
- [ ] generated/build artifact/lockfile 전체를 불필요하게 읽지 않았는가?
- [ ] 가장 큰 출력이 실제 결정에 쓰였는가?

## 캐시 효율

- [ ] 정적 지시가 앞쪽에 있는가?
- [ ] 작업별 동적 입력은 뒤쪽에 있는가?
- [ ] timestamp, 로그, request id가 cacheable prefix 앞에 있지 않은가?
- [ ] 첫 request 이후 cache create가 줄어드는가?
- [ ] 반복 작업에서 cache read ratio가 올라가는가?

## AGENTS.md

- [ ] package manager와 주요 명령이 있다.
- [ ] 주요 경로와 수정 금지 영역이 있다.
- [ ] command output policy가 있다.
- [ ] verification policy가 있다.
- [ ] final response policy가 있다.
- [ ] 2분 안에 읽을 수 있다.
- [ ] 오래된 규칙이 제거된다.

## 사후 리뷰

- [ ] request가 예상보다 많았는가?
- [ ] retry가 어디서 발생했는가?
- [ ] cache read가 낮은 이유를 설명할 수 있는가?
- [ ] cache create가 반복된 이유를 설명할 수 있는가?
- [ ] complete가 산출물보다 설명에 쓰이지 않았는가?
- [ ] 다음 작업에서 줄일 수 있는 정책 하나가 생겼는가?

## 출처

- [Austin1serb/agents-md](https://github.com/Austin1serb/agents-md)
- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
