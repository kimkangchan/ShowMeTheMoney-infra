# Team4 1단계 프로젝트 리소스 네이밍 컨벤션 명세

## 1. 문서 목적

이 문서는 Team4의 1단계 프로젝트에서 생성하는 AWS 인프라, Docker 리소스 및 S3 관리 규칙의 이름을 일관되게 정의한다. 모든 팀원은 AWS 콘솔, Docker 실행 설정, 애플리케이션 환경변수, 구축 문서와 제출 증적에서 이 문서의 이름을 동일하게 사용한다.

본 문서는 Team4 리소스 이름의 기준 문서다. 실제 환경과 문서가 다를 경우 임의로 새 이름을 만들지 않고, 팀 합의 후 이 문서와 관련 설정을 함께 변경한다.

## 2. 적용 범위

이 명세는 다음 항목에 적용한다.

- VPC, Subnet, Internet Gateway, Route Table
- EC2, Elastic IP, EBS, Key Pair
- EC2 및 RDS Security Group
- IAM Role
- RDS와 DB Subnet Group
- S3 Bucket, Versioning, Lifecycle Rule
- Docker Network와 Container
- 외부 접속 도메인 표기

AWS가 자동으로 부여하는 `vpc-...`, `subnet-...`, `sg-...`, `rtb-...` 등의 물리 리소스 ID는 네이밍 대상에 포함하지 않는다.

## 3. 프로젝트 기준값

| 구분 | 값 | 비고 |
| --- | --- | --- |
| Project prefix | `team4` | 모든 팀 관리 리소스 이름의 시작값 |
| AWS Region | `ap-northeast-2` | Asia Pacific (Seoul) |
| VPC CIDR | `10.21.0.0/16` | Team4 전용 주소 범위 |
| Initial database | `showmethemoney` | 기존 애플리케이션 설정과 일치 |
| Docker network | `team4-network` | Frontend와 Backend 내부 통신 |
| Common tag | `Team=team4` | 공통 태그는 이 항목 하나만 사용 |
| AWS Account | Team4 공용 계정 | 팀원 IAM 사용자는 달라도 Account ID는 동일 |
| Access domain | `team4.<DOMAIN>` | 실제 도메인 확정 후 치환 |

## 4. 공통 네이밍 규칙

AWS 및 Docker 리소스의 기본 형식은 다음과 같다.

```text
team4-<resource-or-role>-<purpose-or-scope>
```

다음 규칙을 적용한다.

1. Team4가 직접 관리하는 리소스 이름은 `team4`로 시작한다.
2. 영문 소문자, 숫자, 하이픈(`-`)을 기본 문자로 사용한다.
3. 공백, 한글, 불필요한 특수문자는 사용하지 않는다.
4. 이름만으로 리소스 종류와 용도를 식별할 수 있어야 한다.
5. Public·Private 구분이 필요한 경우 `public`, `private`를 포함한다.
6. 데이터베이스 전용 네트워크 리소스에는 `db`를 포함한다.
7. Availability Zone 구분은 이름 끝의 `a`, `b`로 표시한다.
8. 비밀번호, Access Key, Secret Key, 개인 이름과 같은 민감정보를 포함하지 않는다.
9. AWS Account ID와 Region은 전역 고유성이 필요한 S3 Bucket 이름에만 포함한다.
10. 이름 변경으로 재생성이 필요한 리소스는 생성 전에 본 명세와 대조한다.

일반 예시는 다음과 같다.

```text
team4-vpc
team4-public-a
team4-private-db-a
team4-ec2-sg
team4-ec2-runtime-role
```

## 5. 자리표시자 규칙

문서와 저장소에서는 실제 계정 정보 대신 다음 자리표시자를 사용한다.

| 자리표시자 | 의미 | 실제 적용 예시 |
| --- | --- | --- |
| `<ACCOUNT_ID>` | 리소스를 생성하는 공용 AWS 계정의 12자리 ID | `<ACCOUNT_ID>` |
| `<DOMAIN>` | 팀이 선택한 기본 도메인 | `app.mang.pe.kr (임시)` |

실제 AWS Account ID는 비밀정보는 아니지만, 저장소와 공개 제출물에는 부트캠프 정책에 따라 `<ACCOUNT_ID>` 또는 마스킹된 값으로 표기한다. 비밀번호와 Access Key는 어떤 이름이나 문서에도 기록하지 않는다.

## 6. AWS 리소스 네이밍 명세

### 6.1 네트워크

| 리소스 | 표준 이름 | 상세 기준 |
| --- | --- | --- |
| VPC | `team4-vpc` | CIDR `10.21.0.0/16` |
| Public Subnet A | `team4-public-a` | `ap-northeast-2a`, `10.21.0.0/24` |
| Private DB Subnet A | `team4-private-db-a` | `ap-northeast-2a`, `10.21.20.0/24` |
| Private DB Subnet B | `team4-private-db-b` | `ap-northeast-2b`, `10.21.21.0/24` |
| Internet Gateway | `team4-igw` | `team4-vpc`에 연결 |
| Public Route Table | `team4-public-rt` | `team4-public-a`에 연결 |
| Private DB Route Table | `team4-private-db-rt` | Private DB Subnet A/B에 연결 |
| EC2 Security Group | `team4-ec2-sg` | EC2 외부 진입점 |
| RDS Security Group | `team4-rds-sg` | 3306 Source를 `team4-ec2-sg`로 제한 |

### 6.2 컴퓨팅 및 스토리지

| 리소스 | 표준 이름 | 상세 기준 |
| --- | --- | --- |
| EC2 Instance | `team4-ec2` | Amazon Linux 2023 x86_64 |
| Key Pair | `team4-key` | SSH를 사용할 때 생성 |
| Root EBS | `team4-root-ebs` | gp3 30 GiB, 암호화, 종료 시 삭제 |
| Data EBS | `team4-data-ebs` | gp3 10 GiB, 암호화, `/data` 마운트 |
| Elastic IP | `team4-eip` | `team4-ec2`에 연결 |
| EC2 IAM Role | `team4-ec2-runtime-role` | EC2 Trust와 필요한 최소 권한만 부여 |

IAM Role을 AWS 콘솔에서 EC2용으로 생성하면 Instance Profile은 콘솔에서 함께 관리될 수 있다. CLI 또는 IaC에서 별도 Instance Profile 이름이 필요할 경우 `team4-ec2-instance-profile`을 사용한다.

### 6.3 RDS

| 리소스 또는 설정 | 표준 이름 또는 값 | 상세 기준 |
| --- | --- | --- |
| DB Subnet Group | `team4-db-subnet-group` | `team4-private-db-a`, `team4-private-db-b` 포함 |
| RDS DB Identifier | `team4-mysql` | RDS for MySQL |
| Initial database | `showmethemoney` | 애플리케이션 DB 이름 |
| Master username | `team4_admin` | DB 사용자 규칙에 따른 예외 표기 |
| RDS-managed Secret | AWS 자동 생성 이름 사용 | Secret 값과 전체 ARN을 문서에 기록하지 않음 |

`showmethemoney`와 `team4_admin`은 AWS 리소스 이름이 아니라 데이터베이스 엔진 내부 값이므로 하이픈 기반 공통 규칙의 예외다.

### 6.4 S3, Versioning 및 Lifecycle

S3 Bucket은 AWS 전체에서 고유해야 하므로 다음 형식을 사용한다.

```text
team4-storage-<ACCOUNT_ID>-ap-northeast-2
```

실제 생성 예시는 다음과 같지만 실제 Account ID는 이 문서에 저장하지 않는다.

```text
team4-storage-<ACCOUNT_ID>-ap-northeast-2
```

S3 Versioning은 별도 리소스가 아니라 Bucket 속성이므로 이름을 부여하지 않는다.

```text
Bucket Versioning: Enabled
```

Lifecycle Rule ID는 다음 형식을 사용한다.

```text
team4-<action>-<scope>-<retention>
```

| 구분 | 표준 Rule ID | 범위 | 동작 |
| --- | --- | --- | --- |
| Incomplete multipart 정리 | `team4-abort-incomplete-multipart-7d` | 모든 객체 | 시작 후 7일이 지난 미완료 업로드 삭제 |
| Current export 만료 | `team4-expire-current-exports-30d` | Prefix `exports/` | 생성 후 30일이 지난 현재 버전 만료 |
| Noncurrent export 삭제 | `team4-expire-noncurrent-exports-30d` | Prefix `exports/` | Noncurrent 전환 후 30일이 지난 이전 버전 영구 삭제 |

Versioning 검증 객체는 다음 Key를 사용한다.

```text
exports/version-test.txt
```

이 S3 Bucket은 애플리케이션 런타임 저장소가 아니라 필수과업의 Versioning과 Lifecycle 구성·검증용 리소스다. 애플리케이션의 S3 SDK, 업로드 API 또는 S3 환경변수 이름은 본 명세에 포함하지 않는다.

### 6.5 Docker

| 리소스 | 표준 이름 | 상세 기준 |
| --- | --- | --- |
| Docker Network | `team4-network` | Frontend와 Backend가 공유하는 Bridge Network |
| Backend Container | `team4-backend` | 내부 Port 8080 |
| Frontend Container | `team4-frontend` | 내부 Port 3000 |

Docker Compose를 사용할 경우 실제 Network 이름이 자동 생성 이름으로 바뀌지 않도록 명시적으로 지정한다.

```yaml
networks:
  default:
    name: team4-network
```

Container 이름을 명세와 정확히 일치시켜야 한다면 각 Service에 다음 값을 지정한다.

```yaml
services:
  backend:
    container_name: team4-backend
  frontend:
    container_name: team4-frontend
```

### 6.6 DNS

도메인 등록기관 또는 DNS Hosting 서비스와 관계없이 Team4 애플리케이션 접속 이름은 다음 형식을 기본값으로 사용한다.

```text
team4.<DOMAIN>
```

예시는 다음과 같다.

```text
team4.example.com
```

가비아 DNS를 사용하면 가비아 DNS 관리툴의 A Record Host를 `team4`로 설정한다. Route 53을 사용하면 Public Hosted Zone의 A Record name을 `team4`로 설정한다. 두 방식 모두 Record 값은 `team4-eip`에 할당된 실제 Elastic IP다.

## 7. 태그 명세

공통 태그는 다음 한 개만 사용한다.

| Key | Value | 용도 |
| --- | --- | --- |
| `Team` | `team4` | Team4 소유 리소스 식별 |

VPC, Subnet, Route Table, EC2, EBS, Elastic IP처럼 AWS 콘솔에서 이름을 `Name` 태그로 표시하는 리소스에는 리소스별 `Name` 태그도 입력한다.

```text
Name=team4-vpc
Team=team4
```

`Name`은 각 리소스의 표시 이름이며 공통 분류 태그로 계산하지 않는다. RDS Identifier, S3 Bucket, IAM Role, Security Group처럼 서비스 자체 이름 필드가 있는 경우에도 서비스가 지원하면 동일한 값의 `Name` 태그를 추가할 수 있지만, 공통 필수 태그는 `Team=team4` 하나다.

## 8. 이름 사용 시 금지사항

다음 값은 리소스 이름, Tag, 문서 및 제출 증적에 사용하지 않는다.

- AWS Secret Access Key
- IAM 또는 RDS 비밀번호
- JWT Secret
- 실제 개인 이름, 이메일, 전화번호
- 공개가 승인되지 않은 교육기관 내부 식별자
- 의미를 알 수 없는 임의 숫자 또는 임시 이름
- `test`, `temp`, `new`, `final2`와 같이 용도가 불명확한 접미사

리소스 이름 충돌이 발생하면 임의 문자열을 추가하지 않고 기존 리소스의 소유자와 용도를 먼저 확인한다. 팀 공용 계정에 같은 이름이 이미 존재하면 Team4 리소스인지 검증한 후 재사용 또는 팀 합의에 따른 이름 변경을 수행한다.

## 9. 생성 전 검증 체크리스트

- [ ] 모든 팀 관리 리소스 이름이 `team4`로 시작한다.
- [ ] Region이 `ap-northeast-2`로 선택되어 있다.
- [ ] VPC와 Subnet CIDR이 본 명세와 일치한다.
- [ ] AZ 구분 `a`, `b`가 실제 선택한 Availability Zone과 일치한다.
- [ ] Public·Private·DB 용도가 이름으로 구분된다.
- [ ] S3 Bucket 이름에 공용 AWS Account ID와 Region이 포함되어 있다.
- [ ] 실제 Account ID, 비밀번호, Access Key가 저장소 문서에 기록되지 않았다.
- [ ] S3 Versioning은 이름이 아닌 `Enabled` 속성으로 관리한다.
- [ ] Lifecycle Rule ID와 Prefix, 보관 기간이 본 명세와 일치한다.
- [ ] 공통 태그 `Team=team4`가 적용되어 있다.
- [ ] 표시 이름이 필요한 리소스에 올바른 `Name` 태그가 적용되어 있다.
- [ ] Docker Network와 Container 이름이 실행 설정과 일치한다.
- [ ] RDS Endpoint, IAM ARN, S3 Bucket 이름을 참조하는 설정이 변경된 이름과 일치한다.

## 10. 변경 관리

리소스 이름을 변경할 때는 다음 항목을 함께 검토한다.

- AWS 콘솔의 실제 리소스와 연결 관계
- Docker Compose 또는 Docker 실행 명령
- 애플리케이션 환경변수
- RDS Endpoint와 DB 연결 설정
- IAM Policy의 Resource ARN
- S3 Bucket, Object Prefix 및 Lifecycle Rule
- 가비아 또는 Route 53 DNS Record
- CloudWatch 조회 대상
- 구축 문서와 제출 증적

S3 Bucket 이름과 RDS Identifier처럼 생성 후 직접 변경할 수 없거나 재생성이 필요한 항목은 생성 전에 반드시 팀 검토를 완료한다. 변경이 승인되면 이 문서를 먼저 갱신하고 관련 설정과 AWS 리소스에 동일한 변경을 반영한다.
