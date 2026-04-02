# pivot: LLM 라우터 → 정규식 라우터로 방향 전환

**날짜** 2026-04-02 (목) 10:03:47
**브랜치** `exp/question-router`
**타입** `pivot`

> 관련 파일: `[1][test][] classify questions by transformer.ipynb`

## 판단

LLM 라우터는 VRAM과 처리 속도 측면에서 불리. 규칙 기반 정규식 분류([1][test][v00])로 전환
