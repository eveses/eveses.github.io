---
title: "Load Balancer 비교 & Route 53"
date: 2025-12-13
categories: [aws-saa]
tags: [AWS, ALB, NLB, GLB, DNS, Route53]
---

## ALB - Application Load Balancer

> 7계층에서 동작하고, 트래픽의 내용을 이해하고 의사 결정하는 로드밸런서

- HTTP/S, gRPC 프로토콜 지원
- 지능형 라우팅, 경로/호스트 기반, 헤더/파라미터 기반 라우팅
- AWS WAF와 통합 가능
- 대상그룹: EC2, Lambda, IP, ECS 등
- Sticky Session 지원
- 리다이렉션 (HTTP -> HTTPS) 지원
- X-Forwarded-For 헤더를 통해 실제 IP 확인

## NLB - Network Load Balancer

> 4계층에서 동작하고, 패킷 내용을 확인하지 않고 IP 프로토콜 데이터 기반으로 결정하는 로드밸런서

- TCP, UDP, TLS 지원
- 초당 수백만건의 요청 처리, 매우 낮은 지연 시간
- Elastic IP를 가질 수 있음
- AWS WAF와 통합 불가
- 소스 IP 보존

## GLB - Gateway Load Balancer

> 3, 4 계층에서 동작하고 타 보안장비와 결합해야 할 때 사용하는 로드밸런서

- GENEVE 프로토콜 지원 (IP 패킷 전체), port 6081
- 동작 방식: 트래픽을 GLB가 가로챔 - GLB 뒤의 타사 장비로 트래픽 전송 - 타사 장비에서 트래픽 확인 후 다시 GLB로 전송 - GLB에서 최종 목적지로 전송
- 보안 장비의 확장성, 가용성에 용이
- 타사 보안장비를 통해 모든 트래픽을 검사할때 사용

## ALB vs NLB vs GLB

| | ALB | NLB | GLB|
|---|---|---|---|
| 계층 | Layer 7 | Layer 4 | Layer 3 + 4 |
| 속도 | 빠름 | 매우 빠름 | 처리 장비에 따라 다름 |
| IP | 동적 IP | 정적 IP (Elastic IP) | IP 리스너 없음 |
| 주요기능 | 경로/호스트 기반 라우팅 | 고성능, TCP/UDP, 메우 낮은 지연 시간 | 트래픽 미러링, 보안 검사 |
| WAF통합 | 가능 | 불가능 | 불가능 |
| 사용처 | 웹, 마이크로서비스 | 게이밍, 금융거래, 실시간 | IDS/IPS, Firewall |

> **키워드 별 구분**  
>
> - 경로 분기, HTTP 헤더, 마이크로서비스, AWS WAF 통합 -> **ALB**  
> - 수백만 트래픽, IP 고정, TCP/UDP, 최소 지연시간 -> **NLB**  
> - 모든 트래픽 검사, 타사 보안장비 -> **GLB**  

## Route 53

> 가용성과 확장성이 뛰어난 클라우드 DNS 서비스 (도메인 등록, DNS 라우팅, 헬스체크 기능)

<h3># Record Types</h3>

- A - maps a hostname to IPv4
- AAAA - maps a hostname to IPv6
- CNAME - maps a host name to another hostname, target domain must have A or AAAA
- NS - Name Servers for the hosted zone
- SOA - 호스트 영역 생성 시 자동으로 생성되는 레코드 (관리자 정보 포함)
- MX - Mail Exchange, for Email Server routing

<h3># Public vs Private Hosted Zone</h3>

- Public Hosted Zone : 전 세게 누구나 조회 가능
- Private Hosted Zone : 내 VPC 내부에서만 조회 가능 (사내/내부 시스템용)

## CNAME vs Alias

<h3># CNAME</h3>

> 표준 DNS 기능, 도메인과 다른 도메인 연결 (루트 도메인은 불가), DNS 조회 2번 발생

<h3># Alias</h3>

> AWS 전용 기능, AWS 리소스 IP를 바로 알려줌, 루트 도메인 가능  
AWS 리소스로 연결 시 쿼리 비용 무료, 속도 더 빠르고 DNS 조회 1번

| | CNAME | Alias |
|---|---|---|
| 대상 | 다른 도메인 | AWS 리소스 |
| 쿼리 비용 | 발생 | 무료 |
| Zone Apex | 불가 | 가능 |
| 속도 | 느림 | 빠름 |
| 사용처 | 서브 도메인 매핑 | 루트 도메인 매핑 |

## Routing Policies

<h3># Simple Routing</h3>

> 하나의 도메인에 여러 IP 지정, 요청 시 IP 랜덤 반환, 헬스체크 X

<h3># Weighted Routing</h3>

> 트래픽을 지정된 비율로 분산

<h3># Latency Routing</h3>

> 가장 짧은 지연시간의 리전으로 연결

<h3># Failover Routing</h3>

> 헬스체크를 통해 Primary가 정상일 경우 전송, 비정상일 경우 Secondary로 전송

<h3># Geolocation Routing</h3>

> 사용자의 실제 물리적 위치 기준 라우팅

<h3># Geoproximity Routing</h3>

> 사용자의 위치와 리소스의 위치 기반, Bias(편향) 을 통해 확장/축소

<h3># Multi Value Answer Routing</h3>

> Simple Routing과 유사, 헬스체크 가능, 최대 8개의 IP 랜덤 반환  
ELB 없이 DNS 수준에서 간단한 로드밸런싱
