# 🏗️ Jikgong Backend 프로젝트 가이드

## 📋 프로젝트 개요

**Jikgong(직공)**은 건설 노동자와 기업들의 일자리 매칭을 시켜주는 중개플랫폼의 백엔드 서비스입니다.

---

## 🏗️ 프로젝트 아키텍처

### 전체 아키텍처
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   Database      │
│   (React)       │◄──►│   (Spring Boot) │◄──►│   (MySQL)       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   External      │
                       │   Services      │
                       │   (S3, FCM,     │
                       │    AlimTalk)    │
                       └─────────────────┘
```

### 레이어드 아키텍처
```
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                       │
│                    (Controller)                            │
├─────────────────────────────────────────────────────────────┤
│                     Business Layer                          │
│                     (Service)                              │
├─────────────────────────────────────────────────────────────┤
│                     Data Access Layer                       │
│                     (Repository)                           │
├─────────────────────────────────────────────────────────────┤
│                     Database Layer                          │
│                     (MySQL, Redis)                         │
└─────────────────────────────────────────────────────────────┘
```
---

## 📁 프로젝트 구조

### 전체 디렉토리 구조
```
src/main/java/jikgong/
├── domain/                    # 도메인별 비즈니스 로직
│   ├── apply/
│   ├── common/
│   ├── history/
│   ├── jobpost/
│   ├── like/
│   ├── location/
│   ├── member/
│   ├── notification/
│   ├── offer/
│   ├── profit/
│   ├── project/
│   ├── resume/
│   ├── scrap/
│   ├── skill/
│   ├── workdate/
│   └── workexperience/
├── global/                    # 전역 설정 및 공통 기능
│   ├── alimtalk/
│   ├── annotation/
│   ├── aop/
│   ├── batch/
│   ├── codef/
│   ├── common/
│   ├── config/
│   ├── event/
│   ├── exception/
│   ├── fcm/
│   ├── feignclient/
│   ├── querycount/
│   ├── s3/
│   ├── security/
│   ├── slack/
│   ├── sms/
│   └── utils/
└── JikgongApplication.java   # 메인 애플리케이션 클래스
```

### 각 도메인별 구조
각 도메인은 다음과 같은 구조를 가집니다:
```
domain/{도메인명}/
├── controller/    # REST API 엔드포인트
├── service/       # 비즈니스 로직
├── repository/    # 데이터 접근 계층
├── entity/        # JPA 엔티티
└── dto/          # 데이터 전송 객체
```
---

## 📝 코딩 컨벤션

### Java 코딩 컨벤션
- **클래스명**: PascalCase (예: `MemberService`)
- **메서드명**: camelCase (예: `getMemberById`)
- **변수명**: camelCase (예: `memberId`)
- **상수명**: UPPER_SNAKE_CASE (예: `MAX_RETRY_COUNT`)
- **패키지명**: 소문자 (예: `jikgong.domain.member`)

### 파일 구조 컨벤션
```
Controller: {도메인명}Controller.java
Service: {도메인명}Service.java
Repository: {도메인명}Repository.java
Entity: {도메인명}.java
DTO: {기능명}Request.java, {기능명}Response.java
```

### 예외 처리 규칙
- 비즈니스 로직 예외: `JikgongException` 사용
- 시스템 예외: Spring 기본 예외 처리
- 예외 코드는 `ErrorCode` enum에 정의

---

## 🔐 보안 설정

### 인증 및 권한
- **JWT 기반 인증**: Stateless 인증 방식
- **Role 기반 권한**: WORKER, COMPANY 역할 구분
- **메서드 레벨 보안**: `@PreAuthorize` 어노테이션 사용

### 보안 설정
```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)
public class SecurityConfig {
    // JWT 필터 설정
    // CORS 설정
}
```

---

## 📊 데이터베이스 설계
DDL 기반으로 ERD 생성 후 확인 필요

### JPA 설정
```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        create_empty_composites:
          enabled: true
```

---

## 🚀 배포 및 운영

### Docker 설정
```yaml
version: "3.8"
services:
  jikgong-a:
    image: ${DOCKERHUB_USERNAME}/jikgong-a
    ports:
      - "8080:8080"
    env_file:
      - .env
```

### 배포 스크립트
- `deploy.sh`: 배포 자동화 스크립트
- `docker-compose.{}.yml`: Docker Compose 설정

### 모니터링
- **Spring Actuator**: 헬스체크 및 메트릭
- **Prometheus**: 메트릭 수집
- **Slack Webhook**: 알림 및 모니터링

---

## 📚 API 문서

### Swagger UI
- URL: `http://localhost:8080/swagger-ui/`

### API 응답 형식
```json
{
  "success": true,
  "data": {
    // 응답 데이터
  },
  "message": "성공적으로 처리되었습니다."
}
```

---

## 🔄 CI/CD

### GitHub Actions
- 자동 빌드 및 테스트
- Docker 이미지 빌드
- 자동 배포

### 배포 파이프라인
1. 코드 커밋 → GitHub
2. GitHub Actions 트리거
3. 빌드 및 테스트 실행
4. Docker 이미지 생성 및 Docker Hub 업로드
5. AWS EC2 배포