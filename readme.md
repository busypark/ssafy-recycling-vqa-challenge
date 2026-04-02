# exp: 정규식 라우터 v00 프로토타입

**날짜** 2026-04-02 (목) 10:28:15
**브랜치** `exp/question-router`
**타입** `exp`

> 관련 파일: `[1][test][v00] classify questions by expression.ipynb`

## 가설

질문과 선택지에 대한 정규식만으로 YOLO/VLM을 충분히 분류할 수 있다

## 설정

- 정규식 5종 (카운팅·선택지 수량·재질·질문유형·물건)
- head(100) 테스트
- axis=1 row 단위 적용

## 결과

train_class.csv(100행), dev_class.csv(100행), test_class.csv(100행) 생성. 그러나 `classify_question_advanced()` 함수 내 YOLO 분기 로직 이후 `return "VLM"` 하드코딩이 남아 있어 **모든 행이 VLM으로 분류되는 버그** 존재 (선택지 로직은 주석 처리된 채로 방치됨)

## 판단

버그 발견. OCR 예외 처리 추가, "무엇" 우회 조건 보완, 전체 데이터셋 처리로 수정하여 v01([1][v01])로 재작성
