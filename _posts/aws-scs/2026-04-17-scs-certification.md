---
title: "AWS Certified Security – Specialty 취득 및 SCS-C03 핵심 정리"
date: 2026-04-17
last_modified_at: 2026-09-05
categories: [aws-scs]
tags: [AWS, Security, SCS-C03, Certification]
summary: "AWS Security Specialty 취득 기록과 함께 SCS-C03의 도메인별 출제 비중, 서비스 연결 관계, 실전 판단 기준을 정리합니다."
---

{% include post-intro.html %}

![AWS Certified Security – Specialty 인증 기록](/assets/images/aws/scs/스크린샷%202026-04-17%20오후%2010.46.26.png)

## 시험 구조

SCS-C03은 단순히 보안 서비스 이름을 묻기보다, 주어진 조직·네트워크·규정 조건에서 가장 안전하고 운영 가능한 통제를 선택하는 시험이다. 공식 시험 안내 기준으로 채점 문항 50개와 비채점 문항 15개가 포함되며, 합격 기준 점수는 1,000점 만점 중 750점이다.

| 도메인 | 비중 | 핵심 질문 |
|---|---:|---|
| Detection | 16% | 어떤 로그와 탐지 서비스를 연결할 것인가? |
| Incident Response | 14% | 격리·조사·복구를 어떻게 자동화할 것인가? |
| Infrastructure Security | 18% | 네트워크와 워크로드 경계를 어떻게 보호할 것인가? |
| Identity and Access Management | 20% | 최소 권한과 조직 단위 통제를 어떻게 구현할 것인가? |
| Data Protection | 18% | 데이터 분류, 암호화, 키 수명 주기를 어떻게 설계할 것인가? |
| Security Foundations and Governance | 14% | 다중 계정 환경의 기준과 증적을 어떻게 유지할 것인가? |

## 1. Detection

로그 생성, 중앙 수집, 탐지, 대응을 하나의 흐름으로 이해해야 한다.

- **CloudTrail**: 관리 이벤트와 데이터 이벤트를 기록한다. 조직 추적, 로그 파일 검증, 중앙 S3 버킷을 함께 고려한다.
- **AWS Config**: 리소스 구성 변경 이력과 규정 준수 상태를 평가한다.
- **GuardDuty**: CloudTrail, VPC Flow Logs, DNS 로그 등 여러 신호를 분석해 위협을 탐지한다.
- **Security Hub**: 여러 계정과 보안 서비스의 결과를 표준 형식으로 집계한다.
- **Detective**: 관련 관측 데이터를 연결해 조사 범위를 좁힌다.
- **Macie**: S3의 민감 데이터 탐색과 분류에 사용한다.

> CloudTrail은 “누가 어떤 API를 호출했는가”, Config는 “리소스 구성이 어떻게 바뀌었는가”, GuardDuty는 “그 활동이 위협으로 의심되는가”에 답한다.

## 2. Incident Response

사고 대응은 준비 → 탐지·분석 → 격리 → 제거·복구 → 회고 순서로 설계한다.

- 손상된 EC2는 종료하기 전에 격리용 보안 그룹 적용, 스냅샷·메모리 등 증거 보존 여부를 판단한다.
- EventBridge와 Lambda 또는 Systems Manager Automation으로 반복 대응을 자동화한다.
- 교차 계정 조사 역할과 비상 접근 절차를 사고 전에 준비한다.
- 로그 버킷은 별도 보안 계정에 두고 Object Lock, 버전 관리, 최소 권한으로 변조를 어렵게 한다.
- 액세스 키 노출 시 키 비활성화만 하지 말고 CloudTrail로 사용 범위를 조사하고 세션·권한·관련 비밀을 함께 교체한다.

## 3. Infrastructure Security

- **Security Group**: ENI 단위, 상태 저장, 허용 규칙만 사용
- **Network ACL**: 서브넷 단위, 상태 비저장, 허용·거부 규칙과 순서 사용
- **AWS Network Firewall**: VPC 경계의 상태 저장 네트워크 방화벽
- **AWS WAF**: CloudFront, ALB, API Gateway 등의 HTTP 계층 보호
- **AWS Shield**: DDoS 보호. Standard는 기본 제공되고 Advanced는 추가 대응·가시성 요구에 사용
- **PrivateLink**: 서비스 소비자와 제공자를 사설 엔드포인트로 연결
- **Transit Gateway**: 다수 VPC와 온프레미스 네트워크의 허브 연결

검사 VPC를 설계할 때는 라우팅 대칭성, 장애 조치, Gateway Load Balancer 엔드포인트, 중앙 egress 경로를 함께 확인한다.

## 4. Identity and Access Management

IAM 문제는 Principal, Action, Resource, Condition과 명시적 거부 여부를 순서대로 확인한다.

- 장기 액세스 키보다 IAM 역할과 단기 자격 증명을 사용한다.
- 사람의 접근은 IAM Identity Center와 MFA를 우선 검토한다.
- 계정 안의 권한 정책뿐 아니라 SCP, 리소스 정책, permissions boundary, 세션 정책을 함께 평가한다.
- SCP는 권한을 부여하지 않고 조직 구성원이 가질 수 있는 최대 권한 범위를 제한한다.
- 교차 계정 역할은 역할의 신뢰 정책과 호출자의 권한 정책이 모두 맞아야 한다.
- `iam:PassRole`은 서비스에 역할을 전달하는 권한이므로 대상 역할과 서비스 조건을 좁힌다.

## 5. Data Protection

| 서비스 | 키 포인트 |
|---|---|
| AWS KMS | 키 정책, grants, key rotation, encryption context |
| CloudHSM | 전용 HSM과 고객 제어가 필요한 경우 |
| Secrets Manager | 비밀 저장과 자동 회전 |
| ACM | AWS 통합 서비스의 공인·사설 인증서 관리 |
| S3 Object Lock | WORM 보존과 삭제 방지 |
| AWS Backup | 여러 서비스의 백업 정책·볼트 중앙 관리 |

Envelope encryption에서는 데이터 키로 데이터를 암호화하고 KMS 키로 데이터 키를 보호한다. 교차 계정·교차 리전 암호화에서는 서비스 권한과 키 정책을 별도로 검증해야 한다.

## 6. Security Foundations and Governance

- AWS Organizations와 Control Tower로 다중 계정 기준을 만든다.
- 로그 아카이브, 보안 도구, 워크로드 계정을 분리해 직무 분리와 침해 범위 축소를 확보한다.
- SCP와 Config 규칙으로 예방·탐지 통제를 결합한다.
- Artifact는 AWS의 규정 준수 보고서와 계약 문서를 확인할 때 사용한다.
- Audit Manager는 감사 프레임워크에 맞춘 증거 수집을 지원한다.

## 문제 풀이 기준

1. 요구사항에서 **예방, 탐지, 대응, 복구** 중 무엇을 요구하는지 먼저 구분한다.
2. “가장 적은 운영 부담”이 있으면 관리형 서비스와 자동화를 우선한다.
3. 조직 전체 요구라면 개별 계정보다 Organizations, delegated administrator, 중앙 로그 계정을 고려한다.
4. 암호화 문제는 데이터 키뿐 아니라 KMS 키 정책, 서비스 주체, 교차 계정 권한까지 확인한다.
5. 즉시 삭제보다 증거 보존과 격리를 우선해야 하는 사고 대응 문맥인지 확인한다.

## 최종 점검표

- CloudTrail, Config, GuardDuty, Security Hub의 역할을 시나리오로 구분할 수 있는가?
- IAM 정책 평가에서 명시적 거부와 SCP·boundary를 함께 계산할 수 있는가?
- SG와 NACL의 상태 저장 여부와 적용 범위를 설명할 수 있는가?
- KMS 키 정책과 IAM 정책이 언제 함께 필요한지 아는가?
- 다중 계정 로그 보존과 자동 사고 대응 흐름을 그릴 수 있는가?

## 공식 문서

- [AWS Certified Security – Specialty SCS-C03 시험 안내](https://docs.aws.amazon.com/aws-certification/latest/security-specialty-03/security-specialty-03.html)
- [AWS Certification 시험 가이드](https://docs.aws.amazon.com/aws-certification/latest/examguides/aws-certification-exam-guides.html)
- [AWS 보안 사고 대응 가이드](https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/welcome.html)
