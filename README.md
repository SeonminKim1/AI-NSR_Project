# NSR_Project
국가보안기술연구소 (NSR) 프로젝트

# ■ 바이너리 대상 컴파일러 및 함수정보 추출 기계학습 기술 연구

## ■ 연구 필요성
- 다양한 바이너리 분석 도구
  - BAP, BitBlaze, BinNavi, IDA Pro 등
  - 스트립(stripped) 바이너리 분석 정확도 낮음

- 리눅스용 컴파일러
  - gcc 7.2, 8.0, 8.3, 9.0 …

- 스트립 바이너리 분석 어려움
  - 디버그 심볼이 없음 -> 역공학 어려움
  - 컴파일러 종류/버전/최적화 정도 파악 어려움
  - 함수 위치 파악 어려움

- 스트립 악성 코드 분석을 위한 스트립 바이너리 분석 기술 필요

## ■ 연구개발 목표
- 기계학습 기반의 스트립(stripped) 리눅스 바이너리 분석 기술
  - 스트립 바이너리 제작에 사용된 컴파일러 탐지 기술
    - 바이너리 대상 컴파일러 종류 및 버전 판별 기계학습 모델
  - 스트립 바이너리에서 함수 위치 탐지 기술
    - 함수 위치정보 (시작, 종료) 추출 기계학습 모델 연구
    - 함수 Basic block 정보 추출 기술 연구

## ■ 연구 내용 1 - 리눅스 바이너리 DB
- 리눅스 바이너리 수집과 학습 데이터 생성
  - 리눅스 주요 배포판에 포함된 소스 코드와 바이너리 수집
  - 리눅스 바이너리에서 사용된 컴파일러 종류, 최적화 옵션, 함수 정보 획득 
  - 스트립 바이너리 생성
  - 리눅스 바이너리 데이터베이스 구축

![linux_binary_db](readme_img/linux_binary_db.png')

## ■ 연구 내용 2 - 컴파일러 정보 추출
- 스트립 리눅스 바이너리 제작에 사용된 컴파일러 정보 추출 기술 연구
  - 바이너리 대상 컴파일러 종류 및 버전 판별 기계학습 모델
  - 탐지를 위해 총 3가지 탐지 모델 학습
    - 컴파일러 탐지 모델
    - 컴파일러 버전 탐지 모델
    - 최적화 옵션 탐지 모델
  - 컴파일러는 gcc와 clang/llvm을 대상으로 함
  - 컴파일러 버전은 2개 이상, 최적화 옵션은 2개 이상을 구분 탐지 가능하도록 연구함 

![compiler_detection_hybrid_model](readme_img/compiler_detection_hybrid_model.png')

## ■ 연구 내용 3 - 함수 정보 추출
- 스트립된 리눅스 바이너리에서 함수정보 추출 기술 연구
  - 함수 위치정보 (시작, 종료) 추출 기계학습 모델 연구
  - 스트립된 바이너리에서, K개의 함수 시작 주소와 크기를 추출하는 기계학습 모델 설계
  
- 학습 단계
  - 피처 벡터 추출
    - 스트립된 바이너리와 스트립 되기 전에 파악한 함수 정보를 입력으로 피처 벡터를 추출
    - 피처 벡터의 크기는 다양한 크기로 시도하여 분류 정확도가 높은 크기로 결정
  - 기계학습 – 모델 선정
    - 다양한 기계학습 모델을 이용하여 학습/테스팅 - 정확도가 높은 모델 선정
    - SVM, 결정트리, 딥러닝, 랜덤 포레스트 등
   
- 테스팅/분류 단계
  - 분류
    - 선정된 기계학습 모델을 이용하여 분류
    - 추출된 모든 피처 벡터들에 대해 분류하여 함수 시작위치인지 판단
  - 함수정보 출력
    - 함수의 마지막인 리턴 부분을 찾아서 함수의 시작/크기 확인
  - 함수정보 개선
    - 찾아낸 함수들을 이용하여 호출 그래프를 작성, 부정확한 함수 제거하여 정확도 향상
    
![function_info_extraction](readme_img/function_info_extraction.png')

## 진행상황 (연구내용 3)

### 1. 자료조사 및 관련 논문 리뷰 (20.04.15) 
    - 문서 : 논문 : Recognizing functions in binaries with neural networks_augsut 2015 (USENIX)
    - 문서 : 1차 정리 파일 document/(20.04.15)김선민 Recognizing Functions in Binaries with Neural Networks 요약본

<hr>

### 2. Bidirectional RNN 구현 (1) 및 Stripped Binary 관련 공부 (20.05.08)
    - 문서 : (20.05.08) Recognizing Functions in Binaries with Neural Networks 구현 (논문 부분적 내용 정리)
    - 문서 : (20.05.08) 김선민,장두혁 Bidirectional RNN 구현 및 Stripped Binary 공부
        - 소스코드 : Preprocessing/Preprocessing_Save_hexByteCode.ipynb

<hr>

### 3. BIRNN 구현 (2)  (20.06.05)
    - 문서 : (20.06.05) BIRNN 구현
    - BIRNN 구현 시도 -> Imbalanced Data 문제 발생
    - Imbalanced Data 문제 해결 방법으로 -> 함수 시작 주변 N byte 자르는 방식생각 
        - 소스코드 : RNN_by_All_Binary/BiRNN_gcc5_op0~2_by_All_Binary.ipynb

<hr> 

### 4. Imbalanced Data에 대한 솔루션 (20.06.23)
    - 문서 : (20.06.23) Imbalanced Data에 대한 Solution -> 함수 시작(1) 주변 N byte 자르는 방식 생각
    - 논문 내용 따라 10-fold Cross Validation 진행 (RNN의 Sequential 한 특징 때문에 수동 교차검증? 진행)
    - 다양한 N-Byte 기준 실험 진행
    - 10 N-Byte 기준 함수의 시작 주변 양쪽이 아닌 시작기준 뒤쪽만 하는것도 구상

        - 소스코드 : Passive_CrossValidation/수동교차검증_gcc4_3gram, 10gram, 20gram, 30gram
        - 소스코드 : Sub_Chunk_Test (전체 데이터 중 일부 1000byte씩 뽑아서 실험)
        - 소스코드 : NByte_gram_Test (다양한 gcc, 최적화 옵션 따라서 10byte 실험 진행)

### 5. N Byte gram Solution (20.07.14)
    - 진행중
    - 
        - 소스코드 : RNN_Input_Ngram
        - 소스코드 : RNN_Input_Ngram_데이터비율 조정
