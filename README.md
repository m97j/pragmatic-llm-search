# 📘 pragmatic-llm-search (검색+요약 LLM 챗봇) - 진행 중

## 프로젝트 개요
7B~10B급 오픈소스 LLM을 기반으로 **실시간 검색(RAG) + 요약 + 번역** 기능을 제공하는 AI 검색·요약 챗봇입니다.  
- 사용자는 자연어로 질문 → 모델이 검색 쿼리로 변환 → 검색 결과를 종합해 답변 생성  
- 최신 정보는 **웹 검색 API(Google Custom Search)**로 보완  
- 모델은 추론 단계에서 **프롬프트 체계**로 **플래너(검색 결정/쿼리 리라이트)**와 **생성기(최종 답변 합성)** 역할을 분리해 호출합니다.

---

## 목표
1. 소규모 LLM(7B~10B)로 대형 모델 수준의 검색 정확도 추구  
2. RAG + RLHF(DPO)로 신뢰성/근거 충실한 답변 생성  
3. 실시간 검색 API로 최신성 확보  
4. **Model Tuning(실무형)**: 프롬프트 체계, 디코딩 정책, RAG 컨텍스트 설계 최적화  
5. HF Space 배포로 SaaS 형태 시연  
6. **연구적 확장(아키텍처 수준)**을 통해 확장 가능성 탐구

---

## 기술 스택
- **모델**
  - LLaMA 3 8B (메인 LLM, QLoRA SFT + DPO)
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

## 학습 및 튜닝 단계
0. **Model Tuning 설계(사전 설계)**  
   - 디코딩 정책(Top-k/Top-p/Temperature), 프롬프트 체계, RAG 컨텍스트 길이/Top-k/하이브리드 가중치 설계

1. **SFT (Supervised Fine-Tuning)**  
   - 검색 결과 기반 답변 형식(요약/핵심 포인트/출처 표기) 중심 학습  
   - QLoRA(4-bit)로 효율 튜닝

2. **DPO (Direct Preference Optimization)**  
   - 근거 충실/핵심 포함/장황성 억제 기준의 선호 학습  
   - 환각 감소 및 출처 준수 강화

3. **Retrieval 튜닝 (Optional)**  
   - BM25+Dense 하이브리드, 청크 크기/오버랩, 리랭커 스코어 조정

4. **Model Tuning(추론 단계 최적화)**  
   - 프롬프트 체계 고도화, 디코딩 정책 미세 조정, 컨텍스트 예산 최적화

5. **연구적 확장(Architecture-level, 이후 단계)**  
   - 멀티태스크 헤드(쿼리 리라이트/답변 합성 분리) 탐구  
   - LoRA 타깃 다양화(Q/K/V/FFN 조합 비교)  
   - Speculative decoding, KV-cache 최적화 실험

---

## 추론 파이프라인(프롬프트 체계 포함)
1. **Rate limit/캐시 확인**
2. **LLM(플래너 프롬프트)**  
   - 고정 prefix: “질문 의도를 분석하고, 필요 시 검색용 쿼리를 2–3개 생성하시오. 최신/불확실하면 검색을 권장하시오.”  
   - 입력: 사용자 질문
3. **로컬 검색 + 웹 검색 API**  
   - 벡터DB(BM25+Dense) → Top-k, 필요 시 Google Custom Search API 호출
4. **리랭킹**  
   - 교차 인코더로 Top-50 → Top-5~10
5. **컨텍스트 구성**  
   - 핵심 구절/제목/URL/토큰 예산 내 정규화
6. **LLM(생성기 프롬프트)**  
   - 고정 prefix: “다음 컨텍스트를 근거로 요약/핵심/출처를 포함해 답하시오. 근거 부족 시 ‘정보 부족’을 표기.”  
   - 입력: 정리된 컨텍스트
7. **사후 검증/재검색**  
   - 출처 누락/충돌 시 재시도
8. **응답 스트리밍/메타 표시**  
   - 남은 검색 횟수/소스 링크/타임스탬프
9. **캐시 저장/로그 수집**

---

## 현재 진행 상황
- [x] 프로젝트 기획/아키텍처 설계
- [x] 데이터 구조 설계(코퍼스, Query–Doc, Preference)
- [ ] 코드 스켈레톤(모델 학습/검색/파이프라인/API)
- [ ] QLoRA 기반 SFT
- [ ] DPO 학습 및 Re-ranker 적용
- [ ] 추론 단계 Model Tuning(프롬프트/디코딩 정책)
- [ ] **HF Space 배포 및 데모 페이지**

---

## 향후 계획
- 1주차: 코드 스켈레톤 + 소규모 데이터셋 구축  
- 2주차: QLoRA SFT + API 연결  
- 3주차: DPO + 추론 단계 Model Tuning  
- 4주차: Retrieval 튜닝(Optional) + **HF Space 배포**  
- 5주차 이후: 연구적 확장(아키텍처 수준) 실험

---

## 기대 효과
- 소규모 모델로 대형 모델 수준의 검색 정확도에 접근  
- 근거 충실/출처 포함 응답으로 신뢰성 확보  
- **Fine-tuning + 추론 단계 Model Tuning** 역량을 모두 증명  
- 후속 **아키텍처 확장** 가능성을 제시
