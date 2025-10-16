# 📘 pragmatic-llm-search (검색+요약 LLM 챗봇) - 진행 중

## 프로젝트 개요
이 프로젝트는 **7B~10B급 오픈소스 LLM**을 기반으로,  
**실시간 검색(RAG) + 요약 + 번역** 기능을 제공하는 **AI 검색·요약 챗봇**을 개발하는 것을 목표로 합니다.  

- 사용자는 자연어로 질문 → 모델이 검색 쿼리로 변환 → 검색 결과를 기반으로 답변 생성  
- 최신 정보는 **실시간 웹 검색 API**를 통해 보완  

---

## 목표
1. **소규모 LLM(7B~10B)**을 활용해 대형 모델 수준의 검색 정확도 달성  
2. **RAG + RLHF(DPO)**를 통해 신뢰성 있는 답변 생성  
3. **실시간 검색 API**를 통합하여 최신성 확보  
4. **HF Space 배포**로 SaaS 형태의 서비스 프로토타입 제공  

---

## 기술 스택
- **모델**: LLaMA 3 8B, Mistral 7B, Yi 9B (LoRA/QLoRA 기반 파인튜닝)  
- **프레임워크**: PyTorch, Hugging Face Transformers, PEFT  
- **검색**: FAISS / Weaviate + BM25, Re-ranker (monoT5)  
- **서빙**: Triton Inference Server / vLLM, Docker  / HF Space  
- **백엔드**: FastAPI  
- **프론트엔드**: React / Streamlit / gradio  
- **클라우드[optional]**: Azure App Service, Azure Container Apps  

---

## 데이터셋 설계
- **코퍼스**: 기술 블로그, 위키 문서, FAQ, Markdown 글  
- **Query–Doc 쌍**: 자연어 질문 ↔ 관련 문서 매핑  
- **Answer 데이터**: 검색 결과 기반 요약/답변 (출처 포함)  
- **Preference 데이터(DPO)**: 좋은 답변 vs 나쁜 답변 비교  

---

## 학습 단계
1. **SFT (Supervised Fine-Tuning)**  
   - Query Rewrite + Answer Synthesis 멀티태스크 학습  
   - LoRA/QLoRA 적용  

2. **DPO (Direct Preference Optimization)**  
   - Faithfulness/Completeness 기준으로 선호 데이터 학습  
   - 환각 줄이고 출처 포함 강화  

3. **Retrieval 튜닝**  
   - Dense Embedding + Re-ranker 적용  

---

## 서비스 아키텍처
1. 사용자 질문 입력  
2. 모델이 검색 쿼리로 변환  
3. 로컬 벡터DB 검색 + 필요 시 웹 검색 API 호출  
4. 상위 문서 Re-rank  
5. 모델이 답변 생성 (출처 포함)  
6. UI에 결과 표시  

---

## 현재 진행 상황
- [x] 프로젝트 기획 및 아키텍처 설계  
- [x] 데이터 구조 설계 (코퍼스, Query–Doc, Preference)  
- [ ] 코드 스켈레톤 작성 (데이터 로딩, 검색 파이프라인, API 기본 구조)  
- [ ] LoRA 기반 SFT 진행 예정  
- [ ] DPO 학습 및 Re-ranker 적용 예정  
- [ ] Azure 배포 및 데모 페이지 제작 예정  

---

## 향후 계획
- 1주차: 코드 스켈레톤 + 소규모 데이터셋 구축  
- 2주차: LoRA 기반 SFT 학습 및 API 연결  
- 3주차: DPO 학습 + Re-ranker 적용  
- 4주차: Azure 배포 + 데모 영상 제작  

---

## 기대 효과
- **실용주의 AI** 철학에 맞는 서비스형 LLM 프로토타입  
- 소규모 모델로도 대형 모델 수준의 검색 정확도 달성 가능성 검증  

---
