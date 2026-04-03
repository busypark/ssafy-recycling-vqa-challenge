# exp: Qwen2-VL SFTTrainer + TrainingArguments 조합 초안

**날짜** 2026-04-03 (금) 09:06:41
**브랜치** `exp/qwen-sft-v1`
**타입** `exp`

> 관련 파일: `[4] Qwen2-VL 파인튜닝.ipynb`

## 가설

기존 TrainingArguments와 SFTTrainer를 조합하면 Qwen2-VL QLoRA 파인튜닝이 가능하다

## 설정

- Qwen2-VL-2B-Instruct
- BitsAndBytes 4-bit NF4 양자화
- LoRA r=16, alpha=32, target_modules=[q_proj, v_proj]
- TrainingArguments + SFTTrainer (dataset_text_field="messages")

## 결과

미실행 (셀 출력 없음)

## 판단

실행하지 않고 폐기. `dataset_text_field`는 SFTConfig에서 처리해야 하므로 TrainingArguments 조합은 호환되지 않는 것으로 판단. [3] 노트북 하단에 SFTConfig 방식으로 수정하여 재작성 후 실행
