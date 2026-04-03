# train: Qwen2-VL-2B QLoRA 파인튜닝

**날짜** 2026-04-03 (금) 13:51:29
**브랜치** `main`
**타입** `train`

> 관련 파일: `[3] YOLOv10-S 파인튜닝.ipynb` (하단 셀)

## 가설

QLoRA(4-bit 양자화 + LoRA)로 Qwen2-VL-2B-Instruct를 경량 파인튜닝하면 VQA 성능이 향상된다

## 설정

- 모델: Qwen2-VL-2B-Instruct
- 양자화: BitsAndBytes 4-bit NF4 (double quant, bfloat16 compute)
- LoRA: r=16, alpha=32, target_modules=[q_proj, v_proj], dropout=0.05
- 학습: epochs=3, batch=2, gradient_accumulation=4, lr=2e-4
- 옵티마이저: paged_adamw_8bit
- 프레임워크: SFTConfig + SFTTrainer
- 데이터: train_vlm_resized.jsonl (3,326건)

## 결과

- 학습 가능 파라미터: 2,179,072개 (전체 2,211,164,672개의 **0.0985%**)
- 저장 위치: VQA_Qwen_LoRA/final_model/

## 판단

VLM 트랙 모델 파인튜닝 완료. [5]에서 앙상블 추론에 투입
