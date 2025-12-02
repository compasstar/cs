# Nginx

## Nginx 는 왜 필요할까?
1. 정적컨텐츠 제공 (프론트엔드 배포)
2. 멀티도메인 (하나의 EC2 에 일반 사이트 + 어드민 사이트 두 개 배포하고 각각 도메인 연결하기)
3. HTTPS 적용
4. 리버스 프록시(Reverse Proxy)를 이용해 백엔드 서버 배포 + IP 당 요청 수 제한
5. 로드밸런서

## 1. 프론트엔드(React + Vite) 배포


## 2. 멀티 도메인
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

## 3. HTTPS 적용

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

## 4. 리버스 프록시

### 도메인 발급
- 도메인 구매: api.jscode.p-e.kr --> EC2 IP 주소로 연결
- 기존 주소에 앞에 부분만 추가해주면 된다 

### 서버 도메인 HTTPS 인증서 발급
```
sudo certbot --nginx -d api.jscode.p-e.kr
```

## 5. 로드밸런서
