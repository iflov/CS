---
tags:
- aws
- systems-manager
- parameter-store
- configuration-management
- secrets-management
- devops
- cloud
created: 2025-09-01
updated: 2025-09-01
aliases:
- AWS SSM
- Parameter Store
- SSM Parameter Store
description: AWS Systems Manager Parameter Store를 사용한 구성 데이터 및 시크릿 관리 가이드
status: published
category: guide
---

# AWS Systems Manager Parameter Store 완벽 가이드

> [!info] 개요
> AWS Systems Manager Parameter Store는 구성 데이터와 시크릿을 안전하게 저장하고 관리하는 서비스입니다. 애플리케이션 설정, API 키, 데이터베이스 자격 증명 등을 중앙에서 관리하고 버전을 추적할 수 있습니다.

## 📑 목차

- [[#🎯 AWS Systems Manager란?]]
- [[#💾 Parameter Store 소개]]
- [[#🔥 핵심 기능]]
- [[#⚡ 빠른 시작]]
- [[#🔐 파라미터 타입과 계층]]
- [[#💡 실전 예제]]
- [[#🔗 AWS 서비스 통합]]
- [[#⚠️ 주의사항]]
- [[#💰 비용 최적화]]
- [[#📚 참고자료]]

---

## 🎯 AWS Systems Manager란?

### 정의

AWS Systems Manager(SSM)는 **클라우드 및 온프레미스 인프라를 쉽고 안전하게 보기, 제어, 운영**할 수 있도록 도와주는 관리 서비스입니다.

### 주요 구성 요소

> [!example] Systems Manager 핵심 기능
> - **Parameter Store**: 구성 데이터 관리
> - **Session Manager**: SSH 키 없이 EC2 접속
> - **Patch Manager**: 패치 관리 자동화
> - **Run Command**: 원격 명령 실행
> - **State Manager**: 인스턴스 상태 유지
> - **Inventory**: 인벤토리 수집

---

## 💾 Parameter Store 소개

### 핵심 개념

Parameter Store는 AWS Systems Manager의 기능으로, **구성 데이터를 안전하게 저장**하는 중앙화된 저장소입니다.

> [!note] 중앙화된 데이터 저장소
> 버전 관리가 가능한 안전한 데이터베이스로 다음을 저장할 수 있습니다:
> - 📱 **애플리케이션 설정**
> - 🔑 **시크릿** (비밀번호, API 키, 데이터베이스 자격 증명)
> - 🌍 **환경 변수**
> - 🚦 **Feature Flags**

### 왜 Parameter Store를 사용해야 하나?

> [!tip] 주요 이점
> 민감한 값을 코드나 설정 파일에 하드코딩하는 대신, Parameter Store에 안전하게 저장하고 필요할 때 가져옵니다.

```python
# ❌ 나쁜 예: 하드코딩된 시크릿
db_password = "my-secret-password-123"

# ✅ 좋은 예: Parameter Store 사용
import boto3
ssm = boto3.client('ssm')
response = ssm.get_parameter(Name='/myapp/db/password', WithDecryption=True)
db_password = response['Parameter']['Value']
```

---

## 🔥 핵심 기능

### 1. 보안 저장소 (Secure Storage)

> [!success] AWS KMS 암호화 지원
> - 모든 SecureString 파라미터는 KMS로 암호화
> - 저장 시(at rest) 및 전송 시(in transit) 암호화
> - 고객 관리형 CMK 또는 AWS 관리형 키 사용 가능

### 2. 버전 관리 (Versioning)

파라미터를 업데이트할 때마다 **새 버전이 자동으로 생성**됩니다.

```bash
# 버전 히스토리 조회
aws ssm get-parameter-history \
    --name "/myapp/database/url"
```

### 3. 접근 제어 (Access Control)

> [!warning] IAM 정책 기반 제어
> IAM 정책을 사용하여 누가 어떤 파라미터에 접근할 수 있는지 세밀하게 제어합니다.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:GetParameter",
        "ssm:GetParameters"
      ],
      "Resource": "arn:aws:ssm:*:*:parameter/prod/*"
    }
  ]
}
```

### 4. 광범위한 통합 (Integration)

> [!info] 지원 서비스
> - **AWS CLI**: 명령줄 인터페이스
> - **SDKs**: 모든 프로그래밍 언어
> - **EC2**: 인스턴스 메타데이터
> - **Lambda**: 환경 변수
> - **ECS/EKS**: 컨테이너 환경
> - **CloudFormation**: IaC 통합

### 5. 비용 구조

| 파라미터 타입 | 특징 | 비용 |
|-------------|------|------|
| **Standard** | 4KB 이하, 기본 기능 | **무료** (10,000개까지) |
| **Advanced** | 8KB까지, 정책 지원, 파라미터 유효기간 | **유료** ($0.05/월/파라미터) |

---

## ⚡ 빠른 시작

### 1. 파라미터 생성

> [!example] AWS CLI로 파라미터 생성

```bash
# String 타입 파라미터
aws ssm put-parameter \
    --name "/myapp/api/endpoint" \
    --value "https://api.example.com" \
    --type "String"

# SecureString 타입 (암호화)
aws ssm put-parameter \
    --name "/myapp/db/password" \
    --value "MySecretPassword123!" \
    --type "SecureString"

# StringList 타입
aws ssm put-parameter \
    --name "/myapp/regions" \
    --value "us-east-1,us-west-2,eu-west-1" \
    --type "StringList"
```

### 2. 파라미터 조회

```bash
# 단일 파라미터 조회
aws ssm get-parameter \
    --name "/myapp/api/endpoint"

# 암호화된 파라미터 복호화하여 조회
aws ssm get-parameter \
    --name "/myapp/db/password" \
    --with-decryption

# 여러 파라미터 한 번에 조회
aws ssm get-parameters \
    --names "/myapp/api/endpoint" "/myapp/db/password"
```

### 3. 파라미터 업데이트

```bash
# 파라미터 값 업데이트 (새 버전 생성)
aws ssm put-parameter \
    --name "/myapp/api/endpoint" \
    --value "https://api-v2.example.com" \
    --overwrite
```

---

## 🔐 파라미터 타입과 계층

### 파라미터 타입

> [!note] 3가지 파라미터 타입

| 타입 | 설명 | 사용 사례 |
|------|------|----------|
| **String** | 일반 텍스트 | 설정값, URL, 플래그 |
| **StringList** | 쉼표로 구분된 문자열 | 리전 목록, IP 목록 |
| **SecureString** | KMS로 암호화된 문자열 | 비밀번호, API 키, 토큰 |

### 파라미터 계층 구조

> [!tip] 계층적 네이밍 전략
> 슬래시(/)를 사용하여 계층 구조를 만들어 파라미터를 체계적으로 관리합니다.

```
/환경/서비스/구성요소/속성
```

예시:
```
/prod/payment-service/database/host
/prod/payment-service/database/port
/prod/payment-service/database/username
/prod/payment-service/database/password

/dev/payment-service/database/host
/dev/payment-service/database/port
```

### 경로 기반 정책

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ssm:GetParameter*",
      "Resource": "arn:aws:ssm:*:*:parameter/prod/*",
      "Condition": {
        "StringEquals": {
          "aws:PrincipalTag/Environment": "Production"
        }
      }
    }
  ]
}
```

---

## 💡 실전 예제

### Python 애플리케이션 예제

> [!example] Django 설정 통합

```python
# settings.py
import boto3
from botocore.exceptions import ClientError

def get_parameter(name, decrypt=True):
    """Parameter Store에서 값을 가져오는 헬퍼 함수"""
    ssm = boto3.client('ssm', region_name='us-east-1')
    try:
        response = ssm.get_parameter(
            Name=name,
            WithDecryption=decrypt
        )
        return response['Parameter']['Value']
    except ClientError as e:
        print(f"Error getting parameter {name}: {e}")
        return None

# 환경별 설정
ENV = get_parameter('/myapp/environment', decrypt=False)

# 데이터베이스 설정
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': get_parameter(f'/{ENV}/myapp/db/name', decrypt=False),
        'USER': get_parameter(f'/{ENV}/myapp/db/user', decrypt=False),
        'PASSWORD': get_parameter(f'/{ENV}/myapp/db/password'),
        'HOST': get_parameter(f'/{ENV}/myapp/db/host', decrypt=False),
        'PORT': get_parameter(f'/{ENV}/myapp/db/port', decrypt=False),
    }
}

# API 키
STRIPE_API_KEY = get_parameter(f'/{ENV}/myapp/stripe/api_key')
AWS_ACCESS_KEY_ID = get_parameter(f'/{ENV}/myapp/aws/access_key')
AWS_SECRET_ACCESS_KEY = get_parameter(f'/{ENV}/myapp/aws/secret_key')
```

### Lambda 함수 예제

> [!example] Lambda에서 Parameter Store 사용

```python
import json
import boto3
import os
from functools import lru_cache

ssm = boto3.client('ssm')

@lru_cache(maxsize=128)
def get_parameter(name):
    """캐싱을 통한 성능 최적화"""
    response = ssm.get_parameter(
        Name=name,
        WithDecryption=True
    )
    return response['Parameter']['Value']

def lambda_handler(event, context):
    # 환경 변수로 파라미터 이름 전달
    db_host = get_parameter(os.environ['DB_HOST_PARAM'])
    db_pass = get_parameter(os.environ['DB_PASS_PARAM'])
    api_key = get_parameter(os.environ['API_KEY_PARAM'])
    
    # 비즈니스 로직
    # ...
    
    return {
        'statusCode': 200,
        'body': json.dumps('Success')
    }
```

### Docker/ECS 통합

> [!example] ECS Task Definition

```json
{
  "family": "my-app",
  "taskRoleArn": "arn:aws:iam::123456789012:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "my-app:latest",
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:ssm:region:123456789012:parameter/prod/db/password"
        },
        {
          "name": "API_KEY",
          "valueFrom": "arn:aws:ssm:region:123456789012:parameter/prod/api/key"
        }
      ],
      "environment": [
        {
          "name": "DB_HOST",
          "valueFrom": "arn:aws:ssm:region:123456789012:parameter/prod/db/host"
        }
      ]
    }
  ]
}
```

---

## 🔗 AWS 서비스 통합

### CloudFormation 통합

```yaml
# CloudFormation 템플릿
Parameters:
  Environment:
    Type: String
    Default: dev

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Sub '{{resolve:ssm:/${Environment}/ami-id}}'
      InstanceType: !Sub '{{resolve:ssm:/${Environment}/instance-type}}'
      KeyName: !Sub '{{resolve:ssm:/${Environment}/key-pair}}'
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          DB_PASSWORD={{resolve:secretsmanager:${Environment}-db-password}}
          echo "Database password retrieved"
```

### CodeBuild 통합

```yaml
# buildspec.yml
version: 0.2

phases:
  pre_build:
    commands:
      - echo Retrieving parameters from Parameter Store
      - export API_KEY=$(aws ssm get-parameter --name /prod/api/key --with-decryption --query 'Parameter.Value' --output text)
      - export DB_HOST=$(aws ssm get-parameter --name /prod/db/host --query 'Parameter.Value' --output text)
  
  build:
    commands:
      - echo Building with retrieved parameters
      - docker build --build-arg API_KEY=$API_KEY --build-arg DB_HOST=$DB_HOST -t my-app .
```

---

## ⚠️ 주의사항

### 1. 접근 제어 (Access Control)

> [!warning] 최소 권한 원칙
> 항상 최소 권한(least-privilege) 권한을 사용하세요.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:GetParameter",
        "ssm:GetParametersByPath"
      ],
      "Resource": [
        "arn:aws:ssm:*:*:parameter/prod/myapp/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": "kms:Decrypt",
      "Resource": "arn:aws:kms:*:*:key/*",
      "Condition": {
        "StringEquals": {
          "kms:ViaService": "ssm.region.amazonaws.com"
        }
      }
    }
  ]
}
```

### 2. API 속도 제한 (Rate Limits)

> [!danger] 속도 제한 주의
> 파라미터를 자주 읽으면 API 속도 제한에 도달할 수 있습니다.

**해결 방법:**
- 애플리케이션 레벨 캐싱 구현
- 파라미터를 환경 변수로 주입 (ECS/Lambda)
- 배치로 여러 파라미터 한 번에 조회

```python
# 캐싱 예제
import time
from functools import wraps

def cache_parameter(ttl=300):  # 5분 TTL
    cache = {}
    
    def decorator(func):
        @wraps(func)
        def wrapper(name):
            if name in cache:
                value, timestamp = cache[name]
                if time.time() - timestamp < ttl:
                    return value
            
            value = func(name)
            cache[name] = (value, time.time())
            return value
        return wrapper
    return decorator

@cache_parameter(ttl=600)  # 10분 캐시
def get_cached_parameter(name):
    return get_parameter(name)
```

### 3. 비용 관리

> [!tip] Advanced 파라미터 비용 주의
> - Standard 파라미터는 무료 (10,000개까지)
> - Advanced 파라미터는 유료 (정책, >4KB 크기 등)
> - 불필요한 Advanced 기능 사용 자제

---

## 💰 비용 최적화

### Standard vs Advanced 파라미터

| 기능 | Standard | Advanced |
|------|----------|----------|
| **최대 크기** | 4 KB | 8 KB |
| **파라미터 정책** | ❌ | ✅ |
| **파라미터 수** | 10,000개 | 100,000개 |
| **처리량** | 표준 | 높음 |
| **비용** | 무료 | $0.05/월/파라미터 |

### 비용 절감 전략

> [!success] 최적화 팁
> 1. **Standard 파라미터 우선 사용**: 4KB 이하는 Standard로
> 2. **계층 구조 활용**: GetParametersByPath로 한 번에 조회
> 3. **캐싱 구현**: API 호출 횟수 감소
> 4. **불필요한 버전 정리**: 오래된 버전 삭제

```bash
# 특정 경로의 모든 파라미터 조회 (API 호출 1회)
aws ssm get-parameters-by-path \
    --path "/prod/myapp" \
    --recursive \
    --with-decryption
```

---

## 📚 참고자료

### 공식 문서
- [AWS Systems Manager 공식 문서](https://docs.aws.amazon.com/systems-manager/)
- [Parameter Store 개발자 가이드](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
- [Parameter Store API 레퍼런스](https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_Operations.html)

### 관련 문서
- [[AWS Secrets Manager vs Parameter Store]]
- [[AWS KMS 암호화 가이드]]
- [[IAM 정책 작성 가이드]]
- [[ECS 시크릿 관리]]

### 베스트 프랙티스
- [Parameter Store 네이밍 규칙](https://aws.amazon.com/blogs/mt/organize-parameters-by-hierarchy-tags-or-amazon-cloudwatch-events-with-amazon-ec2-systems-manager-parameter-store/)
- [파라미터 계층 구조 설계](https://aws.amazon.com/blogs/compute/using-aws-systems-manager-parameter-store-secure-string-parameters-in-aws-cloudformation-templates/)
- [보안 모범 사례](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-securestring.html)

---

> [!quote]
> "Parameter Store는 단순히 구성 저장소가 아니라, 전체 인프라의 보안과 운영 효율성을 높이는 핵심 서비스입니다."

> [!success] 다음 단계
> - Parameter Store로 기존 하드코딩된 값 마이그레이션
> - IAM 정책으로 세밀한 접근 제어 구현
> - CI/CD 파이프라인에 Parameter Store 통합
> - 모니터링 및 감사 로깅 설정