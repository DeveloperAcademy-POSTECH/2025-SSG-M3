
>[!question]
>GQ1. HealthKit에서 일반적으로 사용하는 걸음 수, 심박수 말고, 다른 지표(ex. 눈 관련)도 제공할까? 


## Description
- HealthKit은 iOS·watchOS 기기에서 사용자 건강 데이터를 중앙 저장소에 안전하게 관리하도록 돕는 프레임워크입니다. 그중 **안구 압력(intraocularPressure)** 같은 눈 건강 관련 지표를 포함해, 심박수·혈당 등 다양한 바이오메트릭 데이터를 읽고 쓸 수 있습니다.

## 주요 기능
1. **권한 요청 & 확인**
    - `HKHealthStore.requestAuthorization`으로 눈 건강 관련 데이터 타입(예: `HKQuantityTypeIdentifier.intraocularPressure)` 및 기타 헬스 지표(혈당·심박수 등) 읽기 권한을 요청합니다.
        
    
2. **샘플 조회 (HKSampleQuery)**
    - 기간(past 24h, 특정 날짜 등)을 지정해 안압 데이터 샘플을 불러옵니다.
    - 샘플별 시작·종료 시간, 측정 값 등을 순차적으로 처리할 수 있습니다.
        
    
3. **통계 집계 (HKStatisticsQuery)**
    - 일별·주별 평균, 최댓값·최솟값, 합계 같은 집계 통계를 계산합니다.
    - 옵션(`.discreteAverage`, .`discreteMinMax`, `.cumulativeSum` 등)을 조합해 다양한 통계 지표를 얻을 수 있습니다.
        
    
4. **실시간 업데이트 (`HKObserverQuery` + `Background Delivery`)**
    - 새로운 안압 데이터가 기록되면 앱에 콜백이 발생하도록 옵저버를 등록합니다.
    - 백그라운드에서도 수집된 데이터를 즉시 처리할 수 있도록 `enableBackgroundDelivery`를 활성화합니다.
        
    
5. **다중 지표 연동**
    - 안압 외에 혈당(`.bloodGlucose`), 심박수(`.heartRate`) 등 연관 데이터와 함께 사용해 포괄적인 눈 건강 모니터링 워크플로우를 구성할 수 있습니다.

## 코드 예시

```Swift
import HealthKit

let healthStore = HKHealthStore()

// 1. 권한 요청
func requestEyeHealthAuthorization(completion: @escaping (Bool, Error?) -> Void) {
    guard HKHealthStore.isHealthDataAvailable() else {
        completion(false, nil)
        return
    }
    let eyePressureType = HKObjectType.quantityType(forIdentifier: .intraocularPressure)!
    let otherTypes: Set<HKSampleType> = [
        HKObjectType.quantityType(forIdentifier: .bloodGlucose)!,
        HKObjectType.quantityType(forIdentifier: .heartRate)!
    ]
    healthStore.requestAuthorization(toShare: [], read: [eyePressureType] + otherTypes) { success, error in
        completion(success, error)
    }
}

// 2. 안압 샘플 조회 (지난 24시간)
func fetchLast24hIntraocularPressure(completion: @escaping ([HKQuantitySample]?, Error?) -> Void) {
    let type = HKQuantityType.quantityType(forIdentifier: .intraocularPressure)!
    let end = Date()
    let start = Calendar.current.date(byAdding: .day, value: -1, to: end)!
    let predicate = HKQuery.predicateForSamples(withStart: start, end: end, options: .strictStartDate)

    let query = HKSampleQuery(sampleType: type,
                              predicate: predicate,
                              limit: HKObjectQueryNoLimit,
                              sortDescriptors: [NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: true)]) { _, samples, error in
        completion(samples as? [HKQuantitySample], error)
    }
    healthStore.execute(query)
}

// 3. 일별 평균 안압 계산
func fetchDailyAveragePressure(on date: Date, completion: @escaping (Double?, Error?) -> Void) {
    let type = HKQuantityType.quantityType(forIdentifier: .intraocularPressure)!
    let start = Calendar.current.startOfDay(for: date)
    let end = Calendar.current.date(byAdding: .day, value: 1, to: start)!
    let predicate = HKQuery.predicateForSamples(withStart: start, end: end, options: .strictStartDate)

    let statsQuery = HKStatisticsQuery(quantityType: type,
                                        quantitySamplePredicate: predicate,
                                        options: .discreteAverage) { _, result, error in
        guard let avg = result?.averageQuantity() else {
            completion(nil, error)
            return
        }
        completion(avg.doubleValue(for: HKUnit(from: "kPa")), nil)
    }
    healthStore.execute(statsQuery)
}

// 4. 실시간 신규 안압 감지
func observePressureChanges() {
    let type = HKObjectType.quantityType(forIdentifier: .intraocularPressure)!
    let observer = HKObserverQuery(sampleType: type, predicate: nil) { _, _, error in
        if error == nil {
            // 새 데이터 등록 시 처리 로직
        }
    }
    healthStore.execute(observer)
    healthStore.enableBackgroundDelivery(for: type, frequency: .immediate) { _, _ in }
}
```


## Keywords
+ [[Healthkit]]
+ [[AppleFrameworks - HealthKit]]

## References
- Apple Developer Documentation: [HealthKit Framework](https://developer.apple.com/documentation/healthkit)
- Apple Developer Documentation: [HKQuantityTypeIdentifier.intraocularPressure](https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/1615729-intraocularpressure)
- WWDC 2016 “Advances in HealthKit”
- WWDC 2018 “Building a Great HealthKit Experience”

## 작성자
- #Zhen 