# 📘 pragmatic-llm-search (검색+요약 LLM 챗봇) - 진행 중

## 프로젝트 개요
7B~10B급 오픈소스 LLM을 기반으로 **실시간 검색(RAG) + 요약 + 번역** 기능을 제공하는 AI 검색·요약 챗봇입니다.  
- 사용자는 자연어로 질문 → 모델이 검색 쿼리로 변환 → 검색 결과를 종합해 답변 생성  
- 최신 정보는 **웹 검색 API(Google Custom Search)**로 보완  
- 모델은 추론 단계에서 **프롬프트 체계**를 통해 **플래너(검색 결정/쿼리 리라이트)**와 **생성기(최종 답변 합성)** 역할을 분리해 호출합니다.

---

## 목표
1. 소규모 LLM(7B~10B)로 대형 모델 수준의 검색 정확도 추구  
2. RAG + RLHF(DPO)로 신뢰성/근거 충실한 답변 생성  
3. 실시간 검색 API로 최신성 확보  
4. 프롬프트 체계, 디코딩 정책, RAG 컨텍스트 설계 최적화  
5. Hugging Face Space 배포로 SaaS 형태 시연  
6. 연구적 확장(아키텍처 수준)을 통해 확장 가능성 탐구

---

## 기술 스택
- **모델**
  - Qwen 3 8B (메인 LLM, QLoRA SFT + DPO)
  - 임베더: bge-small / e5-small
  - 리랭커: monoT5-small / e5-reranker
- **프레임워크**: PyTorch, Transformers, PEFT, TRL
- **검색**: FAISS/Weaviate + BM25, Google Custom Search API
- **서빙/배포**: vLLM, Docker, **HF Space**
- **백엔드/프론트**: FastAPI, React/Streamlit/gradio

---

## 데이터셋 설계
- **코퍼스(소규모)**: 기술 블로그/위키/FAQ/공개 RAG 데이터셋
- **Query–Doc 쌍**: 자연어 질문 ↔ 관련 문서 매핑
- **Answer 데이터**: 검색 결과 기반 요약/답변(출처 포함)
- **Preference 데이터(DPO)**: Faithfulness/Completeness 기준의 좋은 vs 나쁜 답변 비교  
- 참고: 역할 분리(플래너/생성기)는 학습 데이터로 분리하지 않고, **추론 단계 프롬프트 체계로 처리**합니다.

---

## 개발 파이프라인

### 1차 개발 ― 서비스 중심 MVP
- Base 모델(Qwen 3 8B)을 그대로 활용  
- RAG 파이프라인 구축: 로컬 벡터DB + Google Search API 결합  
- 프롬프트 체계 설계: 플래너(검색 쿼리 생성/검색 여부 결정)와 생성기(최종 답변 합성) 역할 분리  
- Hugging Face Space에 배포하여 실제 SaaS 형태로 동작하는 검색·요약 챗봇 MVP 시연  

### 2차 개발 ― 모델 학습 확장
- **SFT (KorQuAD v2 기반)**: 검색 결과 기반 답변 형식(요약/핵심/출처 포함)을 학습  
- **DPO (Direct Preference Optimization)**: Faithfulness/Completeness 기준으로 선호 학습 → 환각 감소 및 출처 준수 강화  
- **Synthetic 데이터 소량 추가**:  
  - 한국어 입력 → 영어 검색 쿼리 생성  
  - 다중 컨텍스트 종합 + 출처 포함 답변  
- A/B 테스트를 통해 Base vs Fine-tuned 모델 비교, 서비스 품질 개선 수치화  

### 3차 확장 ― 모델 튜닝 및 연구적 확장
- Cross-attention, multimodal head, encoder 추가 등 아키텍처 수준의 확장  
- LoRA 타깃 다양화(Q/K/V/FFN 조합 비교), speculative decoding, KV-cache 최적화 실험  
- 연구 성격이 강한 단계로, 포트폴리오 확장용  

---

## 추론 파이프라인(프롬프트 체계 포함)
1. **Rate limit/캐시 확인**  
2. **LLM(플래너 프롬프트)**  
   - 질문 의도를 분석하고, 필요 시 검색용 쿼리를 2–3개 생성  
   - 최신/불확실한 경우 검색을 권장  
3. **로컬 검색 + 웹 검색 API**  
   - 벡터DB(BM25+Dense) → Top-k, 필요 시 Google Custom Search API 호출  
4. **리랭킹**  
   - 교차 인코더로 Top-50 → Top-5~10  
5. **컨텍스트 구성**  
   - 핵심 구절/제목/URL/토큰 예산 내 정규화  
6. **LLM(생성기 프롬프트)**  
   - 컨텍스트를 근거로 요약/핵심/출처 포함 답변 생성  
   - 근거 부족 시 “정보 부족” 표기  
7. **사후 검증/재검색**  
   - 출처 누락/충돌 시 재시도  
8. **응답 스트리밍/메타 표시**  
   - 남은 검색 횟수/소스 링크/타임스탬프  
9. **캐시 저장/로그 수집**

---

## 현재 진행 상황
- [ ] **1차 개발 ― 서비스 중심 MVP**
  - [x] 프로젝트 기획 및 전체 아키텍처 설계 완료  
  - [x] 데이터 구조 설계 (코퍼스 구축, Query–Doc 매핑, Preference 데이터 정의)  
  - [ ] 코드 스켈레톤 작성 (모델 학습 루프, 검색 모듈, 파이프라인, API 엔드포인트)  
  - [ ] RAG 파이프라인 초기 구현 (로컬 벡터DB + Google Search API 연동)  
  - [ ] 프롬프트 체계 설계 및 테스트 (플래너/생성기 역할 분리)  
  - [ ] **HF Space 배포 및 MVP 데모 페이지 공개**

- [ ] **2차 개발 ― 모델 학습 확장**
  - [ ] KorQuAD v2 기반 QLoRA SFT 진행  
  - [ ] DPO 학습 및 리랭커 적용 (선호 데이터셋 기반)  
  - [ ] Synthetic 데이터 추가 (검색 쿼리 생성, 다중 컨텍스트 종합/출처 포함 답변)  
  - [ ] A/B 테스트 (Base vs Fine-tuned 모델 성능 비교 및 리포트 작성)  
  - [ ] 서비스 품질 개선 지표화 (정확도, 출처 포함률, 환각 감소율)

- [ ] **3차 개발 ― 모델 튜닝 및 연구적 확장**
  - [ ] Cross-attention, multimodal head, encoder 확장 실험  
  - [ ] LoRA 타깃 다양화(Q/K/V/FFN 조합 비교)  
  - [ ] Speculative decoding, KV-cache 최적화 실험  
  - [ ] 연구 성격의 확장 결과 정리 및 포트폴리오 반영

---

## 향후 계획
- **1주차**: 1차 개발 마무리 → 코드 스켈레톤 완성, RAG 파이프라인 초기 구현, HF Space MVP 배포  
- **2주차**: 2차 개발 착수 → KorQuAD v2 기반 QLoRA SFT, DPO 학습, Synthetic 데이터 반영, 추론 단계 프롬프트/디코딩 정책 최적화  
- **3주차**: Retrieval 튜닝 (BM25+Dense 하이브리드, 리랭커 최적화), 성능 평가 및 개선 리포트 작성  
- **4주차 이후**: 3차 개발 단계 → 모델 튜닝(아키텍처 레벨), LoRA 타깃 다양화, speculative decoding, KV-cache 최적화 등 연구적 확장 실험 진행  

---

## 기대 효과
- 소규모 모델로 대형 모델 수준의 검색 정확도에 접근  
- 근거 충실/출처 포함 응답으로 신뢰성 확보  
- Fine-tuning과 추론 단계 최적화 역량을 모두 증명  
- 후속 아키텍처 확장 가능성을 제시  

---
