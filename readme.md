# preprocess: VLM 이미지 768px 리사이징 및 JSONL 경로 갱신

**날짜** 2026-04-02 (목) 15:53:37
**브랜치** `main`
**타입** `preprocess`

> 관련 파일: `[2-3] Resize images.ipynb`

## 가설

이미지를 768px로 줄이면 VRAM 사용량을 낮추면서도 Qwen-VL 성능 저하 없이 학습시킬 수 있다

## 설정

- MAX_SIZE=768
- Pillow thumbnail (비율 유지)
- LANCZOS 리샘플링
- JPEG quality=85
- VLM 트랙 이미지만 대상
- 리사이징 후 JSONL 내 이미지 경로를 `resized_images/`로 일괄 치환

## 결과

| 분할 | 저장 장수 |
|---|---|
| resized_images/train | 3,326장 |
| resized_images/dev | 1,269장 |
| resized_images/test | 3,365장 |

- train_vlm_resized.jsonl, dev_vlm_resized.jsonl 생성

## 판단

VRAM 효율화 전처리 완료. 이후 Qwen2-VL 파인튜닝은 resized 버전 사용
