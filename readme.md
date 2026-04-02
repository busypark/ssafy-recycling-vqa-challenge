# exp: LLM 기반 질문 라우터 시도

**날짜** 2026-04-02 (목) 09:12:34
**브랜치** `exp/question-router`
**타입** `exp`

> 관련 파일: `[1][test][] classify questions by transformer.ipynb`

## 가설

Qwen2.5-3B-Instruct에 few-shot 예시 3개를 주입하면 별도 학습 없이도 질문을 YOLO/VLM으로 정확하게 라우팅할 수 있다

## 설정

- Qwen2.5-3B-Instruct (float16, device_map=auto)
- few-shot 3개 (개수 세기→YOLO, OCR→VLM, 종류→VLM)
- max_new_tokens=3 / temperature=0.01
- head(100) 테스트

## 결과

100행만 테스트 (전체 데이터 미적용). 추론 결과는 저장되나 정확도 별도 미측정

## 판단

VRAM 부담(3B 모델 상시 로드)과 추론 속도 문제로 LLM 라우터 포기. 경량 정규식 방식으로 전환 → `pivot`
