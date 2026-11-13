# 시스템 아키텍처 (세션 1)

## 전체 파이프라인

draw.io 소스 다이어그램: [`docs/architecture.drawio`](./architecture.drawio) (draw.io 데스크톱 앱 또는 https://app.diagrams.net 의 "파일 열기"로 확인)

구성 요소: `Collectors → Dedup → RuleEngine(CEL) → Publishers(사이트별 게시 모듈, 현재 대상 네이버 카페 Open API)` 파이프라인이 `Host(Worker Service)` 안에서 순차 실행되며, 모든 레이어가 `Core`(공통 모델·인터페이스)를 참조하고 `Tests`가 Core/Dedup/RuleEngine을 검증한다. Publishers는 특정 사이트에 고정되지 않은 플러그인형 구조로, 향후 네이버 카페 외 사이트 추가 시 이 레이어에 구현체를 추가한다.

## 프로젝트 구조

| 프로젝트 | 역할 | 참조 |
|---|---|---|
| `HumorAggregator.Core` | 공통 모델·인터페이스 | - |
| `HumorAggregator.Collectors` | 소스별 수집기 | Core |
| `HumorAggregator.Dedup` | 중복 제거 서비스 | Core |
| `HumorAggregator.RuleEngine` | CEL 기반 규칙 평가 | Core |
| `HumorAggregator.Publishers` | 사이트별 게시기 (네이버 Open API) | Core |
| `HumorAggregator.Host` | Worker Service, 오케스트레이션 | 전체 |
| `HumorAggregator.Tests` | 단위 테스트 (xUnit) | Core, RuleEngine, Dedup |

## 배포 아키텍처 (세션 9에서 구체화)

- Docker Compose로 `HumorAggregator.Host` 컨테이너 1개 셀프호스팅 (네이버 Open API 채택으로 헤드리스 브라우저 불필요 → 경량 구조)
- Terraform으로 OCI VM/네트워크 프로비저닝
- GitHub Actions로 빌드·테스트·배포 CI/CD

## 변경 이력

- 2026-11-13: 최초 작성 (세션 1)
