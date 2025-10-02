# AWS


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

### ELB 셋팅
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


## ✅ S3 (파일 및 이미지 업로드)

## ✅ Cloudfront
