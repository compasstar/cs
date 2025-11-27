# Nginx

## Nginx 는 왜 필요할까?
1. 정적 컨텐츠 제공 (프론트엔드 배포)
2. 멀티도메인 (하나의 EC2 에 일반 사이트 + 어드민 사이트 두 개 배포하고 각각 도메인 연결하기)
3. HTTPS 적용
4. 리버스 프록시(Reverse Proxy)를 이용해 백엔드 서버 배포 + IP 당 요청 수 제한
5. 로드 밸런서

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
