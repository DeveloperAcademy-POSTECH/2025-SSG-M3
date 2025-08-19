
>[!question]
>GQ1.  내부 테스트와 외부 테스트의 차이, 한도(인원·디바이스), 심사 여부는 무엇
>GQ2. 테스터 그룹(Internal / External)을 어떻게 구성하고, 빌드-그룹-테스터를 어떤 순서로 연결
>GQ3. App 이름 / 부제목 / 설명 / 키워드 / 프로모션 텍스트의 역할과 우선순위는?
>GQ4. 프로비저닝 프로파일(Provisioning Profile)이란 무엇이며, 개발·배포 과정에서 어떤 역할?
>GQ5. 개발(Development) 프로비저닝 프로파일과 배포(Distribution) 프로비저닝 프로파일의 차이점은?
>GQ6. Xcode의 “Automatically manage signing” 기능이 프로비저닝 프로파일 생성·갱신을 어떻게 자동화하는가?

꼭 알아야 하는건 아니지만 알아두면 유용할..수 도? 있는? 내용들을 알아보궜습니다. 
## Description
-  **TestFlight**는 배포 전 iOS/iPadOS/macOS/tvOS/watchOS 앱을 내부/외부 테스터에게 **최대 90일** 동안 배포해 피드백을 받게 하는 베타 테스트 도구임. 내부 테스터(팀 구성원)는 심사 없이, **외부 테스터**는 **베타 앱 리뷰** 통과 후 테스트 가능함. 
    
- **App Store Connect(ASC)**는 앱 **메타데이터**(이름·부제·키워드·설명·프로모션 텍스트·스크린샷 등)와 **빌드/테스트/배포**를 관리하는 콘솔임. 메타데이터 품질은 노출과 전환에 직접적인 영향을 줌.

## 주요 기능
## 0. TestFlight 전체 흐름 구조도 
```
[로컬 빌드] 
   ↓ (Archive)
[Xcode Organizer → Distribute]
   ↓
[App Store Connect 업로드]
   ↓
[빌드 처리(Processing)]
   ↓
[ASC: TestFlight 탭]
   ├─ [내부 그룹 생성 + 빌드 연결 + 팀원 추가]  → 내부 테스트 즉시 가능(리뷰 없음)
   └─ [외부 그룹 생성 + 빌드 연결 + 테스터/공개 링크] 
         ↓ (첫 빌드 추가 시 자동 '베타 앱 리뷰')
         [승인되면 외부 테스트 시작]
   ↓
[90일 유효기간 내 테스트 · 피드백 수집]
   ↓
[다음 빌드 업로드(반복)]
```
## **GQ1. 내부 테스트 vs 외부 테스트 — 차이, 한도(인원·디바이스), 심사 여부**
- 내부(Internal): 팀 구성원 대상 베타테스트, 최대 100명. 심사 없이 거의 즉시 배포 가능
- 외부(External): 팀 외부 테스트 사용자 대상. 앱당 최대 10,000명 초대 가능(이메일, 공개 링크), 배포 전 심사 필요할 수 있음. 
- TestFlight 테스터 1명은 최대 30개 기기에서 설치/테스트 가능 (Ad Hoc 배포는 등록 기기 수가 제품군별 100대로 제한되어 있으며 Testflight 와 다른 제도임)

## **GQ2. 테스터 그룹(Internal / External) 구성과 “빌드‑그룹‑테스터” 연결 순서**
- 그룹 생성 (내,외부 각각) > 빌드 추가(그룹에 특정 빌드 연결) > 테스터 추가(내부=팀원, 외부=링크)

## **GQ3. App 이름 / 부제목 / 설명 / 키워드 / 프로모션 텍스트 — 역할 & 우선순위**
- 이름: 최대 30자, 제품 페이지와 설치 시 노출. 중요도 높음
- 부제목(Subtitle): 최대 30자. 제품 페이지 및 검색 결과에 이름 아래 표기. 핵심 가치, 차별점 요약
- 설명(Description): 최대 4,000자. 기능/혜택을 상세히 서술(평문,HTML 불가), 전환이나 설득이 목적
- 키워드(Keywords): 100자 한도, 쉼표로 구분(띄어쓰기 없이). 검색 인덱싱에 직접 사용되는 필드 (사용자에게 비노출)
- 프로모션 텍스트: 최대 170자, 설명 상단에 표시. 앱 업데이트 없이 수정 가능 (세일/이벤트)

- 가이드라인: 이름/부제목/스크린샷 등 메타데이터에 가격, 타사 상표, 검색어 나열 등 스팸 금지. 

- 실무적 우선순위 (ASO(App Store Optimization, 앱스토어 검색,노출 최적화) 관점 
	- 1순위 : 이름 + 부제목 : 검색 가시성과 클릭률에 직접 영향. 핵심 키워드는 중복 없이 자연스럽게 배치 
	- 2순위 : 키워드 
	- 3순위 : 설명 - 검색 가중치 공개되지 않았지만, 전환율에 영향을 미치는 것으로 보임. 첫 두 문단에서 가치, 제안 제시. 

## **GQ4. 프로비저닝 프로파일(Provisioning Profile)이란 무엇이며, 개발·배포 과정에서 어떤 역할?**
- 프로비저닝 프로파일이란? 앱 식별자(App ID/Bundle ID), 인증서, 테스트/배포 대상, 권한/엔타이틀먼트 정보를 하나로 묶어서 "이 빌드가 어떤 기기에서 어떤 권한으로 실행되는겁니다 ~ "를 OS 에 증명하는 서류
- 주요 기능은? 
	- 빌드가 특정 개발자/팀 인증서로 서명되었음 증명
	- TestFlight/App store 는 Apple 배포 채널을 통해서만 설치 가능하게 제안
	- 앱이 사용하는 Capabilities가 엔타이틀먼트와 일치하는지 검증. 불일치하면 심사에서 거절. 

## **GQ5. 개발(Development) 프로비저닝 프로파일과 배포(Distribution) 프로비저닝 프로파일의 차이점은?**

- 개발 프로비저닝? : SwiftUI 프리뷰 말고 실제 기기에서 Run 테스트 할때 사용하는 거! 그 처음 연결했을때 신뢰? 물어보고 빌드하는거 ~ 
- Xcode 에서 확인하는 방법 ! : 프로젝트 > Target > Signing & Capabilities > Team - "Automatically manage signing" on > Status 에 "iOS Team Provisioning Profile:Bundle ID" < 이거가 Development 프로파일.. 

- 차이점 정리

| **구분** | **Development**              | **Distribution**                         |
| ------ | ---------------------------- | ---------------------------------------- |
| 사용 단계  | 로컬 개발·디버그                    | TestFlight / App Store 배포                |
| 대상     | **등록된 디바이스** 필요              | 디바이스 등록 불필요 (Apple 배포 채널)                |
| 인증서    | **iOS Development** 계열       | **Apple Distribution** 계열                |
| 설치 경로  | Xcode Run, Ad Hoc(유사), 내부 배포 | TestFlight(내부/외부 테스트), App Store         |
| 목적     | 디버그·프로파일링·기기 테스트             | 심사·배포·실사용 환경                             |
| 엔타이틀먼트 | 개발 중 켠 Capabilities를 반영      | 배포 시 필요한 Capabilities를 반영(APS, iCloud 등) |

- **Development**: 팀이 등록한 테스트 기기에서만 동작함. Xcode가 기기 등록/프로파일 갱신을 도와줌.
    
- **Distribution**: 디바이스 등록 없이 **TestFlight/App Store**로 설치함. 일반 사용자 대상. 베타/상용 심사 경로와 연결됨.

## **GQ6. Xcode의 “Automatically manage signing” 기능이 프로비저닝 프로파일 생성·갱신을 어떻게 자동화하는가?**

- 뭘 자동화하나? 
- App Id(identifier) 생성/동기화: 프로젝트의 Bundle ID와 일치하는 App ID 를 Apple Developer 포털에 생성하고 연결함 
- Certificate 연동: 배포에 필요한 인증서를 확인하고 필요한 경우 개발자 Mac 의 서명 키체인에 자동 설치, 갱신함 
- Provisioning Profiles 생성/갱신:
	- Development 의 경우, 팀에서 연결된 테스트 디바이스 UDID를 감지해서 프로파일에 포함시키고 자동 갱신함. 
	- Distribution 의 경우, TestFlight/App Store 배포용 프로파일을 생성/적용함
- Entitlement 정합성 유지: 프로젝트에서 켠 Capabilities(Push, iCloud, HealthKit 같은 거) 에 맞게 앱 ID/프로파일의 엔타이틀먼트를 맞추고 불일치시 경고, 수정 유도 
그래서? 안쓰면 이걸 다 직접 해야 한다는 뜻~ 

- 주의: 특수 환경(CI 전용 인증서, 다계정 다팀, 수동관리정책, 서드파티(Xcode 없이 배포하려고? ex. Fastlane) 서명 도구 사용) 에서는 수동 관리가 더 예측 가능하고 적절할수도 있음 ! 초보거나 작은 팀 환경에서 사용하는 것을 권장 ! 
## Keywords
[[TestFlight]]

## References
- https://developer.apple.com/help/app-store-connect/test-a-beta-version/invite-external-testers/
- https://www.apptweak.com/en/aso-blog/app-store-product-page-optimization
- 

## 작성자
- #Zhen 