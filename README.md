# 🚀 Terraform을 이용한 AWS S3 정적 웹 호스팅 프로젝트


## 🌟 프로젝트 개요

이 프로젝트는 Terraform을 사용하여 AWS S3 버킷을 생성하고, 정적 웹 호스팅을 설정하는 실습입니다. 처음에는 S3와 EC2 리소스를 한 번에 설정하고, 이후 실습에서는 모듈화를 통해 개별 리소스를 관리하는 방식으로 확장했습니다.

## 🎯 학습 목표

- Terraform을 사용한 AWS 리소스 관리 방법 습득
- S3 버킷 생성 및 정적 웹 호스팅 설정 방법 이해
- Terraform 코드의 모듈화 및 개별 리소스 관리 학습
- 실제 프로젝트 진행 시 발생할 수 있는 문제 해결 능력 향상

## 🧠 주요 개념 설명

### Terraform
Terraform은 인프라를 코드로 관리하는 도구입니다. 다양한 클라우드 제공자의 리소스를 선언적 구성 파일을 통해 정의하고 관리할 수 있게 해줍니다. 이 프로젝트에서는 Terraform을 사용하여 AWS 리소스를 생성하고 관리합니다.

### AWS S3 (Simple Storage Service)
S3는 AWS에서 제공하는 객체 스토리지 서비스입니다. 이 프로젝트에서는 S3를 사용하여 정적 웹사이트를 호스팅합니다. 주요 개념은 다음과 같습니다:

- **Bucket**: 객체를 저장하는 컨테이너
- **Object**: S3에 저장되는 파일
- **정적 웹사이트 호스팅**: S3 버킷을 웹서버처럼 사용하여 정적 파일을 제공

### AWS EC2 (Elastic Compute Cloud)
EC2는 AWS의 가상 서버 서비스입니다. 이 프로젝트에서는 EC2 인스턴스를 생성하여 필요한 경우 S3에 호스팅된 웹사이트와 상호작용할 수 있게 합니다. 주요 개념은 다음과 같습니다:

- **Instance**: 가상 서버
- **Security Group**: 인스턴스에 대한 트래픽을 제어하는 가상 방화벽
- **AMI (Amazon Machine Image)**: 인스턴스를 시작하는 데 필요한 정보를 제공하는 템플릿

## 🛠 사전 요구사항

- AWS 계정
- Terraform 설치 (버전 0.12 이상)
- AWS CLI 설치 및 구성

## 📂 프로젝트 구조

초기 실습:
```
.
├── resource.tf     # S3 버킷 관련 리소스 정의
├── ec2.tf          # EC2 인스턴스 관련 리소스 정의
├── output.tf       # 출력 값 정의
├── index.html      # S3에 업로드할 샘플 HTML 파일
└── main.html       # S3에 업로드할 샘플 HTML 파일
```

모듈화된 추가 실습:
```
.
├── bucket.tf      # S3 버킷 및 관련 설정
├── index.tf       # index.html 업로드 설정
├── new.tf         # new_page.html 업로드 설정
├── update.tf      # 기존 index.html 업데이트 설정
├── index.html     # 메인 페이지 HTML
└── new_page.html  # 새로운 페이지 HTML
```

## 📝 초기 실습 과정

## 💻 코드

### resource.tf
```hcl
# S3 버킷 생성
resource "aws_s3_bucket" "bucket1" {
  bucket = "xce12"  # 생성하고자 하는 S3 버킷 이름
}

# S3 버킷의 public access block 설정
resource "aws_s3_bucket_public_access_block" "bucket1_public_access_block" {
  bucket = aws_s3_bucket.bucket1.id
  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

# 이미 존재하는 S3 버킷에 index.html 파일을 업로드
resource "aws_s3_object" "index" {
  bucket        = aws_s3_bucket.bucket1.id  # 생성된 S3 버킷 이름 사용
  key           = "index.html"
  source        = "index.html"
  content_type  = "text/html"
}

# S3 버킷의 웹사이트 호스팅 설정
resource "aws_s3_bucket_website_configuration" "xweb_bucket_website" {
  bucket = aws_s3_bucket.bucket1.id  # 생성된 S3 버킷 이름 사용
  index_document {
    suffix = "index.html"
  }
}

# S3 버킷의 public read 정책 설정
resource "aws_s3_bucket_policy" "public_read_access" {
  bucket = aws_s3_bucket.bucket1.id  # 생성된 S3 버킷 이름 사용
  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": [ "s3:GetObject" ],
      "Resource": [
        "arn:aws:s3:::xce12",
        "arn:aws:s3:::xce12/*"
      ]
    }
  ]
}
EOF
}
```

### ec2.tf
```hcl
# EC2 인스턴스 생성을 위한 보안 그룹
resource "aws_security_group" "ce12_ec2_sg" {
  name        = "ce12-ec2-sg"
  description = "Security group for ce12-ec2 instance"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# EC2 인스턴스 생성
resource "aws_instance" "ce12_ec2" {
  ami           = "ami-12345678901234567"  # 이 부분을 실제 AMI ID로 교체하세요
  instance_type = "t2.micro"
  key_name      = "your-key-pair-name"  # 사용할 키 페어 이름으로 변경하세요

  vpc_security_group_ids = [aws_security_group.ce12_ec2_sg.id]

  tags = {
    Name = "ce12-ec2"
  }
}
```

### output.tf
```hcl
output "website_endpoint" {
  value = aws_s3_bucket.bucket1.website_endpoint
  description = "The endpoint for the S3 bucket website."
}

output "ce12_ec2_public_ip" {
  value       = aws_instance.ce12_ec2.public_ip
  description = "The public IP address of the ce12-ec2 instance"
}
```

### index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>Index Page</title>
</head>
<body>
   <h1>ce12</h1>
   <p>This is the index page of the xce12 website.</p>
   <a href="main.html">Go to main page</a>
</body>
</html>
```

### main.html (페이지 이동 실습, 선택사항)
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Main Page</title>
</head>
<body>
    <h1>Welcome to the Main Page</h1>
    <p>This is the main page of the xce12 website.</p>
    <a href="index.html">Go back to index</a>
</body>
</html>
```

## 실행
```hcl
terraform init
terraform plan
terraform apply -auto-approve
```

## 웹에서 확인
![image](https://github.com/user-attachments/assets/715a38c6-53c3-490c-8a26-eca6ca4c732e)
![image](https://github.com/user-attachments/assets/9e5d9b20-0801-4f0c-8cb5-47a0aaee6a74)


## 🔧 문제 해결

### AMI ID 오류
AMI ID가 존재하지 않는 경우 다음 단계를 따르세요:
1. AWS CLI를 사용하여 최신 Amazon Linux 2 AMI ID 조회:
   ```
   aws ec2 describe-images --owners amazon --filters "Name=name,Values=amzn2-ami-hvm-2.0.????????.?-x86_64-gp2" "Name=state,Values=available" --query "reverse(sort_by(Images, &CreationDate))[:1].ImageId" --output text
   ```
2. 반환된 AMI ID로 `ec2.tf` 파일 업데이트

## 🔒 보안 고려사항

- S3 버킷의 퍼블릭 액세스 설정을 신중히 관리하세요.
- EC2 보안 그룹의 인바운드 규칙을 필요한 포트로만 제한하세요.
- IAM 역할과 정책을 최소 권한 원칙에 따라 설정하세요.
- 민감한 정보(예: AWS 크레덴셜)를 코드에 직접 포함시키지 마세요.

## 🚀 향후 개선사항

- Terraform 모듈화를 통한 코드 재사용성 향상
- CloudFront를 통한 콘텐츠 전송 최적화
- 자동화된 CI/CD 파이프라인 구축
- 다중 환경(개발, 스테이징, 프로덕션) 설정

## 📝 모듈화된 추가 실습 과정

이 실습에서는 각 리소스를 개별 파일로 분리하여 모듈화했습니다. 이를 통해 각 부분을 독립적으로 관리하고 적용할 수 있습니다.

### 1. S3 버킷 설정 (bucket.tf)

```hcl
data "aws_s3_bucket" "existing_bucket" {
  bucket = "ce12-new-bucket"
}

resource "aws_s3_bucket_public_access_block" "bucket_public_access" {
  bucket = data.aws_s3_bucket.existing_bucket.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_website_configuration" "bucket_website" {
  bucket = data.aws_s3_bucket.existing_bucket.id

  index_document {
    suffix = "index.html"
  }
}

resource "aws_s3_bucket_policy" "bucket_policy" {
  bucket = data.aws_s3_bucket.existing_bucket.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "PublicReadGetObject"
        Effect    = "Allow"
        Principal = "*"
        Action    = "s3:GetObject"
        Resource  = "${data.aws_s3_bucket.existing_bucket.arn}/*"
      },
    ]
  })
}
```

### 2. index.html 업로드 설정 (index.tf)

```hcl
resource "aws_s3_object" "index" {
  bucket       = data.aws_s3_bucket.existing_bucket.id
  key          = "index.html"
  source       = "index.html"
  content_type = "text/html"
  etag         = filemd5("index.html")
}
```

### 3. new_page.html 업로드 설정 (new.tf)

```hcl
resource "aws_s3_object" "new_page" {
  bucket       = data.aws_s3_bucket.existing_bucket.id
  key          = "new_page.html"
  source       = "new_page.html"
  content_type = "text/html"
  etag         = filemd5("new_page.html")
}
```

### 4. 기존 index.html 업데이트 설정 (update.tf)

```hcl
resource "aws_s3_object" "updated_index" {
  bucket       = data.aws_s3_bucket.existing_bucket.id
  key          = "index.html"
  source       = "updated_index.html"
  content_type = "text/html"
  etag         = filemd5("updated_index.html")
}
```

### 5. 개별 적용 방법

각 부분을 개별적으로 적용하려면 다음과 같은 명령어를 사용합니다:

```bash
# 버킷 설정 적용
terraform apply -target=data.aws_s3_bucket.existing_bucket -target=aws_s3_bucket_public_access_block.bucket_public_access -target=aws_s3_bucket_website_configuration.bucket_website -target=aws_s3_bucket_policy.bucket_policy

# index.html 업로드
terraform apply -target=aws_s3_object.index

# new_page.html 업로드
terraform apply -target=aws_s3_object.new_page

# 기존 index.html 업데이트
terraform apply -target=aws_s3_object.updated_index
```

이 방식을 통해 각 리소스를 독립적으로 관리하고 적용할 수 있습니다.

## 🚨 트러블슈팅

1. **리소스 중복 정의**
   - 문제: `Error: Duplicate resource "aws_s3_object" configuration`
   - 해결: 리소스를 개별 파일로 분리하여 중복 정의 방지

2. **파일 없음 오류**
   - 문제: `Error: open updated_index.html: no such file or directory`
   - 해결: 파일 존재 여부 확인 후 리소스 정의, 필요시 주석 처리

3. **부분 적용 시 의존성 문제**
   - 문제: 일부 리소스만 적용 시 의존성 있는 다른 리소스 미적용
   - 해결: `-target` 옵션 사용 시 관련 리소스 모두 지정

## 📝 결론 및 학습 내용 정리

이 프로젝트를 통해 다음과 같은 내용을 학습했습니다:

1. Terraform을 사용한 AWS 리소스 관리 기본 원리
2. S3 버킷 생성 및 정적 웹 호스팅 설정 방법
3. Terraform 코드의 모듈화 및 개별 리소스 관리 기법
4. 실제 프로젝트 진행 시 발생할 수 있는 문제 해결 능력

초기 실습에서는 전체 설정을 한 번에 적용하는 방법을, 추가 실습에서는 각 리소스를 모듈화하여 개별적으로 관리하는 방법을 학습했습니다. 이를 통해 Terraform의 유연성과 다양한 관리 방법을 경험할 수 있었습니다.

주요 학습 포인트:
- Terraform 코드 모듈화의 장점
- 개별 리소스 관리를 통한 유연한 인프라 관리
- `-target` 옵션을 활용한 부분 적용 방법
- 리소스 간 의존성 관리의 중요성

이 프로젝트는 Terraform과 AWS S3를 활용한 기본적인 인프라 관리 기술을 습득하는 데 도움이 되었습니다. 특히 모듈화된 접근 방식은 대규모 프로젝트에서 유용하게 활용될 수 있을 것입니다.
