---
title: "Elastic Load Balancing과 Route 53 설계 비교"
date: 2025-12-13
last_modified_at: 2026-09-05
categories: [aws-saa]
tags: [AWS, ALB, NLB, GWLB, DNS, Route53]
summary: "ALB·NLB·Gateway Load Balancer의 역할과 Route 53 레코드·라우팅 정책을 요구사항별 선택 기준으로 비교합니다."
---

{% include post-intro.html %}

## 로드 밸런서 선택 요약

| 서비스 | 계층·프로토콜 | 강점 | 대표 사용 사례 |
|---|---|---|---|
| ALB | L7, HTTP/HTTPS/gRPC | 요청 내용 기반 라우팅, WAF 통합 | 웹, API, 컨테이너, 마이크로서비스 |
| NLB | L4, TCP/UDP/TLS 등 | 초저지연, 매우 높은 처리량, 고정 IP | 게임, 실시간 처리, 비 HTTP 트래픽 |
| GWLB | L3, GENEVE 6081 | 가상 네트워크 장비의 투명한 삽입과 확장 | 방화벽, IDS/IPS, 심층 패킷 검사 |

## Application Load Balancer

ALB는 HTTP 요청의 호스트, 경로, 헤더, 쿼리 문자열, 메서드 등을 이해해 대상 그룹으로 라우팅한다.

- 대상 유형: EC2 인스턴스, IP 주소, Lambda 함수 등
- 경로 기반(`/api/*`)과 호스트 기반(`admin.example.com`) 라우팅
- HTTP에서 HTTPS로 리디렉션, TLS 종료, 인증서 관리
- AWS WAF와 통합해 L7 공격 규칙 적용
- WebSocket과 gRPC 워크로드 지원
- `X-Forwarded-For`로 원본 클라이언트 IP 전달

ALB 노드는 고정 IP가 아니다. 소비자가 고정 IP를 요구하면 NLB, Global Accelerator 또는 DNS 기반 접근을 검토한다.

## Network Load Balancer

NLB는 연결과 패킷 수준에서 동작해 매우 높은 처리량과 낮은 지연 시간을 제공한다.

- TCP, UDP, TLS 및 지원되는 추가 전송 프로토콜 처리
- 가용 영역별 고정 IP를 제공하며 Elastic IP를 연결할 수 있다.
- 클라이언트 소스 IP를 대상에 보존할 수 있다.
- TLS 리스너에서 인증서와 보안 정책을 중앙 관리할 수 있다.
- AWS WAF는 L7 서비스가 아니므로 NLB에 직접 연결하지 않는다.

고정 IP가 필요하다는 이유만으로 항상 NLB를 고르지는 않는다. 애플리케이션 라우팅과 WAF가 더 중요하다면 ALB 앞에 Global Accelerator를 배치하는 구조도 비교한다.

## Gateway Load Balancer

GWLB는 트래픽 흐름에 방화벽이나 침입 탐지 장비 같은 가상 어플라이언스를 투명하게 삽입한다. Gateway Load Balancer Endpoint와 라우팅 테이블을 이용해 패킷을 검사 계층으로 보내고, GENEVE 프로토콜의 6081 포트로 어플라이언스와 통신한다.

- 보안 장비 플릿의 상태 확인과 수평 확장
- 인바운드, 아웃바운드, VPC 간 트래픽의 중앙 검사
- 원래 패킷의 네트워크 정보를 유지한 서비스 체이닝

GWLB 자체가 방화벽 정책을 제공하는 것은 아니다. 실제 검사는 뒤에 배치한 AWS Marketplace 또는 자체 가상 장비가 수행한다.

## Route 53 기본 구조

Route 53은 권한 있는 DNS, 도메인 등록, 상태 확인과 DNS 기반 라우팅을 제공한다.

### Hosted Zone

- **Public hosted zone**: 인터넷에서 조회 가능한 도메인의 레코드를 관리한다.
- **Private hosted zone**: 연결한 VPC에서만 조회되는 내부 DNS 이름을 관리한다.

### 주요 레코드

| 레코드 | 역할 |
|---|---|
| A / AAAA | 이름을 IPv4 / IPv6 주소에 연결 |
| CNAME | 한 이름을 다른 DNS 이름에 연결 |
| Alias | Route 53이 지원하는 AWS 리소스에 별칭 연결 |
| MX | 메일 서버 지정 |
| NS | 영역의 권한 있는 네임 서버 지정 |
| SOA | DNS 영역의 기본 관리 정보 |

## CNAME과 Alias

| 항목 | CNAME | Alias |
|---|---|---|
| 표준 여부 | 표준 DNS 레코드 | Route 53 확장 기능 |
| Zone apex | 사용할 수 없음 | 사용할 수 있음 |
| 대상 | 다른 DNS 이름 | 지원되는 AWS 리소스 또는 같은 영역 레코드 |
| 대표 대상 | 외부 SaaS 호스트명 | ALB/NLB, CloudFront, S3 웹 사이트 등 |

Alias는 대상 AWS 리소스의 주소 변경을 Route 53이 추적하고, zone apex(`example.com`)에도 사용할 수 있다는 점이 핵심이다. “DNS 조회가 무조건 한 번이라 빠르다”보다 대상 지원 여부와 루트 도메인 요구사항을 기준으로 선택한다.

## 라우팅 정책

- **Simple**: 특별한 분기 없이 하나 이상의 값을 응답한다. 동일 이름의 여러 값을 반환할 수 있지만 개별 레코드 상태 확인과 정교한 분산에는 적합하지 않다.
- **Weighted**: 가중치 비율로 트래픽을 분배한다. 카나리 배포와 점진적 전환에 유용하다.
- **Latency**: 사용자의 DNS 리졸버에서 지연 시간이 가장 낮은 AWS 리전의 리소스를 선택한다.
- **Failover**: 상태 확인 결과에 따라 Primary와 Secondary 사이를 전환한다.
- **Geolocation**: 사용자 위치에 대응하는 규칙을 적용한다. 지역별 콘텐츠·규정 요구에 적합하다.
- **Geoproximity**: 사용자와 리소스의 지리적 거리 및 bias를 이용해 트래픽 영역을 조정한다.
- **IP-based**: 요청을 보낸 리졸버의 CIDR 매핑에 따라 응답한다.
- **Multivalue answer**: 정상으로 판단된 레코드 중 여러 값을 반환하는 간단한 DNS 분산 방식이다.

DNS 장애 조치는 TTL과 클라이언트·리졸버 캐시의 영향을 받으므로 즉시 전환을 보장하지 않는다. 빠른 글로벌 전환과 고정 Anycast IP가 중요하면 AWS Global Accelerator를 비교한다.

## 설계 체크리스트

- 요청 경로·호스트에 따라 분기해야 하는가? → ALB
- 고정 IP, UDP, 매우 낮은 L4 지연이 필요한가? → NLB
- 모든 흐름을 보안 장비로 통과시켜야 하는가? → GWLB
- 단순 DNS 응답이 아니라 상태 기반 장애 조치가 필요한가? → Route 53 health check 연동
- DNS TTL로 인한 전환 지연을 허용할 수 있는가?

## 공식 문서

- [Elastic Load Balancing 동작 방식](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html)
- [Gateway Load Balancer 소개](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/introduction.html)
- [Route 53 라우팅 정책](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)
- [Route 53 Alias 레코드](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-choosing-alias-non-alias.html)
