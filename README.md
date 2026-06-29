# 뉴스 요약 자동화 워크플로우 (7팀)

## 프로젝트 개요

RSS 데이터 수집 → AI 텍스트 요약 → 저장으로 이어지는 자동화 워크플로우를 구축합니다. 단일 기능 구현이 아니라 **스케줄 트리거 · 주제 필터링 · AI 요약 · 에러 처리 · 중복 방지**까지 파이프라인의 전체 흐름을 직접 설계하고 연결하는 것이 핵심입니다.

| 항목 | 내용 |
| --- | --- |
| 자동화 툴 | Make.com |
| AI 요약 | OpenAI (Generate a Completion) |
| 저장소 | Notion 데이터베이스 |
| 실행 주기 | 매일 지정 시간 · 타임존 Asia/Seoul |
| 팀 | 7팀 |

### 팀 구성과 역할

본 프로젝트는 7팀 4명이 함께 진행하며, 파이프라인 단계별로 역할을 다음과 같이 분담했습니다.

| 이름 | 역할 |
| --- | --- |
| 황수빈 (팀장) | 기획 및 조율 / OpenAI 모듈 및 연결 / 문제 해결 / README |
| 박세빈 | RSS 모듈 / Notion 모듈 / 문제 해결 |
| 이윤희 | OpenAI 모듈 |
| 이종명 | Notion DB 관리 / Notion 모듈 / Retry 모듈 / 문제 해결 / README |

## 미션 소개

새 프로젝트의 시장 조사를 위해 수십 개의 커뮤니티·블로그·뉴스 사이트를 일일이 뒤지고, 수백 개의 탭을 띄워 내용을 복사해 옮기다 보면 정작 중요한 '분석'을 시작하기도 전에 지치게 됩니다.

이 프로젝트는 **RSS 수집 → AI 요약 → 협업 도구 자동 저장**으로 이어지는 흐름을 코드 없이 직접 설계하여, 최신 기술 뉴스를 자동으로 수집·요약·정리하는 파이프라인을 완성합니다. 다양한 AI 툴과 자동화 서비스를 유기적으로 연결하는 경험을 통해, 이후 더 복잡한 업무 자동화 시스템을 설계·운영할 수 있는 기반을 다지는 것이 목표입니다.

## 시스템 아키텍처 (워크플로우 구조)

~~~
[1] 스케줄 트리거 (매일 지정 시간 1회 실행, 타임존: Asia/Seoul) - 1일 1회 실행으로 무한 루프 트리거 방지
        |
[2] RSS 수집 - Watch RSS feed items
        |
[3] 중복 검사 - Notion: Search Objects
      - 중복방지 ID(GUID)로 기존 항목 존재 여부 확인
        |
[4] 주제 필터링 - 핵심 키워드 미통과 시 중단 (불필요한 API 호출 차단)
        |
[5] AI 요약 - OpenAI: Generate a Completion (3줄 이내)
        |
[6] 저장 - Notion: Create a Database Item
      - 제목 / 요약문 / 원문 링크 / 발행일시 / 중복방지 ID / 출처 매핑
        |
[7] 예외 처리 - Retry (5분 간격, 최대 2회) -> 실패 시 스킵
~~~

## 기능 요구사항 충족 현황

| 요구사항 | 구현 방식 | 상태 |
| --- | --- | --- |
| RSS로 기술 뉴스 자동 수집 | Make Watch RSS feed items (다중 매체 피드) | OK |
| 매일 정해진 시간 자동 트리거 | 스케줄 트리거 (Asia/Seoul) | OK |
| 주제 해당 기사 선택 + 제목/링크/본문 추출 | RSS 항목에서 title·link·description 추출 | OK |
| 생성형 AI로 3줄 이내 요약 | OpenAI Generate a Completion - 3줄 요약 프롬프트 | OK |
| Notion DB 자동 저장 (속성 매핑) | Create a Database Item - 제목·요약문·원문 링크·발행일시 | OK |
| 무인 자동 완료 | 스케줄 기반 완전 자동 실행 | OK |
| 오류/미수집 시 처리 | Retry(최대 2회) 후 스킵 | OK |
| 동일 기사 중복 저장 방지 | 중복방지 ID(GUID) 기반 검사 | OK |
| 요약 호출 1건당 1회 | 키워드 사전 필터로 불필요 호출 차단 | OK |
| 재시도 최대 2회 상한 | Retry 횟수 2회로 제한 | OK |

## 주제 필터링 기준과 선택 이유

관련 없는 기사를 걸러 정확도를 높이고 불필요한 API 호출 비용을 줄이기 위해 **2단계 필터링**을 적용했습니다.

**1단계 - RSS 키워드 필터링**

AI 서비스명·브랜드명 핵심 키워드 11개로 1차 선별하여, 관련 없는 기사를 AI 호출 이전에 제외합니다. (불필요한 API 호출 절감 목적)

> 생성형 AI, 챗GPT, LLM, 오픈AI, 클로드, 제미나이, Generative AI, ChatGPT, OpenAI, Claude, Gemini

**2단계 - OpenAI 프롬프트 필터링**

키워드가 단순 언급된 기사가 아니라 AI를 실질적으로 다루는 기사만 남기기 위해, 프롬프트 단계에서 한 번 더 선별합니다.

- **기본 개념어:** artificial intelligence, 인공지능, machine learning, 머신러닝, deep learning, 딥러닝, generative, 생성형, LLM, foundation model, 파운데이션 모델, multimodal, 멀티모달, agent, 에이전트, reasoning, thinking, 추론, RAG, 검색 증강 생성, fine-tuning, 파인튜닝, vector DB, 벡터 DB, token, 토큰, context, 컨텍스트
- **트렌드 신호어:** agentic, 에이전틱, computer use, tool call, 도구 호출, memory, 메모리, on-device, 온디바이스, edge, 엣지, realtime, streaming, open model, 오픈소스, open weights, 가중치, eval, benchmark, 벤치마크, inference, latency, context length
- **공급사명:** OpenAI, Anthropic, Google, DeepMind, Meta, Mistral, Microsoft, NVIDIA, Amazon, xAI, Hugging Face, OpenRouter
- **제품/플랫폼명:** ChatGPT, Claude, Gemini, Gemma, Llama, Mistral, DeepSeek, Qwen, Copilot, Bedrock, Vertex AI, Azure AI Foundry, Ollama, LM Studio, vLLM, TensorRT-LLM

## 중복 방지 정책

- **중복방지 키:** 기사 고유 식별자인 **GUID**를 `중복방지 ID` 속성에 저장하여 사용합니다.
- **판별 위치:** 첫 번째 Notion 모듈(Search Objects)에서 **Title(제목)이 아닌 중복방지 ID(GUID)** 기준으로 기존 항목을 검사합니다. 제목은 매체·재발행에 따라 미세하게 달라질 수 있어 중복 판별이 부정확하므로, 고유 ID 기반으로 변경해 동일 기사가 한 번만 저장되도록 했습니다.
- **요약 호출:** 사전 키워드 필터로 불필요한 기사를 제외하여, 기사 1건당 요약 호출 1회 원칙(재시도 제외)을 유지합니다.

## 에러 처리 정책과 선택 이유

- OpenAI 모듈과 Notion 저장 모듈에 **Retry** 모듈을 연결하여, 작업 실패 시 **5분 간격으로 최대 2회 재시도**합니다.
- 일시적인 네트워크 오류나 API 응답 지연으로 인한 실패를 자동으로 복구하기 위함이며, 무한 재시도로 인한 불필요한 비용 발생을 막기 위해 상한을 2회로 제한했습니다.

## 출처(매체명, 자동 분류)

향후 모든 피드를 하나의 시나리오로 통합해도 동작하도록, 기사 링크의 도메인을 기준으로 매체명을 자동 판별하여 `출처` 속성에 저장합니다.

~~~
if(contains(link, "etnews"), "전자신문",
if(contains(link, "aitimes"), "AI타임스",
if(contains(link, "hada.io"), "GeekNews",
if(contains(link, "techcrunch"), "TechCrunch",
if(contains(link, "technologyreview"), "MIT Technology Review",
if(contains(link, "ycombinator"), "Hacker News",
"기타"))))))
~~~

## RSS 피드 목록

아래 매체가 공식 제공하는 RSS 피드만 사용하며, 무단 크롤링·스크래핑 없이 합법적으로 수집합니다.

- 전자신문 - http://rss.etnews.com/04046.xml
- AI타임스 - https://www.aitimes.kr/rss/allArticle.xml
- GeekNews - https://news.hada.io/rss/news
- TechCrunch (AI) - https://techcrunch.com/category/artificial-intelligence/feed/
- MIT Technology Review (AI) - https://www.technologyreview.com/topic/artificial-intelligence/feed/
