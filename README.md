# 오늘 탄다: 최저가 여행을 원하는 사용자를 위한 특가 항공권 예매 서비스
![7조_프로젝트커버](https://github.com/user-attachments/assets/248240e6-90d3-4f68-a767-8ea168e0a860)

## ✈️ 프로젝트 소개
오늘 탄다 프로젝트는 특가 및 일반 항공권을 사용자에게 제공하고, 실시간 항공권 조회 및 예약 기능을 통해 편리하고 신뢰할 수 있는 항공권 구매 경험을 제공하는 서비스입니다.
사용자는 원하는 조건의 항공권을 검색하고, 실시간으로 예약 및 결제를 진행할 수 있습니다.
### 📍 프로젝트 목적
1. **서비스 구조 및 아키텍처**
    - MSA 기반의 유연하고 확장 가능한 서비스 아키텍처 설계 및 개발
    - 서비스 간 독립성과 분산 처리를 고려한 도메인 분리
    - 비동기 메시징을 활용한 이벤트 기반 아키텍처 적용
2. **서비스 안정성 확보**
    - 대용량 트래픽 환경에서도 안정적인 서비스 제공
    - 장애 감지 및 대응을 위한 모니터링 시스템 구축
    - 예약 중 장애나 오류 발생 시 복구 및 재처리 로직 적용
3. **성능 및 사용자 경험 최적화**
    - 실시간 특가 항공편 조회와 예약 기능의 응답 속도 개선
    - 대기열 시스템과 동시성 제어를 통한 중복 예약 방지 및 순차 처리
    - 높은 처리량을 유지하면서도 사용자 경험을 해치지 않는 설계

<br>

## 🧑‍🧑‍🧒‍🧒 Team
| **담당자**           | **역할**       | **담당 업무**                                             |
|------------------|----------------|-------------------------------------------------|
| 진강훈 | BE       | 대기열 서비스, 결제 서비스 설계 및 구현 |
| 김승수 | BE       |      사용자 인증/인가 서비스 설계 및 구현, CI/CD PipeLine 구축 및 배포            |
| 서진영 | BE       |        예약 서비스 설계 및 구현, 모니터링툴 구축 및 시각화           |
| 오연주 | BE       |     항공 서비스 설계 및 구현           |

<br>

## ▶️ Architecture
<img width="750" alt="image (7)" src="https://github.com/user-attachments/assets/82353c1e-af2b-43e6-b50a-de94097b2303" />

<br>

## 📄 API 명세서
API 명세서 ☞ [🔗Link](https://www.notion.so/teamsparta/API-1cb2dc3ef5148015a607f0b2d76c6962)

<br>

## 📚 ERD
![항공권 예매 서비스 (4)](https://github.com/user-attachments/assets/083d6c62-7d02-4c73-8dd6-9e89a4fc2fe1)

<br>

## ⚙️ 주요 기능
- 사용자 관리
    - Redis를 사용하여 토큰 관리
      → 사용자의 권한 상태 변경시 토큰을 블랙리스트에 등록하거나 만료 처리하기 위해 토큰 버전 정보 추가
    - 토큰 버전 변경시 Kafka를 통해 Gateway에 비동기 전파
      → GateWay는 수신한 이벤트 기반으로 해당 토큰을 즉시 만료 처리
- 실시간 항공편 조회
    - Amadeus 외부 API 연동을 통해 실시간 항공편 정보 조회 기능
    - 검색 날짜 기준 최저가 순으로 정렬
    - Redis cache를 통한 항공편 조회 성능 최적화
    - Redisson을 통한 분산 락 적용으로 좌석 수에 대한 동시성 제어 및 데이터 정합성 보장
- 대기열의 생성 및 관리
    - 좌석 선점형 대기열 구조 → 좌석 선택하여 예약 진행 중에 다른 사용자는 동일 좌석에 대한 예약 불가
    - Redis SortedSet을 통한 대기열 생성으로 순차 예약 처리 보장
    - 대기열 동시 진입 시 Redisson을 통한 분산 락 적용으로 최상단 사용자의 순서 보장
- 임시 예약 및 예약 생성
    - 대기열 진입 성공시 Kafka를 통해 예약 생성 비동기 처리
    - Redis에 임시 예약 정보 생성 및 TTL 지정 → 기간내 탑승객 정보 입력 및 결제 완료 시 예약 확정
- 결제
    - 포트원 PG 대행사 연동을 통한 결제 시스템
    - 예약 생성시 결제 요청 → 결제 승인 처리, 예약 취소시 → 결제 취소 처리

<br>

## ❗️ Trouble Shooting
- 대기열 선점 중 과도한 항공편 조회 발생 ☞ [🔗Link](https://github.com/homeProtector/oneul-tanda/wiki/1.-대기열-선점-중-과도한-항공편-조회-발생-및-캐시를-통한-DB-부하-완화)

- 사용자 서비스 개발 과정에서 과도한 의존성 ☞ [🔗Link](https://github.com/homeProtector/oneul-tanda/wiki/2.-사용자-서비스-개발과정에서-dsm을-통한-의존성-체크-후-리팩토링-진행)

- MySQL 정적 데이터 삽입시 타입 불일치 오류 ☞ [🔗Link](https://github.com/homeProtector/oneul-tanda/wiki/3.-MySQL-정적-데이터-삽입시-타입-불일치-오류)

- Redis Cache 역직렬화 과정 오류 ☞ [🔗Link](https://github.com/homeProtector/oneul-tanda/wiki/4.-Redis-Cache-역직렬화-과정-오류)

- Kafka consumer 무한 재시도 이슈 ☞ [🔗Link](https://github.com/homeProtector/oneul-tanda/wiki/5.-Kafka-컨슈머-무한-재시도-이슈)

<br>

## 🛠️ Technologies & Tools
<div align=left>
    <img src="https://img.shields.io/badge/Java 17-%23ED8B00?style=Rectangle&logo=openjdk&logoColor=white"/>
    <img src="https://img.shields.io/badge/Gradle-02303A.svg?style=Rectangle&logo=Gradle&logoColor=white"/>
    <img src="https://img.shields.io/badge/Spring Boot-%236DB33F?style=Rectangle&logo=SpringBoot&logoColor=white"/> 
    <img src="https://img.shields.io/badge/Spring Data JPA-%236DB33F.svg?style=Rectangle&logo=spring&logoColor=white"/>
    <img src="https://img.shields.io/badge/QueryDSL-025E8C?style=Rectangle&logo=Spring&logoColor=white"/>
    <img src="https://img.shields.io/badge/Spring Colud-%236DB33F?style=Rectangle&logo=Spring&logoColor=white"/>
    <img src="https://img.shields.io/badge/Spring Security-%236DB33F.svg?style=Rectangle&logo=springsecurity&logoColor=white"/>
    <img src="https://img.shields.io/badge/JWT-black?style=Rectangle&logo=JSON%20web%20tokens"/>
</div>
<br>
<div align=left>
    <img src="https://img.shields.io/badge/Apache%20Kafka-000?style=Rectangle&logo=apachekafka"/> 
    <img src="https://img.shields.io/badge/Postgres-%23316192.svg?style=Rectangle&logo=postgresql&logoColor=white"/> 
    <img src="https://img.shields.io/badge/Redis-%23DD0031.svg?style=Rectangle&logo=redis&logoColor=white"/> 
    <img src="https://img.shields.io/badge/Redisson-%23DD0031.svg?style=Rectangle&logo=redis&logoColor=white"/> 
    <img src="https://img.shields.io/badge/Docker-%230db7ed.svg?style=Rectangle&logo=docker&logoColor=white"/>
    <img src="https://img.shields.io/badge/AWS EC2-%23FF9900.svg?style=Rectangle&logo=aws&logoColor=white"/> 
    <img src="https://img.shields.io/badge/GitHub%20Actions-%232671E5.svg?style=fRectangle&logo=githubactions&logoColor=white"/>
</div>
<br>
<div align=left>
    <img src="https://img.shields.io/badge/Git-%23F05033.svg?style=Rectangle&logo=git&logoColor=white"/>
    <img src="https://img.shields.io/badge/GitHub-%23121011.svg?style=Rectangle&logo=github&logoColor=white"/>
    <img src="https://img.shields.io/badge/Slack-4A154B??style=Rectangle&logo=slack&logoColor=white"/>
    <img src="https://img.shields.io/badge/Grafana-%23F46800.svg?style=Rectangle&logo=grafana&logoColor=white"/> 
    <img src="https://img.shields.io/badge/Prometheus-E6522C?style=Rectangle&logo=prometheus&logoColor=white"/>
    <img src="https://img.shields.io/badge/k6-7D64FF?style=Rectangle&logo=k6&logoColor=white"/>
    <img src="https://img.shields.io/badge/JMeter-D22128?style=Rectangle&logo=apachejmeter&logoColor=white"/>    
</div>

