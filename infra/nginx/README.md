# Nginx

## Nginx 는 왜 필요할까?
1. 정적컨텐츠 제공 (프론트엔드 배포)
2. 멀티도메인 (하나의 EC2 에 일반 사이트 + 어드민 사이트 두 개 배포하고 각각 도메인 연결하기)
3. HTTPS 적용
4. 리버스 프록시(Reverse Proxy)를 이용해 백엔드 서버 배포 + IP 당 요청 수 제한 + CORS 문제 해결
5. 로드밸런서

## ✅ 1. 프론트엔드(React + Vite) 배포


## ✅ 2. 멀티 도메인
- 도메인 구매: jscode.p-e.kr / admin.jscode.p-e.kr --> EC2 IP 주소로 연결
- nginx 에서 EC2 IP 로 온 도메인을 받아 각각 프로젝트의 index.html 로 연결시켜준다

```
/etc/nginx/conf.d$ sudo vi defualt.conf
```

```
server {
  listen 80;
  server_name jscode.p-e.kr;

  location / {
    root /usr/share/nginx/nginx-frontend-project/dist;
    index index.html;
  }
}

server {
  listen 80;
  server_name admin.jscode.p-e.kr;

  location / {
    root /usr/share/nginx/nginx-frontend-admin-project/dist;
    index index.html;
  }
}
```

## ✅ 3. HTTPS 적용

### Certbot 설치
```
sudo snap install --classic certbot
```

### 심볼릭 링크(바로가기) 생성
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### HTTPS 인증서 발급
```
sudo certbot --nginx -d jscode.p-e.kr
```
- 입력 후 이메일 입력하여 인증받기
- 어드민 페이지도 발급해주자
- 인증이 되었는지 사이트 들어가서 확인해보자
- HTTP 로 들어가도 HTTPS 로 리다이렉트하도록 설정된다 
- `/etc/nginx/conf.d/default.conf` 파일을 보면 Certbot 이 설정한 코드를 확인할 수 있다 

## ✅ 4. 리버스 프록시
- 리버스 프록시 설정을 통해 같은 도메인(출처) 취급이 되어 CORS 문제 또한 자동으로 해결된다 (Spring Boot 에서 allowed-origins 설정 필요 X)

### Spring Boot 서버 배포하기

#### JDK 설치
```
$ sudo apt update && / sudo apt install openjdk-17-jdk-y
```

#### Github 로 부터 Clone
`/home/ubuntu` 경로에서 실행 (`cd ~` 명령어로도 해당 경로 이동 가능)
```
$ git clone https://github.com/JSCODE-COURSE/nginx-backend-springboot.git
$ cd nginx-backend-springboot
```

#### Spring Boot 서버 실행
```
$ ./gradlew clean build -x test
$ cd build/libs
$ nohup java -jar nginx-backend-springboot-0.0.1-SNAPSHOT.jar &

# 8080번 포트에서 실행되고 있는 프로세스 조회
$ lsof -i:8080
```

#### Nginx 설정 파일 작성 
`/etc/nginx/conf.d/websites/api.jscode.p-e.kr/conf`
```
server {
  listen 80;
  server_name api.jscode.p-e.kr;

  # / 으로 시작하는 모든 경로를 처리
  location {
    proxy_pass http://localhost:8080;
  }
}
```

#### 반영된 Nginx 설정 내용 적용
```
$ sudo nginx -t
$ sudo nginx -s reload
```

### 도메인 발급
- 도메인 구매: api.jscode.p-e.kr --> EC2 IP 주소로 연결
- 기존 주소에 앞에 부분만 추가해주면 된다 

### 서버 도메인 HTTPS 인증서 발급
```
sudo certbot --nginx -d api.jscode.p-e.kr
```

### IP 요청 수 제한하기
- 설정파일 경로: `/etc/nginx/conf.d/websites$ sudo vi api.jscode.p-e.kr.conf`
- IP 내역을 10m 의 메모리에 저장, 1초에 최대 3번까지 접속시도 허용
- 요청 너무 많이 보낼 시, 429 Too Many Requests 페이지 출력 
 
```
limit_req_zone $binary_remote_addr zone=mylimit:10m late=3r/s;

server {
  limit_req zone=mylimit;
  limit_req_status 429;
}
```


## ✅ 5. 로드밸런서

- 하나의 EC2 에 스프링부트 서버를 두 개 띄우고 8080, 8081 로 실행한다

### 프로젝트 빌드
```
$ ./gradlew clean build -x test
$ cd build/libs
$ nohup java -jar nginx-backend-springboot-0.0.1-SNAPSHOT.jar --ㄴserver.port=8081 &

# 8081번 포트에서 실행되고 있는 프로세스 조회
$ lsof -i:8081
```

### conf 설정파일

```
upstream backend {
  server localhost:8080;
  server localhost:8081;
}

server {
  server_name api.jscode.p-e.kr;

  location / {
    proxy_pass http://backend;
  }
}
```

### 문법체크
```
sudo nginx -t # 문법체크
sudo nginx -s reload # 리로드
```

### health check 활용하여 로드밸런서 잘 되는지 확인해보기 
- 도메인 api.jscode.p-e.kr/health 로 접속하면 서버 고유 id 값이 나온다
- 만약 로드밸런싱이 잘 안되고 있다면 Server ID 가 변함이 없음
- 새로고침을 할 때마다 두 IP 가 번갈아가며 나오면 정상이다 
