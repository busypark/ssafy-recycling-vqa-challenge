# infer: YOLO + Qwen2-VL 앙상블 추론 및 제출

**날짜** 2026-04-03 (금) 16:42:58
**브랜치** `main`
**타입** `infer`

> 관련 파일: `[5] ensemble and inference.ipynb`

## 가설

질문 유형에 따라 YOLO(개수 세기)와 Qwen2-VL(나머지)을 분리 적용하면 단일 모델보다 전체 정확도가 높아진다

## 설정

**YOLO 트랙**
- best.pt 로드
- 박스 탐지 수와 선택지 숫자 최소 오차 선택

**VLM 트랙**
- 파인튜닝된 Qwen2-VL 로드 (bfloat16)
- max_new_tokens=2로 알파벳만 추출
- a·b·c·d 외 응답 시 포함 알파벳 재추출, 없으면 'a' fallback

**VRAM 관리**
- YOLO 추론 완료 후 gc.collect() + cuda.empty_cache()로 반환 후 VLM 로드

## 결과

| 트랙 | 건수 | 소요 시간 | 속도 |
|---|---|---|---|
| YOLO | 1,709건 | 약 53초 | 31.95 it/s |
| VLM | 3,365건 | 약 37분 | 1.51 it/s |

submission.csv 생성 완료

## 판단

최종 제출 완료. YOLO 추론은 클래스 무관 박스 카운트 방식이므로 커밋 09에서 확인된 클래스 혼동 패턴은 구조적으로 영향 없음. 다만 커밋 09의 BoxF1 분석에서 도출된 최적 confidence 임계값(0.263)은 적용하지 않고 기본값을 사용했으며, 낮은 recall로 인한 박스 누락 가능성도 별도로 보완하지 않음
