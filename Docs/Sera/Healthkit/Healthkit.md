
사실 Healthkit은 그냥 데이터 저장하고 가져오는 것 뿐...입니다... 만 좀 더 설명을 해보자면..
- 데이터 공유 생태계: 사용자의 허락만 있다면, 다른 앱이나 Apple Watch를 통해 이미 HealthKit에 저장된 데이터를 가져와 앱의 기능을 풍부하게 할 수 있음
- 백그라운드 데이터 처리: 앱이 꺼져있는 동안에도 Apple Watch에서 기록된 운동 데이터를 HealthKit을 통해 자동으로 동기화가능
- 표준화된 데이터 형식: 개발자들은 HealthKit 프레임워크(API)를 통해 걸음 수, 혈당, 수면 단계 등 100가지가 넘는 표준화된 데이터 유형을 쉽게 읽거나 쓸 수 있음

### Healthkit 구성요소

|구성 요소|설명|예시|
|---|---|---|
|`HKHealthStore`|HealthKit의 중심 객체. 데이터 요청/저장 등을 이걸로 함.|`let healthStore = HKHealthStore()`|
|`HKObjectType`|모든 Health 데이터의 "타입"을 표현|`stepCount`, `sleepAnalysis`, `heartRate`|
|`HKSample`|실제 하나의 데이터 기록 (값 + 시간 포함)|`HKQuantitySample`, `HKCategorySample`|
|`HKQuantityType`|숫자 데이터 타입|걸음 수, 칼로리, 심박수 등|
|`HKCategoryType`|상태형 데이터 타입|수면, 명상, 생리 등|
|`HKCharacteristicType`|사용자 특징 정보|성별, 생일 등 (읽기 전용)|
|`HKWorkout`|운동 기록 전체 세션 단위|운동 시간, 거리, 칼로리 등|


### CareKit은 Healthkit과 뗄 수 없는 존재.....니까 설명해볼게용 ^.^


**CareKit**은 애플이 만든 오픈소스 프레임워크로, 사용자가 자신의 치료 계획, **건강 루틴**, **증상 기록** 등을 **앱에서 직접 추적**할 수 있게 도와줌→ 병원이 아니라 사용자 본인이 직접 사용하는 앱에 적합함.

다시 말해 손쉽게 환자 관리용 앱을 만들게 도와줌으로써 환자가 평상시에 자신의 건강을 관리하는 것을 도우려고 하는 것. 

HealthKit이 수집한 객관적인 측정치에 환자의 느낌을 더함으로써 환자의 상태를 더 정확하게 이해할 수 있음.

|항목|**HealthKit**|**CareKit**|
|---|---|---|
|**주 목적**|건강 데이터 저장/공유 플랫폼|환자 **자기 관리용 앱 프레임워크**|
|**역할**|건강 데이터 센터 (센서, 앱, 병원 데이터 등)|사용자의 **치료 과정 추적**, **자가관리 루틴** 실행|
|**데이터 흐름**|심박수, 걸음 수, 수면 등 **기계적인 측정 데이터** 중심|복약, 운동, 증상, 기분 등 **사용자 루틴 중심**|
|**주 사용 주체**|모든 건강 앱 (헬스, 피트니스, 수면 등)|병원/환자 케어 앱, 복약관리 앱 등|
|**동작 예시**|Apple Watch로 걸음 수 → HealthKit에 기록됨|"오늘 운동 했나요?" → CareKit Task 체크|

CareKit 앱에서 HealthKit을 지원하면, 사용자에게 건강 및 피트니스 데이터에 접근하고 지정된 보호자와 공유할 수 있는 권한을 요청할 수 있음.

### 예시: 환자가 **하루 운동, 복약, 기분**을 관리하는 앱

1. **CareKit에서 관리:**
    - 운동 Task → 사용자 완료 체크
    - 약 복용 Task → 알림 + 체크
    - 기분 기록 Task → 점수/이모지 선택
2. **HealthKit과 연동:**
    - 운동 Task 완료 → `HKWorkout`으로 기록
    - 기분 기록 → `HKCategoryTypeIdentifier.mood`로 기록
    - 걸음 수 자동 체크 → 사용자가 운동 Task를 완료 안 해도 자동 완료 가능


carekit 코드적 부분 구성요소

|구성요소|설명|
|---|---|
|`OCKTask`|오늘 할 일 (예: 약 복용, 운동 등)|
|`OCKEvent`|특정 날짜에 수행된 Task의 이벤트 (체크/미체크 여부 등)|
|`OCKOutcome`|해당 이벤트의 결과값 (복용 성공/실패, 점수 등)|
|`OCKStore`|모든 데이터를 저장하는 중앙 관리 객체|
|`OCKDailyPageViewController`|날짜별로 Task를 표시해주는 기본 UI|
## 작성자
- 작성한 사람의 닉네임 (예: # Air )