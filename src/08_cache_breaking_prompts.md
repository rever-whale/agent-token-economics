# 8. 캐시를 깨는 프롬프트 구조

캐시는 같은 prefix를 재사용할 때 효과가 있다. 따라서 캐시 효율은 프롬프트 내용만이 아니라 순서와 안정성에 달려 있다. 좋은 내용도 매 요청마다 앞부분이 조금씩 바뀌면 cache read가 낮아지고 cache create가 반복된다.

## 가장 흔한 실수

가장 흔한 실수는 정적 지시와 동적 입력을 섞는 것이다.

나쁜 구조:

```text
오늘 날짜: 2026-06-17
이번 작업: 버튼 버그 수정
프로젝트 규칙: ...
테스트 명령: ...
코딩 스타일: ...
```

이 구조에서는 날짜와 작업명이 앞쪽에 있어 prefix가 매번 달라진다. 프로젝트 규칙이 같아도 캐시가 잘 맞지 않는다.

좋은 구조:

```text
프로젝트 규칙: ...
테스트 명령: ...
코딩 스타일: ...

이번 작업: 버튼 버그 수정
오늘 날짜: 2026-06-17
```

정적인 것을 앞에, 동적인 것을 뒤에 둔다.

## 캐시 친화적인 순서

코딩 에이전트의 입력은 다음 순서가 좋다.

1. 도구 정의와 시스템 지시
2. repo 공통 규칙
3. 반복 작업 템플릿
4. 안정적인 architecture summary
5. 이번 작업 브리프
6. 이번 작업에서 읽은 파일 주변부
7. 최신 로그와 테스트 결과
8. 사용자 추가 지시

앞쪽은 재사용되고, 뒤쪽은 자주 바뀐다. 이 구분을 지키면 cache read가 늘고 cache create는 초반 이후 줄어든다.

## 캐시를 깨는 요소

다음 요소는 가능하면 캐시 breakpoint 뒤에 둔다.

- timestamp와 현재 날짜
- request id, trace id, random seed
- 사용자 최신 메시지
- 테스트 로그
- 셸 명령 결과
- git diff
- 에이전트의 중간 계획
- volatile environment 정보

반대로 다음은 breakpoint 앞에 두기 좋다.

- coding convention
- package manager
- test command policy
- output policy
- security rule
- architecture map
- 반복되는 prompt template

## 자동 캐시의 함정

일부 API는 자동 caching을 제공한다. 편하지만 마지막 cacheable block이 매번 바뀌는 구조라면 효과가 낮을 수 있다. Anthropic 문서는 바뀌는 timestamp가 포함된 block에 breakpoint를 두면 매 request마다 다른 hash가 되어 cache hit가 나지 않는 예를 설명한다.

자동 기능은 출발점으로 좋지만, 개인 개발자도 큰 비용을 다루는 작업에서는 명시적 breakpoint와 지시 순서를 관리해야 한다. 특히 long-running agent session에서는 초반 정적 컨텍스트와 후반 동적 tool output의 경계를 분명히 해야 한다.

## 캐시 친화 프롬프트 골격

실무에서는 다음 골격을 기본값으로 두고, 작업별 정보만 뒤쪽에 붙이는 방식이 가장 단순하다.

```markdown
## Stable Context

### Role and Operating Rules
- You are working in this repository.
- Follow the repo conventions and verification policy.

### Repository Rules
- Package manager:
- Test commands:
- Generated files:
- Output policy:

### Reusable Task Rubric
- Before editing:
- During editing:
- Before final response:

## Dynamic Task Context

### Current Goal

### Scope

### Evidence

### Latest Tool Output

### User Additions
```

`Stable Context`는 자주 바뀌지 않아야 한다. 날짜, request id, 최신 로그, 사용자 추가 요구사항은 `Dynamic Task Context` 아래로 내린다. 이 골격이 지켜지면 첫 request 이후 `cache create`는 줄고, `cache read`가 늘어나는 패턴을 기대할 수 있다.

전체 복사용 템플릿은 [[appendix_e_operating_templates|E. 운영 템플릿]]에 모아두었다.

## 출처

- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
