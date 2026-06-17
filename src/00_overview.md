# 학습 개요

> Phase: COMPLETE
> Progress: 100%

이 책은 코딩 에이전트와 일할 때 토큰을 아끼는 작은 요령 모음이 아니다. 목표는 개인 개발자가 제한된 토큰 예산 안에서 더 많은 개발 작업을 안정적으로 끝내도록, 컨텍스트를 설계하고 측정하고 개선하는 운영법을 만드는 것이다.

AI 시대의 개발은 "모델이 얼마나 똑똑한가"만으로 결정되지 않는다. 현업에서는 요청 횟수, 입력 토큰, 캐시 읽기, 캐시 쓰기, 출력 토큰, 도구 호출 결과, 로그 크기, 반복 수정 횟수 같은 더 지루한 숫자가 실제 생산성을 좌우한다. 특히 코딩 에이전트는 사람보다 훨씬 쉽게 불필요한 파일 전체, 거대한 테스트 로그, 중복된 대화 이력, 오래된 요구사항을 한 번에 끌어안는다. 토큰이 남아 있을 때는 그 낭비가 보이지 않지만, 제한이 걸리면 작업 품질과 속도가 동시에 흔들린다.

핵심 주장은 단순하다.

1. 토큰 절약은 짧게 말하기가 아니라 정확히 넣고 빨리 버리는 기술이다.
2. 좋은 컨텍스트는 크기가 작은 컨텍스트가 아니라 재사용 가능하고 위치가 안정적인 컨텍스트다.
3. 코딩 에이전트의 비용은 모델 호출 한 번보다 탐색, 검증, 재시도 루프 전체로 봐야 한다.
4. 반복되는 작업 지식은 대화창에 남기지 말고 파일, 지시서, 체크리스트, trace로 분리해야 한다.

## 대상 독자

이 책은 Codex, Claude Code, Cursor, Windsurf, GitHub Copilot 계열 에이전트를 실무 개발에 쓰는 개인 개발자에게 맞춘다. API 가격표를 외우는 책이 아니라, 제한된 request와 token budget 안에서 자신의 작업을 설계하는 책이다.

개별 Local LLM을 통한 저비용 에이전트 구축은 중요한 미래 방향이지만, 이 책의 기본 전제는 "현재 현업에서는 여전히 상용 모델과 제한된 토큰 예산을 쓴다"이다. 그래서 GPU 운영, 모델 학습, 자체 추론 서버보다 컨텍스트 패킷, 캐시 효율, 로그 관리, repo memory, 작업 분할, 개인 usage log를 우선한다.

## 이 책에서 쓰는 네 가지 지표

현장에서 도구마다 이름은 다르지만, 이 책은 다음 네 가지 축으로 비용을 읽는다.

- `request`: 모델 호출 횟수다. 너무 많으면 작업 분할이 잘못되었거나 에이전트가 방향을 잃었을 가능성이 높다.
- `cache read`: 이미 캐시된 입력 prefix를 재사용한 토큰이다. 높을수록 정적인 컨텍스트를 잘 재사용하고 있다는 신호다.
- `cache create`: 새로 캐시에 쓴 입력 토큰이다. 작업 초반에는 필요하지만, 매 요청마다 계속 높으면 캐시가 깨지고 있다는 신호다.
- `complete`: 모델이 생성한 출력 토큰이다. 코드 diff, 설명, 추론, 오류 메시지 재서술이 여기에 들어간다.

추가로 `input_tokens`, `output_tokens`, `tool output bytes`, `wall-clock time`, `retry count`, `files read`, `tests run`을 함께 본다. 토큰은 단독 지표가 아니라 작업 흐름의 그림자다. 숫자가 커졌을 때 "모델이 많이 생각했다"로 해석하면 안 된다. 보통은 읽을 필요 없는 것을 읽었거나, 설명할 필요 없는 것을 설명했거나, 검증 실패로 같은 일을 다시 한 것이다.

## 읽는 순서

Part 1은 지표의 언어를 만든다. 특히 [[03_usage_metrics|Request, Cache Read, Cache Create, Complete 읽는 법]]과 [[04_metric_ratios|좋은 비율과 나쁜 비율]]은 이 책의 기준점이다.

Part 2는 토큰 소모량이 큰 업무 습관을 다룬다. 큰 파일 읽기, 로그 폭발, 지시 흔들림, 탐색 결과를 메인 대화에 쌓는 방식, 캐시를 깨는 timestamp와 동적 지시를 따로 본다.

Part 3은 개인 운영법이다. [[09_agents_md_and_project_memory|AGENTS.md와 프로젝트 메모리 설계]], [[10_context_packets|컨텍스트 패킷과 작업 브리프]], [[11_measurement_dashboard|측정 대시보드와 리뷰 루프]]를 통해 혼자서도 반복 가능한 운영 체계로 바꾼다.

Part 4는 사례 연구다. [[13_case_studies|세 가지 작업으로 보는 토큰 운영]]에서 bugfix, feature, review 작업을 놓고 어떤 지표가 왜 커지는지, 어떤 정책이 request/cache create/complete를 줄이는지 연결해서 본다.

Part 5는 통합 리뷰다. [[14_maturity_model|토큰 운영 성숙도 모델]]을 통해 개인 개발자가 현재 어느 단계에 있고, 다음 단계로 가려면 어떤 문서, 지표, 리뷰 루프가 필요한지 판단한다. 마지막으로 [[15_final_review|최종 리뷰와 7일 실행 계획]]에서 책의 내용을 실제 도입 순서로 압축한다.

## 산출물

- mdBook 형식의 원고
- 코딩 에이전트 토큰 사용량 해석 프레임
- request/cache read/cache create/complete 기준 비율
- AGENTS.md와 컨텍스트 패킷 설계 가이드
- 개인 작업 대시보드와 작업 리뷰 체크리스트
- bugfix/feature/review 사례 연구
- 토큰 운영 성숙도 모델과 통합 리뷰 기준
- 7일 실행 계획과 최종 리뷰 기준

## 출처

- [Meirtz/Awesome-Context-Engineering](https://github.com/Meirtz/Awesome-Context-Engineering)
- [Austin1serb/agents-md](https://github.com/Austin1serb/agents-md)
- [microsoft/fastcontext](https://github.com/microsoft/fastcontext)
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
