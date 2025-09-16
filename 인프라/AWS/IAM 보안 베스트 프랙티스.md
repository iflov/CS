---
tags:
- aws
- iam
- security
- best-practices
- cloud-security
created: 2025-01-06
updated: 2025-01-06
aliases:
- IAM Security
- AWS IAM Best Practices
description: AWS IAM 보안 베스트 프랙티스와 권한 관리 전략
status: published
category: guide
---

# AWS IAM 보안 베스트 프랙티스

> [!info] 개요
> AWS Identity and Access Management(IAM)는 AWS 리소스에 대한 접근을 안전하게 제어하는 핵심 서비스입니다. 이 문서는 IAM을 통한 보안 강화 방법과 실무에서 적용해야 할 베스트 프랙티스를 다룹니다.

## 📑 목차

- [[#🔐 Root 계정 보안]]
- [[#👤 IAM User 관리]]
- [[#🎭 IAM Role 활용]]
- [[#📋 정책(Policy) 설계]]
- [[#🔑 자격 증명 관리]]
- [[#📊 모니터링과 감사]]
- [[#⚠️ 일반적인 실수와 해결방법]]
- [[#📚 참고자료]]

---

## 🔐 Root 계정 보안

### 1. Root 계정 사용 최소화

> [!danger] 절대 규칙
> Root 계정은 계정 생성 후 초기 설정에만 사용하고, 일상적인 작업에는 절대 사용하지 마세요.

#### Root 계정만 가능한 작업
- AWS 계정 설정 변경
- 결제 정보 수정
- AWS Support 플랜 변경
- 계정 종료
- AWS Marketplace 판매자 등록
- S3 버킷의 MFA Delete 활성화

### 2. MFA 필수 설정

```bash
# AWS CLI로 MFA 디바이스 등록 확인
aws iam list-mfa-devices --user-name root
```

> [!tip] 추천 MFA 앱
> - **Authy**: 백업 기능 지원
> - **Google Authenticator**: 간단하고 안정적
> - **Hardware MFA**: YubiKey 등 물리적 디바이스

### 3. Access Key 삭제

> [!warning] Root Access Key
> Root 계정의 Access Key는 생성하지 마세요. 이미 있다면 즉시 삭제하세요.

```bash
# Root 계정 Access Key 확인 (IAM User로 실행)
aws iam get-account-summary | grep "AccountAccessKeysPresent"
```

---

## 👤 IAM User 관리

### 1. 최소 권한 원칙 (Principle of Least Privilege)

> [!note] 기본 원칙
> 사용자에게는 업무 수행에 필요한 최소한의 권한만 부여합니다.

#### 단계별 권한 부여 프로세스
1. **초기**: 읽기 권한만 부여
2. **검증**: 필요한 작업 확인
3. **확장**: 필수 권한만 추가
4. **검토**: 주기적으로 권한 재검토

### 2. 그룹을 통한 권한 관리

```hcl
# Terraform으로 그룹 기반 권한 관리
resource "aws_iam_group" "developers" {
  name = "developers"
  path = "/teams/"
}

resource "aws_iam_group" "devops" {
  name = "devops"
  path = "/teams/"
}

resource "aws_iam_group" "readonly" {
  name = "readonly"
  path = "/teams/"
}

# 그룹별 정책 연결
resource "aws_iam_group_policy_attachment" "developers_policy" {
  group      = aws_iam_group.developers.name
  policy_arn = aws_iam_policy.developer_policy.arn
}
```

### 3. 강력한 비밀번호 정책

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:ChangePassword",
      "Resource": "arn:aws:iam::*:user/${aws:username}"
    }
  ]
}
```

> [!example] 비밀번호 정책 설정
> ```bash
> aws iam update-account-password-policy \
>   --minimum-password-length 14 \
>   --require-symbols \
>   --require-numbers \
>   --require-uppercase-characters \
>   --require-lowercase-characters \
>   --allow-users-to-change-password \
>   --max-password-age 90 \
>   --password-reuse-prevention 5
> ```

---

## 🎭 IAM Role 활용

### 1. Cross-Account Access

> [!tip] Role 사용 시나리오
> - EC2 인스턴스에서 AWS 서비스 접근
> - Lambda 함수의 권한 부여
> - 다른 AWS 계정과의 안전한 연동
> - 임시 자격 증명이 필요한 경우

#### EC2 Instance Role 예제

```hcl
# EC2 인스턴스용 Role 생성
resource "aws_iam_role" "ec2_s3_access" {
  name = "ec2-s3-access-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      }
    ]
  })
}

# S3 읽기 정책 연결
resource "aws_iam_role_policy" "s3_read_policy" {
  name = "s3-read-policy"
  role = aws_iam_role.ec2_s3_access.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:ListBucket"
        ]
        Resource = [
          "arn:aws:s3:::my-bucket",
          "arn:aws:s3:::my-bucket/*"
        ]
      }
    ]
  })
}

# Instance Profile 생성
resource "aws_iam_instance_profile" "ec2_profile" {
  name = "ec2-s3-access-profile"
  role = aws_iam_role.ec2_s3_access.name
}
```

### 2. Service-Linked Roles

> [!info] 자동 생성 Role
> 일부 AWS 서비스는 자동으로 Service-Linked Role을 생성합니다.
> - Auto Scaling
> - RDS
> - ElastiCache
> - ECS

---

## 📋 정책(Policy) 설계

### 1. 정책 구조 이해

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow|Deny",
      "Action": [
        "service:action"
      ],
      "Resource": "arn:aws:service:region:account:resource",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "us-east-1"
        }
      }
    }
  ]
}
```

### 2. 조건(Condition) 활용

> [!example] IP 제한 정책
> ```json
> {
>   "Version": "2012-10-17",
>   "Statement": [
>     {
>       "Effect": "Deny",
>       "Action": "*",
>       "Resource": "*",
>       "Condition": {
>         "NotIpAddress": {
>           "aws:SourceIp": [
>             "203.0.113.0/24",
>             "198.51.100.0/24"
>           ]
>         }
>       }
>     }
>   ]
> }
> ```

### 3. 정책 시뮬레이터 활용

```bash
# CLI로 정책 시뮬레이션
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/test-user \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/*
```

---

## 🔑 자격 증명 관리

### 1. Access Key 순환

> [!warning] 90일 규칙
> Access Key는 90일마다 순환하세요.

```bash
# 오래된 Access Key 찾기
aws iam list-access-keys --user-name USERNAME

# 새 Access Key 생성
aws iam create-access-key --user-name USERNAME

# 이전 Key 비활성화
aws iam update-access-key --access-key-id AKIAIOSFODNN7EXAMPLE \
  --status Inactive --user-name USERNAME

# 이전 Key 삭제 (테스트 후)
aws iam delete-access-key --access-key-id AKIAIOSFODNN7EXAMPLE \
  --user-name USERNAME
```

### 2. AWS Secrets Manager 활용

```python
import boto3
import json

def get_secret(secret_name):
    """Secrets Manager에서 비밀 정보 가져오기"""
    client = boto3.client('secretsmanager')
    
    try:
        response = client.get_secret_value(SecretId=secret_name)
        secret = json.loads(response['SecretString'])
        return secret
    except Exception as e:
        print(f"Error retrieving secret: {e}")
        return None

# 사용 예제
db_credentials = get_secret('prod/database/credentials')
```

### 3. Systems Manager Parameter Store

```bash
# 파라미터 저장 (암호화)
aws ssm put-parameter \
  --name "/myapp/database/password" \
  --value "MySecretPassword" \
  --type "SecureString" \
  --key-id "alias/aws/ssm"

# 파라미터 조회
aws ssm get-parameter \
  --name "/myapp/database/password" \
  --with-decryption
```

---

## 📊 모니터링과 감사

### 1. CloudTrail 설정

> [!important] 필수 설정
> 모든 리전에서 CloudTrail을 활성화하고 S3에 로그를 저장하세요.

```bash
# CloudTrail 생성
aws cloudtrail create-trail \
  --name my-trail \
  --s3-bucket-name my-cloudtrail-bucket \
  --is-multi-region-trail
```

### 2. Access Advisor 활용

```bash
# 사용자의 서비스 접근 기록 확인
aws iam generate-service-last-accessed-details \
  --arn arn:aws:iam::123456789012:user/test-user

# 결과 조회
aws iam get-service-last-accessed-details \
  --job-id JOB_ID
```

### 3. AWS Config Rules

> [!tip] 추천 Config Rules
> - `iam-root-access-key-check`: Root Access Key 확인
> - `iam-user-mfa-enabled`: MFA 활성화 확인
> - `iam-password-policy`: 비밀번호 정책 준수 확인
> - `access-keys-rotated`: Access Key 순환 확인

---

## ⚠️ 일반적인 실수와 해결방법

### 1. 과도한 권한 부여

> [!danger] 안티패턴
> ```json
> {
>   "Effect": "Allow",
>   "Action": "*",
>   "Resource": "*"
> }
> ```

> [!success] 올바른 접근
> ```json
> {
>   "Effect": "Allow",
>   "Action": [
>     "s3:GetObject",
>     "s3:PutObject"
>   ],
>   "Resource": "arn:aws:s3:::my-bucket/user/${aws:username}/*"
> }
> ```

### 2. Hard-coded Credentials

> [!danger] 절대 금지
> ```python
> # 잘못된 예
> aws_access_key = "AKIAIOSFODNN7EXAMPLE"
> aws_secret_key = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
> ```

> [!success] 올바른 방법
> ```python
> # 환경 변수 사용
> import os
> aws_access_key = os.environ.get('AWS_ACCESS_KEY_ID')
> 
> # 또는 IAM Role 사용 (EC2, Lambda 등)
> import boto3
> client = boto3.client('s3')  # 자동으로 Role 자격 증명 사용
> ```

### 3. 정책 충돌

> [!note] 평가 순서
> 1. **명시적 Deny**: 항상 우선
> 2. **명시적 Allow**: Deny가 없을 때 적용
> 3. **암시적 Deny**: 기본값

---

## 📚 참고자료

### AWS 공식 문서
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Security Best Practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/security-best-practices.html)
- [IAM Policy Simulator](https://policysim.aws.amazon.com/)

### 보안 도구
- [AWS Security Hub](https://aws.amazon.com/security-hub/)
- [AWS Access Analyzer](https://aws.amazon.com/iam/features/analyze-access/)
- [ScoutSuite](https://github.com/nccgroup/ScoutSuite) - 보안 감사 도구

### 관련 문서
- [[AWS 기초 가이드]]
- [[AWS CLI 고급 사용법]]
- [[AWS Organizations 멀티 계정 전략]]

---

> [!quote]
> "Security is not a product, but a process." - Bruce Schneier

> [!success] 체크리스트
> - [ ] Root 계정 MFA 활성화
> - [ ] Root Access Key 삭제
> - [ ] IAM 사용자별 개별 계정 생성
> - [ ] 최소 권한 원칙 적용
> - [ ] 정기적인 Access Key 순환
> - [ ] CloudTrail 로깅 활성화
> - [ ] 정기적인 권한 감사