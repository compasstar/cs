# CI/CD


## ✅ Github Actions 개인 프로젝트
Github Action 자체가 Github 에 있는 하나의 가상컴퓨터이다 

### 프로세스
1. IDEA에서 코드 입력 후 Git Push
2. Github Actions 실행
3. AWS EC2 에서 Git Pull
4. Spring Boot 서버 빌드 및 재구동
- EC2 에서 빌드를 하기에 EC2 성능에 영향을 준다

### git 계정정보저장
> 명령어 입력하고 한 번만 인증하면 git pull 할 때 자동계정인증 (미리 EC2 에서 인증해두자) 
```
git config --global credential.helper store
```

### Github MarketPlace
- [Github MarketPlace](https://github.com/marketplace)
- 깃허브 액션 라이브러리를 모아둔 마켓플레이스다 (예: `appleboy/ssh-action@v1.0.3`)

### Secrets Variables
- Github Action yml 파일에 직접 민감한정보(private key 등)를 적으면 Github 사이트에 노출되기에 보안이 필요하다
- Github > Settings > Security > Secrets and variables > Actions > Repository secrets 경로에 Secrets Variables 를 생성할 수 있다
- Key-Value 형태로 저장된다
- 해당 변수의 Value 값은 외부에 노출되지 않으며, `${{ secrets.EC2_PRIVATE_KEY }}` 와 같은 문법으로 yml 파일에서 사용할 수 있다
- Application Properties 와 같은 내용을 넣어주자 (변화할 때마다 직접 수정해야 한다) 


### 파일 생성
```
project 기본 디렉토리 > .github > workflows > deploy.yml  # yml 파일 이름은 자유
```


### deploy.yml
```
name: Deploy To EC2

on:
  push:
    branches:
      - main

jobs:
  Deploy:
    runs-on: ubuntu-latest
    steps:
      - name: SSH(원격 접속)로 EC2에 접속하기
        uses: appleboy/ssh-action@v1.0.3
        env:
          APPLICATION_PROPERTIES: ${{ secrets.APPLICATION_PROPERTIES }}
        with:
          host: ${{ secrets.EC2_HOST }}  # EC2 IP 주소
          username: ${{ secrets.EC2_UESRNAME }}  # Ubuntu 가상서버 이름 (기본값은 ubuntu)
          key: $${{ secrets.EC2_PRIVATE_KEY }}  # .pem 파일
          envs: APPLICATION_PROPERTIES  # 변수 선언
          script_stop: true
          script: |
            cd /home/ubuntu/instagram-server
            rm -rf src/main/resources/application.yml  # yml 파일 삭제
            git pull origin main
            echo "$APPLICATION_PROPERTIES" > src/main/resources/application.yml  # yml 파일 생성
            ./gradlew clean build  # 테스트도 포함하기에, 테스트가 실패한다면 배포를 하지 않는다 
            sudo fuser -k -n tcp 8080 || true  # 8080 포트 종료, 만약 종료할 게 없어도 에러가 아닌 true 반환
            nohup java -jar build/libs/*SNAPSHOT.jar > ./output.log 2>&1 &  # 실행한 후, 로그를 output.log 파일에 저장
```


## ✅ Github Actions 일반 프로젝트

### 프로세스
1. IDEA에서 코드 입력 후 Git Push
2. Github Actions 실행 & 빌드
3. AWS EC2 에 빌드 파일을 전달
- Github 에서 빌드를 하기에 EC2 의 메모리에 영향을 주지 않는다

### deploy.yml
```
name: Deploy To EC2

on:
  push:
    branches:
      - main

jobs:
  Deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Github Repository 에 올린 파일들을 불러오기
        uses: actions/checkout@v4

      - name: JDK 17버전 설치
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17

      - name: application.yml 파일 만들기
        run: echo "${{ secrets.APPLICATION_PROPERTIES }}" > ./src/main/resources/application.yml

      - name: 테스트 및 빌드하기
        run: ./gradlew clean build

      - name: 빌드된 파일 이름 변경하기
        run: mv ./build/libs/*SNAPSHOT.jar ./project.jar

      - name: SCP로 EC2에 빌드된 파일 전송하기
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: $${{ secrets.EC2_USERNAME }}
          key ${{ secrets.EC2_PRIVATE_KEY }}
          source: project.jar
          target: /home/ubuntu/instragram-server/tobe

      - name: SSH(원격 접속)로 EC2에 접속하기
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}  
          username: ${{ secrets.EC2_UESRNAME }}
          key: $${{ secrets.EC2_PRIVATE_KEY }} 
          script_stop: true
          script: |
            rm -rf /home/ubuntu/instagram-server/current
            mkdir /home/ubuntu/instagram-server/current
            mv /home/ubuntu/instagram-server/tobe/project.jar /home/ubuntu/instagram-server/current/project.jar
            cd /home/ubuntu/instagram-server/current
            sudo fuser -k -n tcp 8080 || true
            nohup java -jar project.jar > ./output.log 2>&1 &
            rm -rf /home/ubuntu/instagram-server/tobe
```


