>[!question]
>GQ1. 사용자 프로필, 설정, 사용 기록 등 서로 다른 성격의 데이터를 통합 관리해야 할 때, 서비스별 데이터 라이프사이클(예 - 로그는 30일 보관, 설정은 영구 보관)을 고려해 테이블(혹은 컬렉션)을 어떻게 분리·관리하는 것이 효율적일까?

## Description
-효율성 측면에서 다음의 세 가지 측면을 고려하는게 중요. 

- 보존 기간 : 얼마만에 삭제해야 하는 데이터인지 
- 변경/읽기 빈도 : 자주 바뀌는 데이터인지, 읽기 전용인지
- 접근 목적 : 기능 중심, 혹은 분석 목적 등 

## 코드 예시(시스템별)
- Firebase FireStore

| 데이터 종류 | 컬렉션 예시                    | TTL 관리            | 용도                   |
| ------ | ------------------------- | ----------------- | -------------------- |
| 프로필    | `users/{userId}/profile`  | 없음 (영구)           | 최초 가입 이후 거의 변경 없음    |
| 설정     | `users/{userId}/settings` | 없음 (영구)           | 사용자 맞춤 설정            |
| 로그     | `users/{userId}/logs`     | Cloud Function 사용 | 30일 후 삭제 필요 (보안/분석용) |
|        |                           |                   |                      |
- SwiftData 기준 
```
@Model
class UserProfile {
    @Attribute(.unique) var userId: String
    var name: String
    var email: String
    var joinedAt: Date
}

@Model
class UserSettings {
    @Attribute(.unique) var userId: String
    var prefersDarkMode: Bool
    var language: String
}

@Model
class UserLog {
    var userId: String
    var action: String
    var createdAt: Date
}

```

- SQL 기준
```
CREATE TABLE user_profile (
  user_id VARCHAR(255) PRIMARY KEY,
  name VARCHAR(255),
  email VARCHAR(255),
  joined_at DATETIME
);

CREATE TABLE user_settings (
  user_id VARCHAR(255),
  prefers_dark_mode BOOLEAN,
  language VARCHAR(50),
  PRIMARY KEY (user_id)
);

CREATE TABLE user_logs (
  log_id BIGINT AUTO_INCREMENT PRIMARY KEY,
  user_id VARCHAR(255),
  action VARCHAR(255),
  created_at DATETIME
);

```
파티셔닝...도 할 수도 ... 근데 필요할까? 

어쨌든 이건 하나의 예시고, 위 기준을 토대로 데이터 모델링 단계에서 고민이 필요. 

## Keywords
+ [[데이터 모델링 (Data Modeling)]]

## References
- Firebase Firestore 구조 설계
- [SwiftData – Apple Developer](https://developer.apple.com/documentation/swiftdata)
- [MySQL Partitioning](https://dev.mysql.com/doc/refman/8.0/en/partitioning-overview.html)

## 작성자
#Zhen 