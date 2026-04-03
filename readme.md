# train: YOLOv10-S 파인튜닝

**날짜** 2026-04-03 (금) 09:24:03
**브랜치** `main`
**타입** `train`

> 관련 파일: `[3] YOLOv10-S 파인튜닝.ipynb`

## 가설

auto-labeling으로 구축한 재활용품 탐지 데이터셋으로 YOLOv10-S를 파인튜닝하면 개수 추론 정확도가 높아진다

## 설정

- YOLOv10-S (yolov10s.pt)
- epochs=50 / imgsz=640 / batch=16
- patience=15 (조기 종료) / device=0
- dataset.yaml

## 결과

### 학습 곡선

![train/val loss 및 mAP 곡선 (50 epoch)](images/results.png)

50 epoch 학습 완료. train/val loss (box, cls, dfl) 모두 수렴.

### 최종 성능 지표

- 전체 mAP@0.5 = **0.433**
- 최적 F1 = **0.45** (confidence 0.263 기준)

### Precision-Recall 곡선

![클래스별 PR 곡선](images/BoxPR_curve.png)

### F1-Confidence 곡선

![클래스별 F1-Confidence 곡선](images/BoxF1_curve.png)

### Precision / Recall 곡선

![Precision-Confidence 곡선](images/BoxP_curve.png)

![Recall-Confidence 곡선](images/BoxR_curve.png)

### 혼동 행렬

![혼동 행렬 (raw counts)](images/confusion_matrix.png)

![혼동 행렬 (정규화)](images/confusion_matrix_normalized.png)

plastic cup ↔ paper cup 혼동이 두드러짐. plastic bottle·aluminum can은 background 미탐 비율이 높음.

### 학습 배치 시각화 (초기)

![train_batch0](images/train_batch0.jpg) ![train_batch1](images/train_batch1.jpg) ![train_batch2](images/train_batch2.jpg)

### 학습 배치 시각화 (후기)

![train_batch4400](images/train_batch4400.jpg) ![train_batch4401](images/train_batch4401.jpg) ![train_batch4402](images/train_batch4402.jpg)

### 검증셋 정답 vs 예측

| Ground Truth | Prediction |
|---|---|
| ![val_batch0_labels](images/val_batch0_labels.jpg) | ![val_batch0_pred](images/val_batch0_pred.jpg) |
| ![val_batch1_labels](images/val_batch1_labels.jpg) | ![val_batch1_pred](images/val_batch1_pred.jpg) |
| ![val_batch2_labels](images/val_batch2_labels.jpg) | ![val_batch2_pred](images/val_batch2_pred.jpg) |

best.pt를 VQA_YOLO/yolov10_run_1/weights/에 저장

## 판단

results.png의 loss 수렴을 확인하고 다음 단계로 진행. 단, 분석에서 도출된 인사이트(최적 confidence 임계값 0.263, 특정 클래스 간 혼동 패턴, 낮은 recall 클래스 존재)는 이후 커밋 11의 추론 파이프라인에 반영되지 않음. 커밋 11은 클래스 구분 없이 박스 개수만 사용하므로 클래스 혼동 이슈는 구조적으로 무관하나, confidence 임계값 미조정 및 recall 보완 부재는 잠재적 한계로 남음
