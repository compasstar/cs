# AWS

#### 전체 구조
1. Client
2. CDN (CNAME Web 주소로 설정)  
3. Web(S3) (`https://domain.com`)  <-- 도메인 Route53 / HTTPS (AWS Certificate Manager) / 파일 및 이미지(S3)
4. 로드밸런서(ELB)(`https://api.domain.com`) <-- 도메인 Route53  / HTTPS (ACM) 적용 
5. Server(EC2) <-- IP 주소로 연결
6. DB(RDS) <-- IP 주소로 연결


## ✅ EC2 (서버 배포)
- EC2 = 컴퓨터 한 대
- Region(데이터센터 위치) 선택하기: 서비스 사용자들의 위치로 선택하면 된다. (내 위치 아님!)
- OS: Ubuntu
- 보안그룹 인바운드: ssh / TCP / PORT 22, HTTP / TCP / PORT 80
- 스토리지: 30GiB gp3 (프리티어 기준)
- EC2 접속: 인스턴스에 연결 > EC2 Instance Conneect을 사용하여 연걸

### 탄력적 IP 설정
- EC2 인스턴스에 할당받은 IP 는 임시 IP 이다
- 탄력적 IP 로 고정 IP 를 설정해주어야 한다.

### Spring Boot 서버 EC2 에 배포
```
$ sudo apt update
$ sudo apt install openjdk-21-jdk
$ java -version
$ git clone https://github.com/compasstar/project-sample.git
$ cd project-sample
```

> `application.yml` 와 같은 민감한 정보 파일은 `.gitignore` 로 포함되어 버전관리를 하지 않는다.
> 직접 vi 를 통해 작성해주자 (복붙)
```
server:
  port: 80
```

> 서버 실행시키기
```
$ ./gradlew clean build # 기존 빌드 파일 삭제 후 새롭게 JAR 빌드
$ cd ~/project-sample/build/libs
$ sudo java -jar project-sample-0.0.1-SNAPSHOT.jar
```

## ✅ Route53 (도메인 연결)
1. 도메인 구매 (1년 5$ 정도)
2. Route53 > 호스팅 영역 > 레코드 생성
3. 레코드 유형: A-IPv4 / 값: EC2 IP
> 무료 도메인 팁: 내도메인 사이트 


## ✅ ELB (로드밸런싱)
- ELB 서버에서 여러 EC2 로 IP 를 분배하여 연결한다 (User -> ELB -> EC2 Instances)
- EC2 인스턴스를 띄우듯이 ELB 서버 또한 띄워야하며 고유 IP 또한 지니고 있다 (다만 탄력적 IP 는 설정할 필요 없다)
- User 는 ELB 로 접속함으로, 도메인도 ELB 에 적용하고 HTTPS 도 ELB 에 적용시켜야 한다

### ELB 세팅
1. Region 선택
2. 로드 밸런서 유형: Application Load Balancer (HTTPS 활용 가능)
3. 체계: 인터넷 경계, IP: IPv4
4. 네트워크 매핑: 모든 영역 트래픽 가능 (모든 가용 영역 체크하기)
5. 보안그룹 인바운드: HTTP / TCP / 80, HTTPS / TCP / 443 (ssh 는 원격접속 용도라 필요 없음)
6. 대상그룹
   - 대상유형: 인스턴스
   - 프로토콜: HTTP / PORT: 80 / IP유형: IPv4 / 프로토콜 버전: HTTP1
   - 상태검사: HTTP / 주소: `/health` (30초 마다 해당 EC2 가 통신가능한지 체크하고, 에러라면 트래픽 분배 하지 않는다) -> 백엔드에 Health Check 용 API 만들어야 함
   - 대상등록하기: 원하는 EC2 인스턴스들 선택

### ELB 도메인 연결
- Route53 에서 ELB에 도메인 연결 (탄력적 IP 필요 없음)


## ✅ HTTPS 인증서

### 인증서 요청
1. AWS Certificate Manager 서비스에서 인증서 요청
2. DNS 검증 / RSA 2048 키 인증
3. 인증서 > Route S3 에서 DNS 레코드 생성을 통해 인증서를 생성할 수 있다

### ELB 에 리스너 추가
1. 프로토콜 HTTPS / PORT 443
2. 대상그룹 설정
3. 보안정책 선택
4. Certificate 항목에서 인증서를 선택해주면 완료

### HTTP 로 접속해도 HTTPS 로 전환하기
1. 리스너 및 규칙에서 기존 HTTP:80 은 삭제해주자
2. 다시 HTTP:80 으로 리스너를 만든다
3. 다만 라우팅 액션에서 URL 로 리디렉션 선택
4. URL 리디렉션: HTTPS 443 설정

> EC2 + Route53 + ELB 까지 요금이 나가기에, ELB 없이 Nginx 로 무료로 HTTPS 를 적용시키기도 한다. (번잡하다는 단점이 있다)


## ✅ RDS (데이터베이스 연결)
User -> ELB -> EC2 -> RDS

### 왜 EC2 에 DB(MySQL)를 직접 설치하지 않나요?
- 백엔드 서버 터져서 EC2 죽으면 DB 도 같이 죽는다
- EC2 여러 개 중에 하나에 설치할 바에는 하나 새로 만들어서 따로 관리하는게 편할 수 있다
- 부가적인 여러 기능들 제공(자동백업, 모니터링, 다중 AZ 등)

### RDS 세팅
1. Region 선택
2. 엔진선택: MySQL / 템플릿: 프리티어
3. 마스터 사용자 이름 및 암호 설정
4. 스토리지: 범용 SSD(gp3) / 20GiB
5. 퍼블릭 엑세스: 예

### 보안그룹 설정
- EC2 > 보안그룹 > 보안그룹 생성
- 인바운드 규칙: 유형: MYSQL/Aurora / TCP / 3306 / 소스: Any IP 0.0.0.0/0
- RDS > DB 인스턴스 수정 > 연결 > 보안그룹 선택

### 파라미터 그룹 추가 
- DB 의 여러 속성들을 변경해줄 수 있다 
- RDS > 파라미터 그룹 > 파라미터 그룹 생성 (패밀리 mysql8.0)
1. utf8mb4 (인코딩): character_set_client / character_set_connection / character_set_database / character_set_filesystem / character_set_results / character_set_server
2. utf8mb4_unicode_ci (정렬방식): collation_connection / collation_server
3. Asia/Seoul (데이터베이스 시간) : time_zone

- RDS 들어가서 수정
- 추가구성 > DB 파라미터 그룹 > 직접 설정한 그룹으로 바꿔주기
- 재부팅 해야 적용된다

### RDS 접속하기 
- DB 관리툴 종류: Workbench / Datagrip / DBeaver 
- 엔드포인트 + 포트 정보를 이용하여 관리툴에서 Connect
- Server Host(엔드포인트) + Port / Master 아이디 비밀번호로 연결 가능

### Spring Boot 서버에서 RDS 연결하기 
#### application.yml
```
server:
	port: 80
spring:
	datasource:
		url: jdbc:mysql://___________:3306/instagram # RDS 인스턴스 엔드포인트
		username: ______ # RDS 마스터 사용자 이름
		password: ______ # RDS 마스터 암호
		driver-class-name: com.mysql.cj.jdbc.Driver
	jpa:
		hibernate:
			ddl-auto: update
		show-sql: true
```
실제로는 `.gitignore` 로 버전관리에 제외해야 한다 

```
$ sudo lsof -i:80 # 80번 포트에서 실행되는 프로세스 확인
$ sudo kill {PID 값} # 80번 포트에서 실행되는 프로세스가 있다면 종료
$ cd ~/aws-rds-springboot
$ ./gradlew clean build -x test # 스프링 부트 프로젝트 빌드
$ cd build/libs
$ sudo nohup java -jar aws-rds-springboot-0.0.1-SNAPSHOT.jar & # JAR 파일 실행
$ sudo lsof -i:80 # 80번 포트에서 실행되는 프로세스 조회
```
EC2 에서 RDS 와 잘 연결되는지 확인 가능 


## ✅ S3 (파일 및 이미지 업로드)
- 버킷(Bucket): 저장소 / 객체(Object): 업로드 된 파일
- ACL 비활성화됨, 모든 퍼블릭 엑세스 차단 해제 (모든 사용자가 다운로드 할 수 있어야 한다)

### 버킷에 GetObject 정책 추가하기
- 모든 서비스 > S3 > GetObject > 리소스 추가
- `arn:aws:s3:::{버킷이름입력}/{오브젝트이름입력}`
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid":"Statement1",
      "Effect":"Allow",
      "Principal":"*",
      "Action":"s3:GetObject",
      "Resource":"arn:aws:s3:::instagram-static-files/*",
    }
  ]
}
```

### 버킷에 EC2 에서 이미지 업로드 허용하기 
- AWS 메뉴 IAM (Identity and Access Management) 접속
- 사용자 생성 > 직접 정책 연결 > 권한 정책 > AmazonS3FullAccess (모든 권한 획득하기)
- 보안 자격 증명 > 액세스 키 만들기 > AWS 외부에서 실행되는 애플리케이션 (Spring Boot 같은게 모두 AWS 외부에서 실행되는 애플리케이션이다) 
- 액세스 키 저장해두기 

#### application.yml
```
server:
	port: 80
spring:
	datasource:
		url: jdbc:mysql://_________:3306/instagram # RDS 인스턴스 엔드포인트
		username: ______ # RDS 마스터 사용자 이름
		password: ______ # RDS 마스터 암호
		driver-class-name: com.mysql.cj.jdbc.Driver
	jpa:
		hibernate:
			ddl-auto: create
		show-sql: true
	cloud:
		aws:
			credentials:
				access-key: _________ # IAM 통해서 발급받은 액세스 키
				secret-key: _________ # IAM 통해서 발급받은 비밀 액세스 키
			s3:
				bucket: _______ # 생성한 S3 버킷명
			region:
				static: ap-northeast-2
```


## ✅ S3, CloudFront (웹 페이지 배포)

### CloudFront 란? 
- CDN(Content Delivery Network) 서비스이다. 원 저장소는 S3 이지만, 전세계 곳곳에 컨텐츠의 복사본을 복사하여 임시 저장소에 구축한다. 덕분에 빠르게 컨텐츠를 전송할 수 있다.
- HTTPS 를 적용시키는 기능 또한 제공한다
- Users -> CloudFront -> S3

### S3 버킷 생성하기
- ACL 비활성화됨, 모든 퍼블릭 엑세스 차단 해제 (모든 사용자가 다운로드 할 수 있어야 한다)

### 버킷에 GetObject 정책 추가하기
- 모든 서비스 > S3 > GetObject > 리소스 추가
- - `arn:aws:s3:::instagram-web-page/*`
- `arn:aws:s3:::{버킷이름입력}/{오브젝트이름입력}`
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid":"Statement1",
	  "Principal": "*",
	  "Effect":"Allow",
      "Action": ["s3:GetObject"],
      "Resource":["arn:aws:s3:::instragram-web-page/*"]
    }
  ]
}
```

### S3 에 웹서비스 업로드하기
- 객체 메뉴에서 파일 드래그해서 업로드하면 됨 index.html 을 포함한 build 폴더 안의 모든 내용물을 업로드하자 
- 속성 메뉴에서 '정적 웹 사이트 호스팅' 활성화 -> 인덱스 문서에 index.html 넣어주자
- 주소가 뜬다. 해당 주소로 이동하면 웹 페이지 접속 가능

### CloudFront 생성하기
- 배포 > 생성
- 원본 도메인: S3 호스팅한 웹 서비스 주소 입력, 웹 사이트 엔드포인트 사용 버튼 클릭
- Redirect HTTP to HTTPS 선택
- 보안 보호 비활성화 선택 (비활성화 되어도 보안 충분)
- 가격 분류: 북미, 유럽, 아시아, 중동 및 아프리카에서 사용
- 기본값 루트 객체: index.html

### 도메인 연결하기, HTTPS 적용하기
#### AWS Certificate Manager(ACM) 메뉴
- 지역을 미국 동부(버지니아 북부)로 변경해서 발급받아야 한다 (AWS 지침)
- 인증서 요청, 도메인 이름 (jscode-edu.link) -> 인증서가 생성되었다
- Route 53에서 DNS 레코드 생성
#### CloudFront 메뉴
- 설정 편집에서 대체 도메인 이름(CNAME) jscode-edu.link 입력
- 사용자 정의 SSL 인증서에서 생성한 인증서 입력 
#### Route 53 메뉴
- 레코드 생성: 레코드 이름: jscode-edu.link / 트래픽 라우팅 대상: CloudFront 배포에 대한 별칭 / 배포 선택하기 
