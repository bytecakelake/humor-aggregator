# 시스템 아키텍처 (세션 1)

## 전체 파이프라인

```
          ┌───────────────────────┐
          │   Collectors (소스별)   │   확정된 유머 출처마다 1개 구현체
          │  HumorAggregator.      │   (세션 2에서 소스 확정, 세션 4에서 구현)
          │  Collectors            │
          └───────────┬────────────┘
                      │  HumorItem (원본)
                      ▼
          ┌───────────────────────┐
          │   Dedup (중복 제거)      │   텍스트/이미지 유사도 + 해싱
          │  HumorAggregator.      │   (세션 5)
          │  Dedup                 │
          └───────────┬────────────┘
                      │  HumorItem (중복 제거됨)
                      ▼
          ┌───────────────────────┐
          │  Rule Engine (CEL)     │   카페별 게시 규칙(선정 기준,
          │  HumorAggregator.      │   가공 방식, 게시 주기 등)을
          │  RuleEngine            │   CEL 표현식으로 평가
          └───────────┬────────────┘   (세션 6)
                      │  게시 대상 + 가공된 콘텐츠
                      ▼
          ┌───────────────────────┐
          │  Publishers (사이트별)   │   현재: 네이버 카페 Open API
          │  HumorAggregator.      │   (POST .../cafe/{clubid}/menu/
          │  Publishers            │    {menuid}/articles, OAuth2)
          └───────────┬────────────┘   (세션 7)
                      │
                      ▼
               네이버 카페 게시 완료

  ┌─────────────────────────────────────────────┐
  │  HumorAggregator.Core                        │
  │  공통 모델(HumorItem 등), 인터페이스 정의        │
  │  (ICollector, IDedupService, IRuleEngine,     │
  │   IPublisher) — 모든 레이어가 참조               │
  └─────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────┐
  │  HumorAggregator.Host (Worker Service)       │
  │  오케스트레이션, 스케줄러, 로깅/알림 (세션 8)     │
  │  Core/Collectors/Dedup/RuleEngine/Publishers  │
  │  전체 참조                                     │
  └─────────────────────────────────────────────┘
```

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
