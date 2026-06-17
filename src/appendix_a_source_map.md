# A. 소스 맵

이 부록은 참고자료가 책의 어떤 주장에 연결되는지 정리한다.

## Meirtz/Awesome-Context-Engineering

이 저장소는 context engineering을 단순 prompt 작성이 아니라 LLM에 제공되는 전체 정보 payload를 설계하는 문제로 본다. 특히 2026 agent era update는 context engineering의 중심이 "좋은 prompt를 pack하는 법"에서 agent runtime, memory, tools, protocols, approvals, long-horizon execution으로 이동했다고 설명한다.

이 책에서는 다음 장에 연결한다.

- [[01_token_budget_as_operations|1장]]: 토큰 제한을 운영 문제로 보는 관점
- [[06_instruction_drift|6장]]: 정적 prompt보다 runtime context와 memory를 관리해야 한다는 관점
- [[10_context_packets|10장]]: context packet 설계
- [[14_maturity_model|14장]]: context engineering을 개인 개발자의 성숙도와 운영 체계로 보는 관점

## Austin1serb/agents-md

이 저장소는 AGENTS.md와 Codex prompt pattern을 통해 context discipline, safer command output, lower token usage를 다룬다. 특히 byte-capped command output을 가장 큰 현재 win으로 제시하며, line limit보다 byte cap이 안전하다고 설명한다.

이 책에서는 다음 장에 연결한다.

- [[05_large_reads_and_logs|5장]]: 큰 로그와 파일 출력 제어
- [[09_agents_md_and_project_memory|9장]]: AGENTS.md 구조
- [[12_operating_playbook|12장]]: 개인 작업 정책으로 고정하는 방법
- [[13_case_studies|13장]]: byte cap, output policy, 작업 브리프가 지표에 미치는 영향
- [[15_final_review|15장]]: 7일 실행 계획과 최소 운영 세트

## microsoft/fastcontext

FastContext는 coding agent를 위한 efficient repository explorer를 훈련하는 연구/도구 방향이다. repo 탐색을 별도 explorer 문제로 보고, 최종 답변에 필요한 evidence를 찾는 구조를 강조한다.

이 책에서는 다음 장에 연결한다.

- [[07_exploration_in_main_context|7장]]: 탐색과 실행 컨텍스트 분리
- [[11_measurement_dashboard|11장]]: files read, files changed, citation quality 같은 탐색 효율 지표
- [[14_maturity_model|14장]]: 탐색 noise를 줄이는 성숙도 상위 단계

## Anthropic: Effective context engineering for AI agents

Anthropic의 이 글은 agent context를 finite resource로 보고, 원하는 결과를 만들 가능성을 가장 크게 높이는 "작지만 high-signal인 token set"을 유지해야 한다고 설명한다. 또한 agentic search와 just-in-time context retrieval, long-horizon task를 위한 compaction, structured note-taking, sub-agent architecture를 소개한다.

이 책에서는 다음 장에 연결한다.

- [[01_token_budget_as_operations|1장]]: 토큰 예산을 attention budget과 운영 자원으로 보는 관점
- [[07_exploration_in_main_context|7장]]: 전체 데이터를 미리 넣지 않고 file path, query, link 같은 lightweight reference로 런타임 탐색하는 방식
- [[10_context_packets|10장]]: high-signal context packet 설계
- [[12_operating_playbook|12장]]: compaction, note-taking, sub-agent 분리 같은 long-horizon 운영 전략
- [[13_case_studies|13장]]: high-signal context가 request, retry, complete를 줄이는 사례
- [[14_maturity_model|14장]]: stable/dynamic context와 long-horizon context architecture
- [[15_final_review|15장]]: 개인 개발자가 high-signal context 운영을 시작하는 최소 절차
- [[16_work_vault_case_analysis|16장]]: 실제 vault의 continuation index와 lazy loading 구조 분석

## Anthropic Claude Prompt Caching Docs

Anthropic 문서는 prompt caching의 동작, 가격 multiplier, cache read/create/input token breakdown, cache invalidation pattern을 설명한다. 특히 `cache_creation_input_tokens`, `cache_read_input_tokens`, `input_tokens`를 분리해서 total input token을 계산하는 방식이 이 책의 지표 체계에 직접 연결된다.

이 책에서는 다음 장에 연결한다.

- [[03_usage_metrics|3장]]: request/cache read/cache create/complete 해석
- [[04_metric_ratios|4장]]: cache read ratio와 cache create pressure
- [[08_cache_breaking_prompts|8장]]: 캐시를 깨는 prompt 구조
- [[13_case_studies|13장]]: cache create가 반복되는 나쁜 trace 해석
- [[15_final_review|15장]]: request/cache/output 기록을 도입하는 7일 계획

## Local: work_vault

`work_vault`는 개인 개발자가 여러 프로젝트의 AI 작업 맥락, 하네스, 세션 기록을 관리하는 실제 로컬 vault다. 이 책에서는 구조화된 agent memory, continuation index, usage tracking, session record의 장단점을 분석하는 사례로 사용한다.

이 책에서는 다음 장에 연결한다.

- [[16_work_vault_case_analysis|16장]]: 실제 개인 작업 vault를 토큰 운영 기준으로 분석한 사례

## 출처

- [Meirtz/Awesome-Context-Engineering](https://github.com/Meirtz/Awesome-Context-Engineering)
- [Austin1serb/agents-md](https://github.com/Austin1serb/agents-md)
- [microsoft/fastcontext](https://github.com/microsoft/fastcontext)
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- 로컬 사례: `/Users/cayde/Desktop/workspace/public/work_vault`
