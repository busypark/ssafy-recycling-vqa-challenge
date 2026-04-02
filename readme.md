# format: VLM 트랙 학습용 JSONL 생성

**날짜** 2026-04-02 (목) 14:41:09
**브랜치** `main`
**타입** `format`

> 관련 파일: `[2-2] Prompt Formatting - JSONL.ipynb`

## 가설

질문·선택지·정답을 Qwen-VL 표준 프롬프트 포맷으로 변환하면 SFT 파인튜닝에 바로 투입할 수 있다

## 설정

- YOLO 트랙 제외한 VLM 트랙만 필터링
- train 정답: 단일 answer 컬럼 사용
- dev 정답: answer1~answer5 다수결 (Counter 최빈값)
- 프롬프트 포맷: `<image>경로\n질문: ...\na. ...\nb. ...\nc. ...\nd. ...\n정답:`

## 결과

- train_vlm.jsonl: 3,326건
- dev_vlm.jsonl: 1,269건

## 판단

VLM 학습 데이터 완성. 이미지 경로 업데이트를 위해 [2-3] 리사이징 후 JSONL 재생성 필요
