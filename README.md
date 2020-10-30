### 국가보안기술연구소(NSR) 연구

## ■ 함수정보 추출 기계학습 기술 연구
### ■ 연구 필요성
- 현재 시중에 다양한 바이너리 분석 도구
  - BAP, BitBlaze, BinNavi, IDA Pro 등
  - 그러나 스트립(stripped)된 바이너리의 분석 정확도 낮음

- 리눅스용 다양한 컴파일러가 존재
  - gcc 7.2, 8.0, 8.3, 9.0 …

- 스트립 바이너리에 대한 분석 어려움
  - 디버그 심볼이 없음 -> 역공학 어려움
  - 컴파일러 종류/버전/최적화 정도 파악 어려움
  - 함수 위치 파악 어려움
  - 스트립 악성 코드 분석을 위한 스트립 바이너리 분석 기술 필요

### ■ 연구 목표
- 기계학습 기반의 스트립(stripped) 리눅스 바이너리 분석 기술
  - 스트립 바이너리에서 함수 위치 탐지 기술
    - 함수 위치정보 (시작, 종료) 추출 기계학습 기반의 모델 연구
    
<hr>

## 진행상황 정리 (20.10.30 완료)

### 0. 사용한 데이터
    - GCC6, 7, 8, 9 컴파일러의 Binutils, Coreutils의 elf 파일 포맷 실행 파일
    - 해당 실행파일을 최적화 O0~O3 에 따라 컴파일 진행 및 16진수 바이트 코드 형태로 추출
    
### 1. 자료조사 및 관련 논문 리뷰 (~ 20.04.15) 
    - 문서 : 논문 : (USENIX)Recognizing functions in binaries with neural networks_augsut 2015 
    - 문서 : document/(20.04.15)김선민 Recognizing Functions in Binaries with Neural Networks 요약본

<hr>

### 2. Bidirectional RNN 구현 (1) 및 Stripped Binary 관련 공부 (~ 20.05.08)
    - 문서 : (20.05.08) Recognizing Functions in Binaries with Neural Networks 구현 (논문 추가 내용 정리)
    - 문서 : (20.05.08) 김선민,장두혁 Bidirectional RNN 구현 및 Stripped Binary 공부
    - 소스코드 : Preprocessing/Preprocessing_Save_hexByteCode.ipynb

<hr>

### 3. BIRNN 구현 (2)  (~ 20.06.05)
    - 문서 : (20.06.05) BIRNN 구현
    - BIRNN 구현 시도 -> Imbalanced Data 문제 발생
    - Imbalanced Data 문제 해결 방법으로 -> 함수 시작 주변 N byte 자르는 방식생각 
    - 소스코드 : RNN_by_All_Binary/BiRNN_gcc5_op0~2_by_All_Binary.ipynb

<hr> 

### 4. 데이터 인사이트
    - Insight (1) 컴파일러별 함수 시작점 주변 바이트들이 일정한 규칙성을 띔
        - ex) gcc6의 경우 함수 시작점 ~ 6개의 byte는 55, 6f, 72 .. 유사한 330쌍
        - ex) gcc7, 8, 9 도 비슷, 단 최적화 진행시 개수가 훨씬 더 늘어남
    - Insight (2) 데이터의 매우 극단적인 Imbalance data 문제
        - ex) 함수라 하면 함수시작점 1개의 byte 와 함수길이-1 만큼의 함수 시작점이 아닌 byte를 가지고 있음
    - Insignt (3) 최적화 옵션별 학습데이터 구성 다르게
        - ex) 함수 최적화 o0~o3별 학습데이터를 구성하지만, 최종 사용모델에서는 합쳐서 사용
    - Insight (4) 데이터 물리적 양
        - ex) 데이터를 바이트로 변환시 전체 파일이 2000만 ~ 3000만 바이트 가까이 됨. -> 물리적인 효율을 얻기 위해 N-Byte를 사용한 이유도 있음 (대략 전체 데이터의 1/3 효율)
        
    - 소스코드 : 함수주변바이트확인/
    
### 5. 불균형 데이터 문제 Imbalanced Data에 대한 솔루션 (~ 20.06.23)
    - 문서 : (20.06.23) Imbalanced Data에 대한 Solution 
    - 기존 Imbalance 해결법인 (오버샘플링, 언더샘플링 등은 RNN의 Sequential한 특징 때문에 적용이 힘듬)
    - Imbalance 해결보다는 완화하는 방식으로 생각 및 물리적 효율성을 위한 함수 시작점 주변 N-Byte 방식 생각**
    - 함수 시작 주변 N byte 자르는 방식 적용 및 다양한 N-Byte 기준 실험 진행
    - 함수의 시작 주변 양쪽 or 함수 시작점이 중간 등 여러가지 경우의 수 고려 (3byte, 10byte, 20byte ... 노가다 실험)
  - ~~소스코드 : Passive_CrossValidation/수동교차검증_gcc4_3gram, 10gram, 20gram, 30gram~~
  - ~~소스코드 : Sub_Chunk_Test (전체 데이터 중 일부 1000byte씩 뽑아서 실험)~~
  - ~~소스코드 : NByte_gram_Test (다양한 gcc, 최적화 옵션 따라서 10byte 실험 진행)~~

<hr> 

### 6. 최종 데이터 구성 전처리 (Preprocesing) (~ 20.07.07)
    - 문서 : (20.07.07)함수시작구분RNN
    - 전처리 1) 16진수 바이트를 One-Hot-Encoding을 작업
    - 전처리 2) 함수 시작 부분 부터 N-Byte 씩 잘라서 사용
    - 전처리 3) 함수 시작 부분 정보 포함 데이터 + 미포함데이터
      - 함수시작정보 포함데이터 + 미포함 데이터 5:5 비율 실험결과
    - 소스코드 : 함수시작정보포함데이터&정보미포함데이터
        
<hr>

### 7. 다양한 하이퍼 파라미터로 실험 진행 (~ 20.07.22)
    - 문서 : (20.07.22)바이너리 대상 컴파일러 및 함수정보 추출 기계학습 기술 연구(장두혁,김선민)
    - 실험결과 파일 : (20.07.22)국보연 실험(binutils).xlsx, (20.07.25)국보연 실험결과(coreutils).xlsx
    - N Byte 방식 다양한 하이퍼 파라미터로 실험 진행 -> 90% 결과 도달
    - N Byte 방식 GCC6 최적화 O0 ~ O3의 실험 진행 및 결과
    - 소스코드 : 함수시작정보포함데이터 & 정보미포함데이터_GCC6 이하 폴더들 
     
<hr> 

### 7. 다양한 하이퍼 파라미터로 실험 진행2 (~ 20.07.29) 
    - 문서 : (20.07.29)국보연 GCC6 결과최종
    - 실험결과 파일 : (20.07.27)국보연 실험결과(binutils&coreutils).xlsx
    - GCC 6 바이트추출시 CODE 부분만 데이터 재수집
    - 학습 데이터 재 추출 후 학습 진행 -> 99% 결과 도달
  - ~~소스코드 : binutils&coreutils_GCC6_최종~~
  - 소스코드 : gcc6_model, gcc7_model, gcc8_model, gcc9_model
      
<hr>

### 8. 최종 결과 exe 프로그램 만들기 ( ~ 09.26 )
    - 실험결과 파일 : prediction 폴더 산하 파일들
    - 현재 실험 결과 모델 저장 및 모델 내장 후 자동 추론 프로그램 만들기
    - binutils 학습, coreutils test 진행 ==> 다소 결과가 떨어지는 모습을 보임.
    - binutils + coreutils 학습 kfold ==> 거의 99%
    - binutils + coreutils 학습 후 동일데이터 prediction ==> 97% 정도
    - 최종적 gcc 상위버전 현재 6으로 진행, 7,8,9 에 대해서 실험 예정중
    
### 9. 지속적 보완 ( 9 ~ 10월 중)
    - 이전 KFold 학습데이터 : 각 실행파일 중 코드섹션의 함수 부분만 학습 -> 전체 섹션에 대한 모든 함수 부분 학습
    - 소스코드 : Prediction2/bin_core_gcc6_op0_Input Sequence 32_hidden units 48x2 ~ o3
    
### 10. 향후 보완점 및 결과 보고서 피드백 내용 (내부피드백 + 담당자피드백)
    - 더 많은 바이너리 확보시 더 일반적인 모델을 만들 수 있음 (기계학습의 기본 원리이기도 함)
    - 모델 실행시 바이너리의 어떠한 실행파일의 어떠한 함수가 주로 오답이였는지, 경향에 대해 알 수 있다면 향후 해당함수에 더 공략법을 세울 수 있을 것임
    - 해당 방식에 대한 하이퍼 파라미터 외에 모델 구조에 대한 실험 비교가 진행됬으면 어떨까 하는
    - 하이퍼 파라미터에 대해 현재는 노가다 였음. 하이퍼 파라미터 최적화에 대한 알고리즘이나 좋은 논문이 있다면 해당 과정에 소요되는 시간을 더 줄일수 있을 것.
    - Window나 arm(모바일 아키텍쳐) 의 경우에는 어떨까 하는 궁금증
    
### 10. 최종 보완 및 연구 종료 (~ 20.10.26)
  - kfold 에서 train (적당한 epoch 학습) + validation 으로 변경
  - 최적 hyperparameter 값 찾기 
  - 최적화 버전(o0 ~ o3) 각각 vs 최적화 통합 버전 비교 -> 최적화 통합 버전에서 조금더 좋은 결과를 보임
