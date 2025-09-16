---
tags:
- aws
- ec2
- terraform
- compute
- vpc
- infrastructure
created: 2025-01-15
updated: 2025-01-15
aliases:
- AWS EC2
- Elastic Compute Cloud
- EC2 Terraform
description: AWS EC2(Elastic Compute Cloud) 개념과 Terraform을 사용한 EC2 인스턴스 및 VPC 관리
status: published
category: guide
---

# AWS EC2 with Terraform

> [!info] 개요
> AWS EC2(Elastic Compute Cloud)는 클라우드에서 크기 조정 가능한 컴퓨팅 용량을 제공하는 웹 서비스입니다. 이 문서에서는 Terraform을 사용하여 EC2 인스턴스와 관련 네트워크 리소스를 구성하고 관리하는 방법을 다룹니다.

## 📑 목차

- [[#☁️ EC2 개요]]
- [[#🏗️ VPC와 네트워킹 기초]]
- [[#🖥️ EC2 인스턴스 생성]]
- [[#🔐 보안 그룹 설정]]
- [[#💾 스토리지 관리]]
- [[#🌐 네트워크 구성]]
- [[#💰 가격 정책]]
- [[#📊 실전 예제]]
- [[#📚 참고자료]]

---

## ☁️ EC2 개요

### EC2란?

> [!note] EC2 정의
> EC2는 AWS에서 제공하는 **가상 서버 서비스**로, 필요에 따라 컴퓨팅 리소스를 늘리거나 줄일 수 있는 탄력적인 컴퓨팅 환경을 제공합니다.

### EC2 핵심 기능

#### Virtual Computing Environments (Instances)
- 다양한 운영체제와 애플리케이션 실행
- CPU, 메모리, 스토리지, 네트워크 용량 선택 가능

#### Preconfigured Templates (AMIs)
- Amazon Machine Images를 통한 빠른 인스턴스 배포
- 사전 구성된 OS 및 소프트웨어 포함

#### Secure Login Information
- Key Pair를 통한 안전한 SSH 접근
- AWS가 공개 키 저장, 사용자가 개인 키 보관

#### Storage Volumes
- **Instance Store**: 임시 저장소 (인스턴스 종료 시 데이터 삭제)
- **EBS (Elastic Block Store)**: 영구 저장소

#### Additional Features
- **Multiple Physical Locations**: 리전과 가용 영역
- **Firewall Settings**: Security Groups를 통한 네트워크 보안
- **Elastic IP Addresses**: 고정 공인 IP
- **Metadata**: 태그를 통한 리소스 관리
- **Virtual Private Clouds (VPCs)**: 격리된 네트워크 환경

---

## 🏗️ VPC와 네트워킹 기초

### VPC 아키텍처 구성 요소

> [!tip] VPC 계층 구조
> ```
> Region (지역)
> └── VPC (Virtual Private Cloud)
>     ├── Internet Gateway
>     ├── Availability Zone A
>     │   ├── Public Subnet
>     │   ├── Private Subnet
>     │   └── NAT Gateway
>     └── Availability Zone B
>         ├── Public Subnet
>         └── Private Subnet
> ```

### 네트워크 구성 요소 설명

#### VPC (Virtual Private Cloud)
- AWS 클라우드 내 논리적으로 격리된 네트워크
- CIDR 블록으로 IP 대역 정의 (예: 10.0.0.0/16)

#### Subnet
- VPC 내 IP 주소 범위
- Public Subnet: 인터넷 게이트웨이와 직접 연결
- Private Subnet: NAT Gateway를 통해 인터넷 접근

#### Internet Gateway
- VPC와 인터넷 간 통신 활성화
- Public Subnet의 인스턴스에 인터넷 연결 제공

#### NAT Gateway
- Private Subnet의 인스턴스가 인터넷으로 아웃바운드 연결
- 인바운드 연결은 차단

---

## 🖥️ EC2 인스턴스 생성

### 기본 VPC 및 네트워크 설정

```hcl
# provider.tf
provider "aws" {
  region = "us-west-2"
}

# vpc.tf
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "main-vpc"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-igw"
  }
}

# Public Subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-west-2a"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet"
  }
}

# Private Subnet
resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.10.0/24"
  availability_zone = "us-west-2a"

  tags = {
    Name = "private-subnet"
  }
}

# Route Table for Public Subnet
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "public-route-table"
  }
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

### EC2 인스턴스 생성

```hcl
# data.tf - AMI 조회
data "aws_ami" "amazon_linux_2" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# key_pair.tf
resource "aws_key_pair" "deployer" {
  key_name   = "deployer-key"
  public_key = file("~/.ssh/id_rsa.pub")
}

# ec2.tf
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux_2.id
  instance_type = "t2.micro"
  
  subnet_id                   = aws_subnet.public.id
  vpc_security_group_ids      = [aws_security_group.web.id]
  associate_public_ip_address = true
  key_name                    = aws_key_pair.deployer.key_name

  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Hello from Terraform EC2</h1>" > /var/www/html/index.html
  EOF

  tags = {
    Name        = "web-server"
    Environment = "Development"
  }
}

# Elastic IP (선택사항)
resource "aws_eip" "web" {
  instance = aws_instance.web.id
  domain   = "vpc"

  tags = {
    Name = "web-eip"
  }
}
```

---

## 🔐 보안 그룹 설정

### Security Group 구성

> [!warning] 보안 그룹 중요성
> Security Group은 EC2 인스턴스의 가상 방화벽 역할을 합니다. 인바운드와 아웃바운드 트래픽을 제어합니다.

```hcl
# security_group.tf
resource "aws_security_group" "web" {
  name        = "web-security-group"
  description = "Security group for web server"
  vpc_id      = aws_vpc.main.id

  # SSH 접근 (제한된 IP에서만)
  ingress {
    description = "SSH from admin IP"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["YOUR_IP/32"]  # 본인의 IP로 변경
  }

  # HTTP 접근
  ingress {
    description = "HTTP from anywhere"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # HTTPS 접근
  ingress {
    description = "HTTPS from anywhere"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # 모든 아웃바운드 트래픽 허용
  egress {
    description = "All outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "web-sg"
  }
}

# Database Security Group (Private Subnet용)
resource "aws_security_group" "database" {
  name        = "database-security-group"
  description = "Security group for database"
  vpc_id      = aws_vpc.main.id

  # Web 서버에서만 접근 허용
  ingress {
    description     = "MySQL/Aurora from web server"
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.web.id]
  }

  egress {
    description = "All outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "database-sg"
  }
}
```

---

## 💾 스토리지 관리

### EBS 볼륨 추가

```hcl
# ebs.tf
resource "aws_ebs_volume" "data" {
  availability_zone = aws_instance.web.availability_zone
  size              = 10
  type              = "gp3"
  encrypted         = true

  tags = {
    Name = "web-data-volume"
  }
}

resource "aws_volume_attachment" "data_attach" {
  device_name = "/dev/sdf"
  volume_id   = aws_ebs_volume.data.id
  instance_id = aws_instance.web.id
}
```

### Root Block Device 설정

```hcl
resource "aws_instance" "app" {
  ami           = data.aws_ami.amazon_linux_2.id
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.private.id

  # Root 볼륨 설정
  root_block_device {
    volume_type           = "gp3"
    volume_size           = 20
    iops                  = 3000
    throughput            = 125
    encrypted             = true
    delete_on_termination = true
    
    tags = {
      Name = "app-root-volume"
    }
  }

  tags = {
    Name = "app-server"
  }
}
```

---

## 🌐 네트워크 구성

### NAT Gateway 설정 (Private Subnet용)

```hcl
# nat.tf
resource "aws_eip" "nat" {
  domain = "vpc"

  tags = {
    Name = "nat-eip"
  }
}

resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public.id

  tags = {
    Name = "main-nat-gateway"
  }

  depends_on = [aws_internet_gateway.main]
}

# Private Subnet Route Table
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }

  tags = {
    Name = "private-route-table"
  }
}

resource "aws_route_table_association" "private" {
  subnet_id      = aws_subnet.private.id
  route_table_id = aws_route_table.private.id
}
```

### 다중 가용 영역 구성

```hcl
# variables.tf
variable "availability_zones" {
  default = ["us-west-2a", "us-west-2b"]
}

variable "public_subnet_cidrs" {
  default = ["10.0.1.0/24", "10.0.2.0/24"]
}

variable "private_subnet_cidrs" {
  default = ["10.0.10.0/24", "10.0.11.0/24"]
}

# subnets.tf
resource "aws_subnet" "public_multi" {
  count                   = length(var.public_subnet_cidrs)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-${count.index + 1}"
    Type = "Public"
  }
}

resource "aws_subnet" "private_multi" {
  count             = length(var.private_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name = "private-subnet-${count.index + 1}"
    Type = "Private"
  }
}
```

---

## 💰 가격 정책

### EC2 인스턴스 타입과 가격

> [!tip] 인스턴스 타입 선택 가이드
> 
> | 인스턴스 | 시간당 요금 | vCPU | 메모리 | 스토리지 | 네트워크 성능 |
> |----------|------------|------|--------|----------|--------------|
> | t4g.nano | $0.0042 | 2 | 0.5 GiB | EBS Only | Up to 5 Gbps |
> | t3.micro | $0.0104 | 2 | 1 GiB | EBS Only | Up to 5 Gbps |
> | t3.small | $0.0208 | 2 | 2 GiB | EBS Only | Up to 5 Gbps |
> | t3.medium | $0.0416 | 2 | 4 GiB | EBS Only | Up to 5 Gbps |
> | m5.large | $0.096 | 2 | 8 GiB | EBS Only | Up to 10 Gbps |
> | c5.large | $0.085 | 2 | 4 GiB | EBS Only | Up to 10 Gbps |

### 비용 최적화 전략

```hcl
# Spot Instance 사용 예제
resource "aws_instance" "spot" {
  ami           = data.aws_ami.amazon_linux_2.id
  instance_type = "t3.micro"
  
  instance_market_options {
    market_type = "spot"
    
    spot_options {
      max_price          = "0.0050"  # 최대 입찰가
      spot_instance_type = "one-time"
    }
  }
  
  tags = {
    Name = "spot-instance"
  }
}
```

---

## 📊 실전 예제

### 웹 서버와 데이터베이스 구성

> [!example] 2-Tier 아키텍처
> Public Subnet에 웹 서버, Private Subnet에 데이터베이스를 배치하는 일반적인 구성

```hcl
# Web Server in Public Subnet
resource "aws_instance" "web_tier" {
  count         = 2
  ami           = data.aws_ami.amazon_linux_2.id
  instance_type = "t3.micro"
  
  subnet_id                   = aws_subnet.public_multi[count.index].id
  vpc_security_group_ids      = [aws_security_group.web.id]
  associate_public_ip_address = true
  key_name                    = aws_key_pair.deployer.key_name

  user_data = templatefile("${path.module}/user_data.sh", {
    instance_number = count.index + 1
  })

  tags = {
    Name = "web-server-${count.index + 1}"
    Tier = "Web"
  }
}

# Database Server in Private Subnet
resource "aws_instance" "db_tier" {
  ami           = data.aws_ami.amazon_linux_2.id
  instance_type = "t3.small"
  
  subnet_id              = aws_subnet.private_multi[0].id
  vpc_security_group_ids = [aws_security_group.database.id]
  key_name               = aws_key_pair.deployer.key_name

  root_block_device {
    volume_size = 30
    volume_type = "gp3"
    encrypted   = true
  }

  tags = {
    Name = "database-server"
    Tier = "Database"
  }
}

# Application Load Balancer
resource "aws_lb" "web" {
  name               = "web-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets           = aws_subnet.public_multi[*].id

  tags = {
    Name = "web-alb"
  }
}

resource "aws_lb_target_group" "web" {
  name     = "web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    enabled             = true
    healthy_threshold   = 2
    unhealthy_threshold = 2
    timeout             = 5
    interval            = 30
    path                = "/"
    matcher             = "200"
  }
}

resource "aws_lb_target_group_attachment" "web" {
  count            = length(aws_instance.web_tier)
  target_group_arn = aws_lb_target_group.web.arn
  target_id        = aws_instance.web_tier[count.index].id
  port             = 80
}

resource "aws_lb_listener" "web" {
  load_balancer_arn = aws_lb.web.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}
```

### Output 값 정의

```hcl
# outputs.tf
output "web_server_public_ips" {
  description = "Public IP addresses of web servers"
  value       = aws_instance.web_tier[*].public_ip
}

output "load_balancer_dns" {
  description = "DNS name of the load balancer"
  value       = aws_lb.web.dns_name
}

output "database_private_ip" {
  description = "Private IP of database server"
  value       = aws_instance.db_tier.private_ip
}

output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}
```

---

## 📚 참고자료

### 관련 문서
- [[AWS 기초 개념]]
- [[AWS S3 with Terraform]]
- [[Terraform 기초]]
- [[11. Terraform 프로바이더]]

### 외부 리소스
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [EC2 Pricing](https://aws.amazon.com/ec2/pricing/on-demand/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

---

> [!quote]
> "EC2는 단순한 가상 서버가 아니라, 무한한 확장 가능성을 가진 컴퓨팅 플랫폼입니다."