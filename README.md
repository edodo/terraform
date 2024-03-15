# terraform
## I terraform 샘플  
1. 테라폼 설치  
```
yum install -y yum-utils
yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
yum -y install terraform
terraform -version
```  
2. 로그인 자격증명 설정  
aws 예)  
검색 > iam  
IAM > 액세스관리 > 사용자 => 사용자 생성 버튼 클릭  
사용자 세부정보 지정 :  
- 사용자 이름 : terraform  
- 권한옵션 : 직접 정책연결  
- 권한정책 : AdministratorAccess 체크박스 체크하기  
다음 > 사용자생성  
  
생성된 사용자 이름 클릭  
- 보안자격증명 > 액세스키 > 액세스키 만들기 클릭  
  
액세스키 만들기  
CommandLineInterface(CLI) 클릭  
확인 체크박스 클릭 > 다음 > 액세스키 만들기 클릭  
액세스키와 비밀 액세스키를 복사해서 아래 설정 정보로 붙여 넣기  
```
export 에이더블유에스_ACCESS_KEY_ID=키복사
export 에이더블유에스_SECRET_ACCESS_KEY=시크릿키 복사
```
위 명령어를 vpc 상에서 실행후 aws 접속  
aws 접속 테스트  
```
aws sc3 ls
```  

3. aws 접속 상태에서 테라폼파일 생성 작업
```
mkdir tf-test && cd $_
vi main.tf
```  
main.tf 파일  생성  
* ami : Amazon Machine Image 운영체제  
* t2.micro : 1core 1Gb 자원  
* resource : aws_instance.example 생성
* 보안그룹 : aws_security_group.instance.id  
```
provider "aws" {
  region = "ap-northeast-2"
}

resource "aws_instance" "example" {
  ami                    = "ami-035da6a0773842f64"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.instance.id]

  user_data = <<-EOF
              #cloud-boothook
              #!/bin/bash
              yum install -y httpd
              systemctl enable --now httpd
              echo "<h1>example</h1>" > /var/www/html/index.html
              EOF

  tags = {
    Name = "terraform-example"
  }
}
```  

보안그룹설정
```

resource "aws_security_group" "instance" {

  name = var.security_group_name

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
  }

  tags = {
    Name = "tf-web"
  }
}

```  

변수 설정  
```
variable "security_group_name" {
  description = "The name of the security group"
  type        = string
  default     = "terraform-example-instance"
}
```  

출력 설정  
```
output "public_ip" {
  value       = aws_instance.example.public_ip
  description = "The public IP of the Instance"
}
```  

테라폼 초기화(설치), 생성, 적용, 삭제  
```
terraform init
terraform plan
terraform apply
 Enter a value : yes

terraform destroy
```


## II aws_set 샘플 작성해 보기  

```
mkdir aws-set && cd $_
vi variables.tf

variable "security_group_name" {
  description = "The name of the security group"
  type        = string
  default     = "terraform-example-instance"
}
```  
variables.tf 파일  
main.tf 파일
```
vi main.tf

provider "aws" {
  region = "ap-northeast-2"
}

### vpc start ###

resource "aws_vpc" "test_vpc" {
  cidr_block  = "192.168.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support = true
  instance_tenancy = "default"

  tags = {
    Name = "test-vpc"
  }
}

data "aws_availability_zones" "available" {
  state = "available"
}

resource "aws_subnet" "test-pub_2a" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.0.0/20"
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.available.names[0]
  tags = {
    Name = "test-pub-2a"
  }
}

resource "aws_subnet" "test-pub_2b" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.16.0/20"
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.available.names[1]
  tags = {
    Name = "test-pub-2b"
  }
}

resource "aws_subnet" "test-pub_2c" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.32.0/20"
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.available.names[2]
  tags = {
    Name = "test-pub-2c"
  }
}

resource "aws_subnet" "test-pub_2d" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.48.0/20"
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.available.names[3]
  tags = {
    Name = "test-pub-2d"
  }
}

resource "aws_subnet" "test-pvt_2a" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.64.0/20"
  availability_zone = data.aws_availability_zones.available.names[0]
  tags = {
    Name = "test-pvt-2a"
  }
}

resource "aws_subnet" "test-pvt_2b" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.80.0/20"
  availability_zone = data.aws_availability_zones.available.names[1]
  tags = {
    Name = "test-pvt-2b"
  }
}

resource "aws_subnet" "test-pvt_2c" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.96.0/20"
  availability_zone = data.aws_availability_zones.available.names[2]
  tags = {
    Name = "test-pvt-2c"
  }
}

resource "aws_subnet" "test-pvt_2d" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.112.0/20"
  availability_zone = data.aws_availability_zones.available.names[3]
  tags = {
    Name = "test-pvt-2d"
  }
}

resource "aws_internet_gateway" "test_igw" {
  vpc_id = aws_vpc.test_vpc.id
  tags = {
    Name = "test-igw"
  }
}

resource "aws_route_table" "test_pub_rtb" {
  vpc_id = aws_vpc.test_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.test_igw.id
  }
  tags = {
    Name = "test-pub-rtb"
  }
}

resource "aws_route_table" "test_pvt_rtb" {
  vpc_id = aws_vpc.test_vpc.id
  tags = {
    Name = "test-pvt-rtb"
  }
}

resource "aws_route_table_association" "test-pub_2a_association" {
  subnet_id = aws_subnet.test-pub_2a.id
  route_table_id = aws_route_table.test_pub_rtb.id
}

resource "aws_route_table_association" "test-pub_2b_association" {
  subnet_id = aws_subnet.test-pub_2b.id
  route_table_id = aws_route_table.test_pub_rtb.id
}

resource "aws_route_table_association" "test-pub_2c_association" {
  subnet_id = aws_subnet.test-pub_2c.id
  route_table_id = aws_route_table.test_pub_rtb.id
}

resource "aws_route_table_association" "test-pub_2d_association" {
  subnet_id = aws_subnet.test-pub_2d.id
  route_table_id = aws_route_table.test_pub_rtb.id
}

resource "aws_route_table_association" "test-pvt_2a_association" {
  subnet_id = aws_subnet.test-pvt_2a.id
  route_table_id = aws_route_table.test_pvt_rtb.id
}

resource "aws_route_table_association" "test-pvt_2b_association" {
  subnet_id = aws_subnet.test-pvt_2b.id
  route_table_id = aws_route_table.test_pvt_rtb.id
}

resource "aws_route_table_association" "test-pvt_2c_association" {
  subnet_id = aws_subnet.test-pvt_2c.id
  route_table_id = aws_route_table.test_pvt_rtb.id
}

resource "aws_route_table_association" "test-pvt_2d_association" {
  subnet_id = aws_subnet.test-pvt_2d.id
  route_table_id = aws_route_table.test_pvt_rtb.id
}

### vpc end ###

### ec2 start ###

resource "aws_instance" "example" {
  ami                    = "ami-035da6a0773842f64"
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.test-pub_2a.id
  vpc_security_group_ids = [aws_security_group.instance.id]
  key_name               = "test-key"
  user_data              = file("user-data.sh")

  tags = {
    Name = "terraform-example"
  }
}

resource "aws_security_group" "instance" {
  vpc_id = aws_vpc.test_vpc.id
  name = var.security_group_name

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["106.253.56.124/32"]
  }
  ingress {
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "terraform-sg"
  }
}

### ec2 end ###

# vi outputs.tf
output "public_ip" {
  value       = aws_instance.example.public_ip
  description = "The public IP of the Instance"
}

output "public_dns" {
  value       = aws_instance.example.public_dns
  description = "The Public dns of the Instance"
}

output "private_ip" {
  value       = aws_instance.example.private_ip
  description = "The Private_ip of the Instance"
}

# terraform init
# terraform plan
# terraform apply (-auto-approve 옵션을 추가하면 yes 생략 가능)
# terraform output public_ip
# terraform destroy (-auto-approve 옵션을 추가하면 yes 생략 가능)

---------------
--- ec2 alb ---
---------------

# mkdir ec2-alb && cd $_
# vi main.tf
provider "aws" {
  region = "ap-northeast-2"
}

### vpc start ###

resource "aws_vpc" "test_vpc" {
  cidr_block  = "192.168.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support = true
  instance_tenancy = "default"

  tags = {
    Name = "test-vpc"
  }
}

data "aws_availability_zones" "available" {
  state = "available"
}

resource "aws_subnet" "test-pub_2a" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.0.0/20"
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.available.names[0]
  tags = {
    Name = "test-pub-2a"
  }
}

resource "aws_subnet" "test-pub_2b" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.16.0/20"
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.available.names[1]
  tags = {
    Name = "test-pub-2b"
  }
}

resource "aws_subnet" "test-pub_2c" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.32.0/20"
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.available.names[2]
  tags = {
    Name = "test-pub-2c"
  }
}

resource "aws_subnet" "test-pub_2d" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.48.0/20"
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.available.names[3]
  tags = {
    Name = "test-pub-2d"
  }
}

resource "aws_subnet" "test-pvt_2a" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.64.0/20"
  availability_zone = data.aws_availability_zones.available.names[0]
  tags = {
    Name = "test-pvt-2a"
  }
}

resource "aws_subnet" "test-pvt_2b" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.80.0/20"
  availability_zone = data.aws_availability_zones.available.names[1]
  tags = {
    Name = "test-pvt-2b"
  }
}

resource "aws_subnet" "test-pvt_2c" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.96.0/20"
  availability_zone = data.aws_availability_zones.available.names[2]
  tags = {
    Name = "test-pvt-2c"
  }
}

resource "aws_subnet" "test-pvt_2d" {
  vpc_id = aws_vpc.test_vpc.id
  cidr_block = "192.168.112.0/20"
  availability_zone = data.aws_availability_zones.available.names[3]
  tags = {
    Name = "test-pvt-2d"
  }
}

resource "aws_internet_gateway" "test_igw" {
  vpc_id = aws_vpc.test_vpc.id
  tags = {
    Name = "test-igw"
  }
}

resource "aws_route_table" "test_pub_rtb" {
  vpc_id = aws_vpc.test_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.test_igw.id
  }
  tags = {
    Name = "test-pub-rtb"
  }
}

resource "aws_route_table" "test_pvt_rtb" {
  vpc_id = aws_vpc.test_vpc.id
  tags = {
    Name = "test-pvt-rtb"
  }
}

resource "aws_route_table_association" "test-pub_2a_association" {
  subnet_id = aws_subnet.test-pub_2a.id
  route_table_id = aws_route_table.test_pub_rtb.id
}

resource "aws_route_table_association" "test-pub_2b_association" {
  subnet_id = aws_subnet.test-pub_2b.id
  route_table_id = aws_route_table.test_pub_rtb.id
}

resource "aws_route_table_association" "test-pub_2c_association" {
  subnet_id = aws_subnet.test-pub_2c.id
  route_table_id = aws_route_table.test_pub_rtb.id
}

resource "aws_route_table_association" "test-pub_2d_association" {
  subnet_id = aws_subnet.test-pub_2d.id
  route_table_id = aws_route_table.test_pub_rtb.id
}

resource "aws_route_table_association" "test-pvt_2a_association" {
  subnet_id = aws_subnet.test-pvt_2a.id
  route_table_id = aws_route_table.test_pvt_rtb.id
}

resource "aws_route_table_association" "test-pvt_2b_association" {
  subnet_id = aws_subnet.test-pvt_2b.id
  route_table_id = aws_route_table.test_pvt_rtb.id
}

resource "aws_route_table_association" "test-pvt_2c_association" {
  subnet_id = aws_subnet.test-pvt_2c.id
  route_table_id = aws_route_table.test_pvt_rtb.id
}

resource "aws_route_table_association" "test-pvt_2d_association" {
  subnet_id = aws_subnet.test-pvt_2d.id
  route_table_id = aws_route_table.test_pvt_rtb.id
}

### vpc end ###

variable "security_group_name" {
  description = "The name of the security group"
  type        = string
  default     = "test-sg-alb"
}

resource "aws_security_group" "test_web_sg_alb" {
  name   = var.security_group_name
#  vpc_id = data.aws_vpc.test_vpc.id
  vpc_id = aws_vpc.test_vpc.id
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "test-sg-alb"
  }
}

resource "aws_lb" "frontend" {
  name               = "alb-example"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.test_web_sg_alb.id]
  subnets            = [
    aws_subnet.test-pub_2a.id,
    aws_subnet.test-pub_2c.id
  ]

  tags = {
    Name = "test-alb"
  }

  lifecycle { create_before_destroy = true }
}


resource "aws_instance" "alb_vm_01" {
  ami                    = "ami-035da6a0773842f64"
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.test-pub_2a.id
  vpc_security_group_ids = [aws_security_group.test_web_sg_alb.id]
  key_name  = "test-key"
  user_data = <<-EOF
              #! /bin/bash
              yum install -y httpd
              systemctl enable --now httpd
              echo "Hello, Terraform01" > /var/www/html/index.html
              EOF

  tags = {
    Name = "ALB01"
  }
}

resource "aws_instance" "alb_vm_02" {
  ami                    = "ami-035da6a0773842f64"
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.test-pub_2c.id
  vpc_security_group_ids = [aws_security_group.test_web_sg_alb.id]
  key_name  = "test-key"
  user_data = <<-EOF
              #! /bin/bash
              yum install -y httpd
              systemctl enable --now httpd
              echo "Hello, Terraform02" > /var/www/html/index.html
              EOF

  tags = {
    Name = "ALB02"
  }
}

resource "aws_lb_target_group" "tg" {
  name        = "TargetGroup"
  port        = 80
  target_type = "instance"
  protocol    = "HTTP"
  vpc_id      = aws_vpc.test_vpc.id

  health_check {
    path                = "/health/"
    protocol            = "HTTP"
    matcher             = "200"
    interval            = 15
    timeout             = 3
    healthy_threshold   = 2
    unhealthy_threshold = 2
  }
}
resource "aws_alb_target_group_attachment" "tgattachment01" {
  target_group_arn = aws_lb_target_group.tg.arn
  target_id        = aws_instance.alb_vm_01.id
  port             = 80
}
resource "aws_alb_target_group_attachment" "tgattachment02" {
  target_group_arn = aws_lb_target_group.tg.arn
  target_id        = aws_instance.alb_vm_02.id
  port             = 80
}

resource "aws_lb_listener" "front_end" {
  load_balancer_arn = aws_lb.frontend.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.tg.arn
  }
}
output "lb_dns_name" {
  description = "The DNS name of the load balancer."
  value       = aws_lb.frontend.dns_name
}

```
테라폼 수행  

terraform init  
terraform validate (구성만 참조하고 원격 상태, 공급자 API 등과 같은 원격 서비스에 액세스하지 않고 디렉터리의 구성 파일을 유효성 검사합니다.)  
terraform plan  
terraform apply -auto-approve  
terraform output lb_dns_name  
terraform destroy -auto-approve  