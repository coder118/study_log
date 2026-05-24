
# 테라폼 코드 예시

    // 테라폼 설정의 시작
    terraform {
    // 필요한 프로바이더(클라우드 서비스 제공자)를 설정
    required_providers {
        // AWS 프로바이더를 사용한다고 선언
        aws = {
        // AWS 프로바이더의 출처를 hashicorp/aws로 지정
        source = "hashicorp/aws"
        }
    }
    }

    // AWS를 제공자로 사용한다고 선언
    provider "aws" {
    // AWS에서 사용할 리전을 변수로부터 받아옴
    region = var.region
    }

    // AWS VPC(Virtual Private Cloud) 리소스를 생성하고 이름을 'vpc_1'로 설정
    resource "aws_vpc" "vpc_1" {
    // VPC의 IP 주소 범위를 설정
    cidr_block = "10.0.0.0/16"

    // DNS 지원을 활성화
    enable_dns_support = true
    // DNS 호스트 이름 지정을 활성화
    enable_dns_hostnames = true

    // 리소스에 대한 태그를 설정
    tags = {
        Name = "${var.prefix}-vpc-1"
    }
    }

    // AWS 서브넷 리소스를 생성하고 이름을 'subnet_1'로 설정
    resource "aws_subnet" "subnet_1" {
    // 이 서브넷이 속할 VPC를 지정. 여기서는 'vpc_1'를 선택
    vpc_id = aws_vpc.vpc_1.id
    // 서브넷의 IP 주소 범위를 설정
    cidr_block = "10.0.1.0/24"
    // 서브넷이 위치할 가용 영역을 설정
    availability_zone = "${var.region}a"
    // 이 서브넷에 배포되는 인스턴스에 공용 IP를 자동으로 할당
    map_public_ip_on_launch = true

    // 리소스에 대한 태그를 설정
    tags = {
        Name = "${var.prefix}-subnet-1"
    }
    }

    // AWS 서브넷 리소스를 생성하고 이름을 'subnet_2'로 설정
    resource "aws_subnet" "subnet_2" {
    // 이 서브넷이 속할 VPC를 지정. 여기서는 'vpc_1'를 선택
    vpc_id = aws_vpc.vpc_1.id
    // 서브넷의 IP 주소 범위를 설정
    cidr_block = "10.0.2.0/24"
    // 서브넷이 위치할 가용 영역을 설정
    availability_zone = "${var.region}b"
    // 이 서브넷에 배포되는 인스턴스에 공용 IP를 자동으로 할당
    map_public_ip_on_launch = true

    // 리소스에 대한 태그를 설정
    tags = {
        Name = "${var.prefix}-subnet-2"
    }
    }

    // AWS 인터넷 게이트웨이 리소스를 생성하고 이름을 'igw_1'로 설정
    resource "aws_internet_gateway" "igw_1" {
    // 이 인터넷 게이트웨이가 연결될 VPC를 지정. 여기서는 'vpc_1'를 선택
    vpc_id = aws_vpc.vpc_1.id

    // 리소스에 대한 태그를 설정
    tags = {
        Name = "${var.prefix}-igw-1"
    }
    }


    // AWS 라우트 테이블 리소스를 생성하고 이름을 'rt_1'로 설정
    resource "aws_route_table" "rt_1" {
    // 이 라우트 테이블이 속할 VPC를 지정. 여기서는 'vpc_1'를 선택
    vpc_id = aws_vpc.vpc_1.id

    // 라우트 규칙을 설정. 여기서는 모든 트래픽(0.0.0.0/0)을 'igw_1' 인터넷 게이트웨이로 보냄
    route {
        cidr_block = "0.0.0.0/0"
        gateway_id = aws_internet_gateway.igw_1.id
    }

    // 리소스에 대한 태그를 설정
    tags = {
        Name = "${var.prefix}-rt-1"
    }
    }

    // 라우트 테이블 'rt_1'과 서브넷 'subnet_1'을 연결
    resource "aws_route_table_association" "association_1" {
    // 연결할 서브넷을 지정
    subnet_id = aws_subnet.subnet_1.id
    // 연결할 라우트 테이블을 지정
    route_table_id = aws_route_table.rt_1.id
    }

    // 라우트 테이블 'rt_1'과 서브넷 'subnet_2'을 연결
    resource "aws_route_table_association" "association_2" {
    // 연결할 서브넷을 지정
    subnet_id = aws_subnet.subnet_2.id
    // 연결할 라우트 테이블을 지정
    route_table_id = aws_route_table.rt_1.id
    }

    // AWS 보안 그룹 리소스를 생성하고 이름을 'sg_1'로 설정
    resource "aws_security_group" "sg_1" {
    // 보안 그룹의 이름을 설정. 이름 앞에는 변수로부터 받은 prefix를 붙임
    name = "${var.prefix}-sg-1"

    // 인바운드 트래픽 규칙을 설정
    // 여기서는 모든 프로토콜, 모든 포트에 대해 모든 IP(0.0.0.0/0)로부터의 트래픽을 허용
    ingress {
        from_port = 0
        to_port   = 0
        protocol  = "all"
        cidr_blocks = ["0.0.0.0/0"]
    }

    // 아웃바운드 트래픽 규칙을 설정
    // 여기서는 모든 프로토콜, 모든 포트에 대해 모든 IP(0.0.0.0/0)로의 트래픽을 허용
    egress {
        from_port = 0
        to_port   = 0
        protocol  = "all"
        cidr_blocks = ["0.0.0.0/0"]
    }

    // 이 보안 그룹이 속할 VPC를 지정. 여기서는 'vpc_1'를 선택
    vpc_id = aws_vpc.vpc_1.id

    // 리소스에 대한 태그를 설정
    tags = {
        Name = "${var.prefix}-sg-1"
    }
    }

# AWS 보안 그룹(Security Group)

EC2 인스턴스에 대한 인바운드 및 아웃바운드 트래픽을 제어하는 가상 방화벽입니다

- 위 코드에서는 모든 트래픽을 허용하는 보안 그룹을 생성합니다

> ### 인바운드 규칙
 모든 IP(0.0.0.0/0)로부터 모든 프로토콜과 포트에 대한 접근을 허용
> ### 아웃바운드 규칙
모든 IP(0.0.0.0/0)로 모든 프로토콜과 포트에 대한 접근을 허용
<br>
<주의>실제 프로덕션 환경에서는 보안을 위해 필요한 포트와 IP만 허용하는 것이 좋습니다

## AWS 보안 그룹의 종류 (EC2 수준/ VPC 수준)

1. #### EC2 수준의 검문소: 보안 그룹 (Security Group)

서버(EC2 인스턴스) 바로 앞에 세우는 경비원입니다.
<br>
테라폼 코드 맨 밑의 aws_security_group이 보안 그룹이다. 이 경비원은 특정 서버 하나하나에 직접 옷처럼 입혀주는 방화벽이다.
<br>
- 작동 방식: 서버 바로 문앞에서 지키고 서서, 들어오는 패킷(트래픽)을 검사.
<br>
> 현재 코드 상태
코드를 보면 ingress (들어오는 문)와 egress (나가는 문)에 from_port = 0, to_port = 0, protocol = "all", cidr_blocks = ["0.0.0.0/0"]이라고 되어있다. 이게 무슨 뜻이냐면 "포트 번호 상관없이 지구상의 모든 컴퓨터가 다 들어오고 나가게 문을 활짝 열어라"라는 뜻입니다. (실제 서비스에서는 매우 위험한 상태입니다.)

2. #### VPC 수준의 검문소: 네트워크 ACL (NACL)

네트워크 ACL(NACL)은 서브넷(동네/방) 입구에 세우는 대형 바리케이드입니다.
<br>

지금 테라폼 코드에는 생성하는 코드가 없지만, AWS VPC를 만들면 **기본적으로 자동으로** 만들어져서 작동하고 있다.

- 작동 방식: 개별 서버가 아니라, subnet_1, subnet_2 같은 서브넷이라는 마을 단위의 울타리 입구에서 지키고 서 있다.
<br>

아무리 EC2 수준(보안 그룹)에서 문을 활짝 열어두어도, 이 VPC 수준(NACL)에서 특정 포트를 막아버리면 그 서브넷 안에 있는 모든 서버는 해당 포트로 통신할 수 없다. 동네 입구 톨게이트에서 차단당하는 느낌이다.


### 아파트 비유로 한눈에 이해하기

- 인터넷 공간: 아파트 단지 바깥세상

- VPC 수준 (NACL): 아파트 단지 정문에 있는 종합 경비 초소. (여기서 막히면 아파트 단지 땅 안으로 아예 들어가지도 못함 -> 서브넷 단위 차단)

- EC2 수준 (보안 그룹): 아파트 단지를 지나 내가 찾아간 '우리 집 현관문 도어락'. (정문 초소를 통과했어도 현관문 비번(포트 허용)을 모르면 집 안(EC2 서버)으로 못 들어감)


# 포트(Port)란 무엇일까요?

컴퓨터(서버)는 수많은 일을 동시에 할 수 있는 멀티태스킹 장치. 그래서 서버에는 외부와 통신할 수 있는 '가상의 문'이 65,535개나 뚫려 있다. 이 문 번호를 포트(Port)라고 부른다.
인터넷 세상에서는 서비스 종류마다 **쓰는 문 번호가 약속**되어 있습니다.

- 80번, 443번 문: 웹사이트 서핑을 위한 문 (네이버, 구글 접속 등)

- 22번 문: 서버 관리자가 원격으로 로그인해서 리눅스 명령어를 입력하는 문 (매우 중요하고 위험한 문)

- 3306번 문: 데이터베이스(DB)에 접근하는 문

## "포트로 보안을 지킨다"의 실제 예시

예를 들어, 내가 이 AWS 인프라에 웹사이트(블로그나 쇼핑몰)를 하나 만들어서 띄웠다고 가정.

1. 원하는 상황: 전 세계 모든 사람들이 내 웹사이트에 접속해서 구경하면 좋겠음. (80번, 443번 문은 열어둠)

2. 경계하는 상황: 해커들이 내 서버를 장악하기 위해 관리자 로그인 문(22번)을 마구 두드리는 것은 막고 싶음. 관리자인 나만 집에서 안전하게 들어가고 싶음.

이때 보안 그룹(EC2 수준 방화벽)을 사용해 다음과 같이 규칙을 세우는 것이다.
<br>
<br>

- 올바른 보안 설정 예시
    - 규칙 1 (웹 서핑 허용): 목적지 포트 80, 443은 IP 주소 0.0.0.0/0 (전 세계 누구나) 들어오게 허용해라.

    - 규칙 2 (관리자 로그인 통제): 목적지 포트 22는 오직 관리자의 집 IP (예: 211.234.xx.xx) 만 들어오게 허용하고, 나머지는 싹 다 차단해라.