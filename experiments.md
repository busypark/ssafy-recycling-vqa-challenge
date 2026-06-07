# 실험 기록

## 컨벤션

### 브랜치

| 브랜치 | 설명 |
|---|---|
| `main` | 최종 파이프라인에 살아남은 흐름 |
| `exp/{name}` | 시도했다가 방향을 바꾼 실험 |

### 커밋 타입

| 타입 | 의미 |
|---|---|
| `classify:` | 질문 유형 분류·라우팅 |
| `label:` | YOLO-World 기반 자동 라벨링 |
| `format:` | 학습용 데이터 포맷 변환 |
| `preprocess:` | 이미지 전처리 |
| `train:` | 모델 파인튜닝·학습 |
| `infer:` | 추론 및 제출파일 생성 |
| `exp:` | 실패 또는 미실행 실험 시도 |
| `pivot:` | 방향 전환 / 실험 포기 |

---

## 실험 환경

| 항목 | 내용 |
|---|---|
| 컨테이너 | Docker (Linux) |
| 인터페이스 | Jupyter Notebook |
| GPU | NVIDIA RTX 5060Ti |
| Python | 3.11 |
| CUDA | 12.8 |
| PyTorch | 2.7.0+cu128 |
| ultralytics | 8.4.33 |
| transformers | 4.57.6 |
| peft | 0.18.1 |
| trl | 0.24.0 |
| bitsandbytes | 0.49.2 |
| datasets | 4.8.4 |

---

## `main`

### 2026-04-02 (목) 11:17:52 · `classify: 정규식 기반 질문 유형 라우터 v01`
> 관련 파일: `[1][v01] classify questions by expression.ipynb`

- **가설** 질문 텍스트와 선택지에 대한 정규식 패턴으로 개수 세기(YOLO 트랙)와 그 외(VLM 트랙)를 충분히 정확하게 분류할 수 있다
- **설정** 정규식 5종 (카운팅 키워드·선택지 수량 패턴·재질명·질문 유형·물건) / OCR 예외 처리(적힌·글자·라벨·로고 등 감지 시 VLM 우회) / "무엇" 포함 시 VLM 우회 / 전체 데이터셋 처리 (`head(100)` 제거)
- **결과** train_class.csv(5,073행), dev_class.csv(4,413행), test_class.csv(5,074행) 생성. YOLO 트랙: train 1,747건 / dev 3,144건, VLM 트랙: train 3,326건 / dev 1,269건
- **판단** 라우터 완성. 이후 YOLO 트랙과 VLM 트랙으로 분기하여 각각 전처리 및 학습 진행. 분류 결과는 이후 모든 단계([2-1]~[5])의 기준으로 사용됨

---

### 2026-04-02 (목) 13:05:28 · `label: YOLO-World 기반 재활용품 이미지 자동 라벨링`
> 관련 파일: `[2-1] yolo auto-labeling.ipynb`

- **가설** YOLO-World의 오픈 보캐블러리 탐지 기능으로 수작업 없이 재활용품 이미지에 YOLO 형식 라벨을 자동 생성할 수 있다
- **설정** YOLOv8s-World / 재활용품 9개 클래스 정의 (plastic bottle, glass bottle, aluminum can, paper box, plastic cup, paper cup, plastic bag, styrofoam, tube container) / conf=0.1 (저신뢰도 허용, 최대한 많이 탐지) / YOLO 트랙(`class == 'YOLO'`) 이미지만 처리
- **결과** yolo/images/train(1,747장) / yolo/images/val(3,144장) 이미지 복사, yolo/labels/{train,val}/ 라벨 .txt 파일 생성, dataset.yaml 자동 생성
- **판단** 자동 라벨링 완료. 단, labels.jpg로 확인한 클래스 분포에서 plastic bag(2건)·styrofoam(90건) 등 극소 클래스가 존재하며, 이 불균형은 이후 어떤 단계에서도 재라벨링·오버샘플링·클래스 가중치 조정 등으로 보정되지 않음

---

### 2026-04-02 (목) 14:41:09 · `format: VLM 트랙 학습용 JSONL 생성`
> 관련 파일: `[2-2] Prompt Formatting - JSONL.ipynb`

- **가설** 질문·선택지·정답을 Qwen-VL 표준 프롬프트 포맷으로 변환하면 SFT 파인튜닝에 바로 투입할 수 있다
- **설정** YOLO 트랙 제외한 VLM 트랙만 필터링 / train 정답: 단일 answer 컬럼 사용 / dev 정답: answer1~answer5 다수결(Counter 최빈값) / 프롬프트 포맷: `<image>경로\n질문: ...\na. ...\nb. ...\nc. ...\nd. ...\n정답:`
- **결과** train_vlm.jsonl(3,326건), dev_vlm.jsonl(1,269건) 생성
- **판단** VLM 학습 데이터 완성. 이미지 경로 업데이트를 위해 [2-3] 리사이징 후 JSONL 재생성 필요

---

### 2026-04-02 (목) 15:53:37 · `preprocess: VLM 이미지 768px 리사이징 및 JSONL 경로 갱신`
> 관련 파일: `[2-3] Resize images.ipynb`

- **가설** 이미지를 768px로 줄이면 VRAM 사용량을 낮추면서도 Qwen-VL 성능 저하 없이 학습시킬 수 있다
- **설정** MAX_SIZE=768 / Pillow thumbnail (비율 유지) / LANCZOS 리샘플링 / JPEG quality=85 / VLM 트랙 이미지만 대상 / 리사이징 후 JSONL 내 이미지 경로를 `resized_images/`로 일괄 치환
- **결과** resized_images/train(3,326장), resized_images/dev(1,269장), resized_images/test(3,365장) 저장 / train_vlm_resized.jsonl, dev_vlm_resized.jsonl 생성
- **판단** VRAM 효율화 전처리 완료. 이후 Qwen2-VL 파인튜닝은 resized 버전 사용

---

### 2026-04-03 (금) 09:24:03 · `train: YOLOv10-S 파인튜닝`
> 관련 파일: `[3] YOLOv10-S 파인튜닝.ipynb`

- **가설** auto-labeling으로 구축한 재활용품 탐지 데이터셋으로 YOLOv10-S를 파인튜닝하면 개수 추론 정확도가 높아진다
- **설정** YOLOv10-S (yolov10s.pt) / epochs=50 / imgsz=640 / batch=16 / patience=15 (조기 종료) / device=0 / dataset.yaml
- **결과** 50 epoch 학습 완료. train/val loss (box, cls, dfl) 모두 수렴 확인 (results.png). 전체 mAP@0.5 = 0.433, 최적 F1 = 0.45 (confidence 0.263 기준). 클래스별로는 glass bottle(PR-AUC 0.733)·plastic cup(0.735)이 상위, tube container(0.079)·plastic bag(0.166)·aluminum can(0.202)이 하위. confusion matrix에서 plastic cup ↔ paper cup 혼동이 두드러지며, plastic bottle·aluminum can은 background 미탐 비율이 높음. best.pt를 VQA_YOLO/yolov10_run_1/weights/에 저장
- **판단** results.png의 loss 수렴을 확인하고 다음 단계로 진행. 단, 분석에서 도출된 인사이트(최적 confidence 임계값 0.263, 특정 클래스 간 혼동 패턴, 낮은 recall 클래스 존재)는 이후 커밋 11의 추론 파이프라인에 반영되지 않음. 커밋 11은 클래스 구분 없이 박스 개수만 사용하므로 클래스 혼동 이슈는 구조적으로 무관하나, confidence 임계값 미조정 및 recall 보완 부재는 잠재적 한계로 남음

---

### 2026-04-03 (금) 13:51:29 · `train: Qwen2-VL-2B QLoRA 파인튜닝`
> 관련 파일: `[3] YOLOv10-S 파인튜닝.ipynb` (하단 셀)

- **가설** QLoRA(4-bit 양자화 + LoRA)로 Qwen2-VL-2B-Instruct를 경량 파인튜닝하면 VQA 성능이 향상된다
- **설정** Qwen2-VL-2B-Instruct / BitsAndBytes 4-bit NF4 양자화 (double quant, bfloat16 compute) / LoRA r=16, alpha=32, target_modules=[q_proj, v_proj], dropout=0.05 / epochs=3 / batch=2 / gradient_accumulation=4 / lr=2e-4 / SFTConfig + SFTTrainer (paged_adamw_8bit) / train_vlm_resized.jsonl 사용
- **결과** 학습 가능 파라미터 2,179,072개 (전체 2,211,164,672개의 0.0985%) / VQA_Qwen_LoRA/final_model/ 저장 완료
- **판단** VLM 트랙 모델 파인튜닝 완료. [5]에서 앙상블 추론에 투입

---

### 2026-04-03 (금) 16:42:58 · `infer: YOLO + Qwen2-VL 앙상블 추론 및 제출`
> 관련 파일: `[5] ensemble and inference.ipynb`

- **가설** 질문 유형에 따라 YOLO(개수 세기)와 Qwen2-VL(나머지)을 분리 적용하면 단일 모델보다 전체 정확도가 높아진다
- **설정** [YOLO 트랙] best.pt 로드 → 박스 탐지 수와 선택지 숫자 최소 오차 선택 / [VLM 트랙] 파인튜닝된 Qwen2-VL 로드, max_new_tokens=2로 알파벳만 추출, a·b·c·d 외 응답 시 포함 알파벳 재추출 후 없으면 'a' fallback / YOLO 추론 완료 후 gc.collect() + cuda.empty_cache()로 VRAM 반환 후 VLM 로드
- **결과** YOLO 트랙 1,709건 추론 (약 53초, 31.95 it/s) / VLM 트랙 3,365건 추론 (약 37분, 1.51 it/s) / submission.csv 생성 완료
- **판단** 최종 제출 완료. YOLO 추론은 클래스 무관 박스 카운트 방식이므로 커밋 09에서 확인된 클래스 혼동 패턴은 구조적으로 영향 없음. 다만 커밋 09의 BoxF1 분석에서 도출된 최적 confidence 임계값(0.263)은 적용하지 않고 기본값을 사용했으며, 낮은 recall로 인한 박스 누락 가능성도 별도로 보완하지 않음

---

## `exp/question-router`

> 분기 시점: `main`의 질문 분류 방법 탐색 단계 ([1][v01] 이전)
> 포기 및 복귀: 정규식 기반 v01([1][v01])로 수렴

---

### 2026-04-02 (목) 09:12:34 · `exp: LLM 기반 질문 라우터 시도`
> 관련 파일: `[1][test][] classify questions by transformer.ipynb`

- **가설** Qwen2.5-3B-Instruct에 few-shot 예시 3개를 주입하면 별도 학습 없이도 질문을 YOLO/VLM으로 정확하게 라우팅할 수 있다
- **설정** Qwen2.5-3B-Instruct (float16, device_map=auto) / few-shot 3개 (개수 세기→YOLO, OCR→VLM, 종류→VLM) / max_new_tokens=3 / temperature=0.01 / head(100) 테스트
- **결과** 100행만 테스트 (전체 데이터 미적용). 추론 결과는 저장되나 정확도 별도 미측정
- **판단** VRAM 부담(3B 모델 상시 로드)과 추론 속도 문제로 LLM 라우터 포기. 경량 정규식 방식으로 전환 → `pivot`

---

### 2026-04-02 (목) 10:03:47 · `pivot: LLM 라우터 → 정규식 라우터로 방향 전환`
> 관련 파일: `[1][test][] classify questions by transformer.ipynb`

- **판단** LLM 라우터는 VRAM과 처리 속도 측면에서 불리. 규칙 기반 정규식 분류([1][test][v00])로 전환

---

### 2026-04-02 (목) 10:28:15 · `exp: 정규식 라우터 v00 프로토타입`
> 관련 파일: `[1][test][v00] classify questions by expression.ipynb`

- **가설** 질문과 선택지에 대한 정규식만으로 YOLO/VLM을 충분히 분류할 수 있다
- **설정** 정규식 5종 (카운팅·선택지 수량·재질·질문유형·물건) / head(100) 테스트 / axis=1 row 단위 적용
- **결과** train_class.csv(100행), dev_class.csv(100행), test_class.csv(100행) 생성. 그러나 `classify_question_advanced()` 함수 내 YOLO 분기 로직 이후 `return "VLM"` 하드코딩이 남아 있어 **모든 행이 VLM으로 분류되는 버그** 존재 (선택지 로직은 주석 처리된 채로 방치됨)
- **판단** 버그 발견. OCR 예외 처리 추가, "무엇" 우회 조건 보완, 전체 데이터셋 처리로 수정하여 v01([1][v01])로 재작성

---

## `exp/qwen-sft-v1`

> 분기 시점: [3] YOLO 파인튜닝 이후, Qwen2-VL 파인튜닝 방법 탐색
> 포기 및 복귀: SFTConfig 방식([3] 하단 셀)으로 수정 후 실행

---

### 2026-04-03 (금) 09:06:41 · `exp: Qwen2-VL SFTTrainer + TrainingArguments 조합 초안`
> 관련 파일: `[4] Qwen2-VL 파인튜닝.ipynb`

- **가설** 기존 TrainingArguments와 SFTTrainer를 조합하면 Qwen2-VL QLoRA 파인튜닝이 가능하다
- **설정** Qwen2-VL-2B-Instruct / BitsAndBytes 4-bit NF4 / LoRA r=16 / TrainingArguments + SFTTrainer (dataset_text_field="messages")
- **결과** 미실행 (셀 출력 없음)
- **판단** 실행하지 않고 폐기. `dataset_text_field`는 SFTConfig에서 처리해야 하므로 TrainingArguments 조합은 호환되지 않는 것으로 판단. [3] 노트북 하단에 SFTConfig 방식으로 수정하여 재작성 후 실행
