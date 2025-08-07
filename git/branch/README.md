# branch

### 브랜치 확인
```
git branch
```

### 브랜치 생성
```
git branch new-branch main
```
`main` 브랜치로부터 분기해와서 `new-branch` 생성

### 브랜치 전환
```
git checkout <브랜치 이름>

//브랜치 생성 후 전환
git checkout -b <브랜치 이름>
```

### 브랜치 업로드
```
git add .
git commit -m "message"
git push origin [브랜치명]
```
원격저장소에 새로운 브랜치가 생성된다

### 브랜치 병합
```
git checkout main
git merge new-branch
git push
```
`main` 브랜치로 우선 이동 후, `new-branch` 를 병합한다

### 브랜치 삭제
```
//로컬 브랜치 삭제
git branch -d local-branch

//원격 브랜치 삭제
git push origin -d remote-branch
```

`d`는 `--delete` 로 대체 가능하다
