# humor-aggregator 프로젝트 로드맵

> 여러 출처의 유머 콘텐츠를 수집하여 중복 없이 다양한 플랫폼(현재는 네이버 카페)에 자동 게시하는 프로그램.
> 카페별로 복잡한 게시 규칙이 존재할 수 있으므로, 규칙을 스크립팅으로 정의할 수 있는 엔진을 포함한다.

## 기술 스택

- 언어/런타임: .NET (LTS 버전)
- 배포 환경: Oracle Cloud Infrastructure (OCI)
- IaC: Terraform
- 셀프호스팅: Docker Compose
- 형상관리: GitHub
- 라이선스: Apache-2.0

## 세션 로드맵

작업은 아래 9개 세션 단위로 진행한다. 각 세션은 독립된 작업 흐름이며, 순서대로 수행한다.

| # | 세션 | 범위 | 핵심 산출물 | 상태 |
|---|------|------|------------|------|
| 1 | 기획/아키텍처 | 요구사항 확정, 시스템 다이어그램, 핵심 설계 결정(규칙 스크립팅 방식, 네이버 게시 방식), 솔루션 구조, 리포지토리 초기화 | 설계 문서, GitHub repo, .NET solution skeleton | 완료 |
| 2 | 유머 출처 리서치 | 후보 소스 발굴·평가(품질, 갱신 빈도, 수집 난이도, 법적/정책 리스크), 최종 소스 리스트 확정 | 출처 평가 보고서 + 최종 소스 리스트 | 예정 |
| 3 | 데이터 모델 & 저장소 | 유머 아이템 스키마, DB 선택, 마이그레이션 | DB 스키마, EF Core 모델 | 예정 |
| 4 | 수집(Collector) 모듈 | 확정된 소스별 크롤러/스크래퍼 구현 | Collector 플러그인 구조 + 구현체 | 예정 |
| 5 | 중복 제거(Dedup) 모듈 | 텍스트/이미지 유사도 판별, 해싱 전략 | Dedup 서비스 + 테스트 | 예정 |
| 6 | 규칙 스크립팅 엔진 | 카페별 게시 규칙을 스크립트로 정의할 수 있는 엔진 설계·구현 | 스크립팅 엔진 + 예제 규칙 | 예정 |
| 7 | 네이버 카페 Publisher | 로그인/세션 관리, 게시 자동화, 규칙 엔진 연동 | Publisher 모듈 | 예정 |
| 8 | 오케스트레이션 & 운영 | 파이프라인 연결(수집→중복제거→규칙적용→게시), 스케줄러, 에러처리/로깅/알림 | 전체 파이프라인 실행 가능 상태 | 예정 |
| 9 | 인프라/배포 | Terraform(OCI 리소스), Docker Compose(셀프호스팅), GitHub Actions CI/CD | 배포 가능한 운영 환경 | 예정 |

## 세션 1 핵심 설계 결정 (확정)

1. **규칙 스크립팅 방식: Google CEL (Common Expression Language)**
   - 후보 라이브러리: `bsidio/celly` (순수 C#, 외부 의존성 없음, 공식 CEL conformance suite 100% 통과) 우선 검토, 대안으로 `Cel.NET`(CEL-Java 포트, Protobuf 의존)
   - 선언적 규칙과 표현식 유연성을 동시에 확보하는 절충안
   - 세션 6에서 실제 라이브러리 선정 및 PoC로 재검증
2. **네이버 카페 게시 방식: 공식 Open API**
   - `POST https://openapi.naver.com/v1/cafe/{clubid}/menu/{menuid}/articles` (OAuth 2.0 인증)
   - 브라우저 자동화(Playwright 등) 불필요 → Docker 셀프호스팅 구조 경량화
   - **리스크(세션 7에서 재검증 필수)**: (a) 상품게시판은 API 글쓰기 불가, (b) 비소유/비운영 카페에 대한 접근 권한 범위 사전 확인 필요, (c) Rate limit 확인

## 세션 2 평가 기준 (유머 출처 리서치)

- 콘텐츠 품질: 자극적/저품질 콘텐츠 비율, 편집·모더레이션 수준
- 갱신 빈도 및 양: 새 콘텐츠 유입 속도, 소스 간 콘텐츠 중첩도
- 수집 난이도: HTML 구조 안정성, 공식 API 유무, 로그인/캡챠 등 방어 수준
- 법적/정책 리스크: robots.txt·이용약관상 크롤링 허용 여부, 재게시 시 출처 표기 의무

## 세션 1 산출물

- 시스템 아키텍처 다이어그램: [`docs/architecture.md`](./architecture.md)
- .NET 10 solution 스켈레톤: `HumorAggregator.slnx` (Core/Collectors/Dedup/RuleEngine/Publishers/Host/Tests 7개 프로젝트, 빌드 성공 확인)
- GitHub repo: `humor-aggregator` (public)

## 변경 이력

- 2026-11-13: 최초 작성, 9단계 로드맵 승인
- 2026-11-13: 세션 1 핵심 설계 결정 확정 — 규칙 스크립팅은 CEL, 네이버 게시는 공식 Open API(브라우저 자동화 대신)
- 2026-11-13: 세션 1 마무리 — 아키텍처 다이어그램 작성, .NET 10 솔루션 스켈레톤 생성, GitHub repo(public) 초기화
