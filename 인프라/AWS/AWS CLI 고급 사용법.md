---
tags:
- aws
- cli
- devops
- automation
- scripting
created: 2025-01-06
updated: 2025-01-06
aliases:
- AWS CLI Advanced
- AWS Command Line
description: AWS CLI 고급 사용법과 자동화 스크립팅 가이드
status: published
category: guide
---

# AWS CLI 고급 사용법

> [!info] 개요
> AWS CLI의 고급 기능을 활용한 효율적인 클라우드 리소스 관리, 자동화 스크립팅, 그리고 실무에서 유용한 팁과 트릭을 다룹니다.

## 📑 목차

- [[#⚙️ 고급 설정]]
- [[#🎯 프로파일 관리]]
- [[#🔍 쿼리와 필터링]]
- [[#📝 출력 포맷팅]]
- [[#🚀 자동화 스크립팅]]
- [[#🛠️ 유용한 명령어 모음]]
- [[#⚡ 성능 최적화]]
- [[#🐛 디버깅과 트러블슈팅]]
- [[#📚 참고자료]]

---

## ⚙️ 고급 설정

### 1. 환경 변수 활용

> [!tip] 주요 환경 변수
> ```bash
> # AWS 자격 증명
> export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
> export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG
> export AWS_SESSION_TOKEN=AQoDYXdzEJr...  # 임시 자격 증명용
> 
> # 기본 설정
> export AWS_DEFAULT_REGION=ap-northeast-2
> export AWS_DEFAULT_OUTPUT=json
> 
> # 프로파일 지정
> export AWS_PROFILE=production
> 
> # 엔드포인트 커스터마이징
> export AWS_ENDPOINT_URL=http://localhost:4566  # LocalStack 등
> ```

### 2. 설정 파일 구조

```ini
# ~/.aws/config
[default]
region = us-east-1
output = json
cli_follow_urlparam = false
cli_timestamp_format = iso8601

[profile development]
region = us-west-2
output = table
role_arn = arn:aws:iam::123456789012:role/Developer
source_profile = default
mfa_serial = arn:aws:iam::123456789012:mfa/username

[profile production]
region = ap-northeast-2
output = json
role_arn = arn:aws:iam::987654321098:role/Administrator
source_profile = default
```

### 3. 별칭(Alias) 설정

```bash
# ~/.aws/cli/alias
[toplevel]
whoami = sts get-caller-identity
list-instances = ec2 describe-instances \
  --query 'Reservations[].Instances[].{ID:InstanceId,Type:InstanceType,State:State.Name,Name:Tags[?Key==`Name`]|[0].Value}'
last-logs = logs tail --follow --format short
s3-size = s3 ls --recursive --human-readable --summarize
```

---

## 🎯 프로파일 관리

### 1. 다중 계정 관리

```bash
# 프로파일 추가
aws configure --profile client-a
aws configure --profile client-b

# 프로파일 목록 확인
aws configure list-profiles

# 특정 프로파일로 명령 실행
aws s3 ls --profile client-a
aws ec2 describe-instances --profile client-b
```

### 2. Role Assume 자동화

> [!example] MFA를 사용한 Role Assume
> ```bash
> #!/bin/bash
> # assume-role.sh
> 
> ROLE_ARN="arn:aws:iam::123456789012:role/AdminRole"
> SESSION_NAME="admin-session-$(date +%s)"
> MFA_SERIAL="arn:aws:iam::123456789012:mfa/username"
> 
> echo -n "Enter MFA token: "
> read MFA_TOKEN
> 
> # Assume role with MFA
> CREDENTIALS=$(aws sts assume-role \
>   --role-arn "$ROLE_ARN" \
>   --role-session-name "$SESSION_NAME" \
>   --serial-number "$MFA_SERIAL" \
>   --token-code "$MFA_TOKEN" \
>   --query 'Credentials.[AccessKeyId,SecretAccessKey,SessionToken]' \
>   --output text)
> 
> # Export credentials
> export AWS_ACCESS_KEY_ID=$(echo $CREDENTIALS | cut -d' ' -f1)
> export AWS_SECRET_ACCESS_KEY=$(echo $CREDENTIALS | cut -d' ' -f2)
> export AWS_SESSION_TOKEN=$(echo $CREDENTIALS | cut -d' ' -f3)
> 
> echo "Role assumed successfully!"
> ```

### 3. AWS SSO 통합

```bash
# SSO 설정
aws configure sso
# SSO start URL: https://my-sso-portal.awsapps.com/start
# SSO Region: us-east-1

# SSO 로그인
aws sso login --profile my-sso-profile

# SSO 세션 확인
aws sts get-caller-identity --profile my-sso-profile
```

---

## 🔍 쿼리와 필터링

### 1. JMESPath 쿼리 마스터하기

> [!note] JMESPath 기본 문법
> - `.` : 현재 노드
> - `[]` : 배열 인덱싱
> - `*` : 와일드카드
> - `?` : 필터 표현식
> - `|` : 파이프 연산자

#### 실용적인 쿼리 예제

```bash
# EC2 인스턴스 정보 추출
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].[InstanceId,State.Name,Tags[?Key==`Name`].Value|[0]]' \
  --output table

# 특정 태그를 가진 리소스 찾기
aws ec2 describe-instances \
  --query 'Reservations[].Instances[?Tags[?Key==`Environment` && Value==`Production`]].[InstanceId,PrivateIpAddress]'

# 중첩된 데이터 평탄화
aws ec2 describe-security-groups \
  --query 'SecurityGroups[].{
    Name:GroupName,
    ID:GroupId,
    InboundRules:IpPermissions[].{
      Protocol:IpProtocol,
      FromPort:FromPort,
      ToPort:ToPort,
      Sources:IpRanges[].CidrIp
    }
  }'
```

### 2. 서버 사이드 필터링

```bash
# EC2 필터 사용
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
            "Name=instance-type,Values=t3.micro,t3.small"

# 태그 기반 필터링
aws ec2 describe-instances \
  --filters "Name=tag:Environment,Values=Production" \
            "Name=tag:Application,Values=WebServer"
```

### 3. 복잡한 필터 조합

```bash
# S3 버킷에서 특정 조건의 객체 찾기
aws s3api list-objects-v2 \
  --bucket my-bucket \
  --query "Contents[?Size > \`1000000\` && LastModified >= \`2024-01-01\`].[Key,Size,LastModified]" \
  --output table
```

---

## 📝 출력 포맷팅

### 1. 출력 형식 옵션

```bash
# JSON (기본)
aws ec2 describe-instances --output json

# Table (읽기 편함)
aws ec2 describe-instances --output table

# Text (스크립팅용)
aws ec2 describe-instances --output text

# YAML
aws ec2 describe-instances --output yaml

# 커스텀 구분자
aws ec2 describe-instances --output text --query 'Reservations[].Instances[].[InstanceId,State.Name]' | column -t
```

### 2. 출력 후처리

```bash
# jq를 사용한 JSON 처리
aws ec2 describe-instances | jq '.Reservations[].Instances[] | {id: .InstanceId, state: .State.Name}'

# CSV 형식으로 변환
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].{InstanceId:InstanceId,Type:InstanceType,State:State.Name}' \
  --output json | jq -r '.[] | [.InstanceId, .Type, .State] | @csv'
```

---

## 🚀 자동화 스크립팅

### 1. 배치 작업 처리

> [!example] 여러 리소스 한번에 처리
> ```bash
> #!/bin/bash
> # stop-all-dev-instances.sh
> 
> ENVIRONMENT="Development"
> 
> # 개발 환경 인스턴스 목록 가져오기
> INSTANCE_IDS=$(aws ec2 describe-instances \
>   --filters "Name=tag:Environment,Values=$ENVIRONMENT" \
>             "Name=instance-state-name,Values=running" \
>   --query 'Reservations[].Instances[].InstanceId' \
>   --output text)
> 
> if [ -z "$INSTANCE_IDS" ]; then
>   echo "No running instances found in $ENVIRONMENT"
>   exit 0
> fi
> 
> echo "Stopping instances: $INSTANCE_IDS"
> aws ec2 stop-instances --instance-ids $INSTANCE_IDS
> ```

### 2. 대기 조건(Waiters) 활용

```bash
# 인스턴스가 실행될 때까지 대기
aws ec2 wait instance-running --instance-ids i-1234567890abcdef0

# 스택 생성 완료 대기
aws cloudformation wait stack-create-complete --stack-name my-stack

# 커스텀 대기 로직
until aws ec2 describe-instances \
  --instance-ids i-1234567890abcdef0 \
  --query 'Reservations[0].Instances[0].State.Name' \
  --output text | grep -q "running"
do
  echo "Waiting for instance to start..."
  sleep 5
done
```

### 3. 병렬 처리

```bash
#!/bin/bash
# parallel-backup.sh

# GNU Parallel을 사용한 병렬 S3 업로드
find ./backup -type f | parallel -j 4 aws s3 cp {} s3://my-backup-bucket/{/}

# xargs를 사용한 병렬 처리
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].InstanceId' \
  --output text | xargs -n 1 -P 4 -I {} aws ec2 describe-instance-attribute \
  --instance-id {} --attribute instanceType
```

---

## 🛠️ 유용한 명령어 모음

### 1. EC2 관리

```bash
# 모든 리전의 인스턴스 확인
for region in $(aws ec2 describe-regions --query 'Regions[].RegionName' --output text); do
  echo "Region: $region"
  aws ec2 describe-instances --region $region \
    --query 'Reservations[].Instances[].{ID:InstanceId,Type:InstanceType,State:State.Name}' \
    --output table
done

# 사용하지 않는 EBS 볼륨 찾기
aws ec2 describe-volumes \
  --filters "Name=status,Values=available" \
  --query 'Volumes[].{VolumeId:VolumeId,Size:Size,Created:CreateTime}'

# 보안 그룹 규칙 백업
aws ec2 describe-security-groups \
  --query 'SecurityGroups[]' > security-groups-backup-$(date +%Y%m%d).json
```

### 2. S3 작업

```bash
# 버킷 크기 계산
aws s3 ls s3://my-bucket --recursive --human-readable --summarize

# 특정 날짜 이후 수정된 파일 동기화
aws s3 sync . s3://my-bucket/ \
  --exclude "*" \
  --include "*.log" \
  --metadata-directive REPLACE \
  --metadata "backup-date=$(date +%Y-%m-%d)"

# 버킷 간 복사 with 진행률
aws s3 cp s3://source-bucket/ s3://dest-bucket/ \
  --recursive \
  --storage-class GLACIER \
  --metadata-directive COPY
```

### 3. CloudWatch Logs

```bash
# 실시간 로그 테일링
aws logs tail /aws/lambda/my-function --follow

# 로그 검색
aws logs filter-log-events \
  --log-group-name /aws/lambda/my-function \
  --start-time $(date -d '1 hour ago' +%s)000 \
  --filter-pattern "ERROR"

# 로그 스트림 내보내기
aws logs create-export-task \
  --log-group-name /aws/lambda/my-function \
  --from $(date -d '7 days ago' +%s)000 \
  --to $(date +%s)000 \
  --destination my-log-bucket \
  --destination-prefix lambda-logs/
```

---

## ⚡ 성능 최적화

### 1. CLI 성능 설정

```bash
# ~/.aws/config
[default]
cli_pager = 
s3 =
  max_concurrent_requests = 100
  max_queue_size = 10000
  multipart_threshold = 64MB
  multipart_chunksize = 16MB
  max_bandwidth = 100MB/s
  use_accelerate_endpoint = true
  addressing_style = virtual
```

### 2. 페이지네이션 처리

```bash
# 자동 페이지네이션
aws s3api list-objects-v2 \
  --bucket my-bucket \
  --page-size 100 \
  --max-items 1000

# 수동 페이지네이션
NEXT_TOKEN=""
while true; do
  if [ -z "$NEXT_TOKEN" ]; then
    RESPONSE=$(aws ec2 describe-instances --max-results 10)
  else
    RESPONSE=$(aws ec2 describe-instances --max-results 10 --next-token $NEXT_TOKEN)
  fi
  
  # 결과 처리
  echo "$RESPONSE" | jq '.Reservations[].Instances[].InstanceId'
  
  # 다음 토큰 확인
  NEXT_TOKEN=$(echo "$RESPONSE" | jq -r '.NextToken // empty')
  [ -z "$NEXT_TOKEN" ] && break
done
```

---

## 🐛 디버깅과 트러블슈팅

### 1. 디버그 모드

```bash
# 상세 디버그 출력
aws s3 ls --debug

# HTTP 트래픽 확인
aws s3 ls --debug 2>&1 | grep "DEBUG"

# 특정 로그 레벨
export AWS_LOG_LEVEL=debug
aws s3 ls
```

### 2. 공통 문제 해결

> [!warning] 자격 증명 문제
> ```bash
> # 현재 자격 증명 확인
> aws sts get-caller-identity
> 
> # 자격 증명 체인 확인
> aws configure list
> 
> # 환경 변수 초기화
> unset AWS_ACCESS_KEY_ID
> unset AWS_SECRET_ACCESS_KEY
> unset AWS_SESSION_TOKEN
> ```

### 3. 에러 처리

```bash
#!/bin/bash
# 에러 처리 예제

set -euo pipefail  # 에러 시 즉시 종료

# 재시도 로직
MAX_RETRIES=3
RETRY_COUNT=0

while [ $RETRY_COUNT -lt $MAX_RETRIES ]; do
  if aws s3 cp file.txt s3://my-bucket/; then
    echo "Upload successful"
    break
  else
    RETRY_COUNT=$((RETRY_COUNT + 1))
    echo "Upload failed. Retry $RETRY_COUNT/$MAX_RETRIES"
    sleep $((RETRY_COUNT * 2))  # 지수 백오프
  fi
done

if [ $RETRY_COUNT -eq $MAX_RETRIES ]; then
  echo "Upload failed after $MAX_RETRIES attempts"
  exit 1
fi
```

---

## 📚 참고자료

### AWS 공식 문서
- [AWS CLI Command Reference](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html)
- [AWS CLI User Guide](https://docs.aws.amazon.com/cli/latest/userguide/)
- [JMESPath Tutorial](https://jmespath.org/tutorial.html)

### 유용한 도구
- [aws-shell](https://github.com/awslabs/aws-shell) - 인터랙티브 쉘
- [awless](https://github.com/wallix/awless) - 더 나은 AWS CLI
- [aws-vault](https://github.com/99designs/aws-vault) - 자격 증명 관리

### 관련 문서
- [[AWS 기초 가이드]]
- [[IAM 보안 베스트 프랙티스]]
- [[Terraform을 활용한 AWS 인프라 관리]]

---

> [!quote]
> "Automation is not about replacing humans, it's about amplifying human capabilities." - Unknown

> [!success] Pro Tips
> - 자주 사용하는 명령은 별칭으로 등록하세요
> - `--dry-run` 옵션으로 먼저 테스트하세요
> - 출력을 파일로 저장해 버전 관리하세요
> - 민감한 정보는 환경 변수나 AWS Secrets Manager를 사용하세요