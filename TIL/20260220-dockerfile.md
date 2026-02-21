## dockerfile이란?
Dockerfile (설계도): 애플리케이션을 빌드하기 위한 단계별 명령어가 담긴 텍스트 파일.

## image란?
Image (박제된 상태): Dockerfile을 빌드하여 생성된 실행 가능한 패키지. 변경 불가능(Immutable)하며, Git의 Commit과 유사함.

## container란?
Container (실행된 인스턴스): 이미지를 실행한 상태. 독립적인 환경에서 프로세스가 작동하며, 객체지향의 Instance와 유사함.

# docker로 배포하는 과정
배포는 크게 Build → Ship → Run의 과정을 거칩니다.
1. Build: 이미지 만들기
내 컴퓨터(로컬환경)에서 Dockerfile을 작성하고 이미지를 생성합니다.

# -t는 이미지의 이름(태그)을 지정
```bash
docker build -t my-express-app .
```

2. Ship: 이미지 공유하기 (Docker Hub)
생성한 이미지를 원격 저장소인 Docker Hub에 올립니다. (GitHub에 Push하는 것과 동일)
# 이미지에 계정 정보 포함하여 태그 설정
```bash
docker tag my-express-app 사용자계정/my-express-app
```
# Docker Hub로 업로드
```bash
docker push 사용자계정/my-express-app
```

3. Run: 서버에서 실행하기
실제 서비스할 서버(AWS, Azure 등)에서 이미지를 내려받아 컨테이너를 띄웁니다.
bash
# Docker Hub에서 이미지 다운로드
```bash
docker pull 사용자계정/my-express-app
```

# 컨테이너 실행 (-d: 백그라운드 실행, -p: 포트 연결)
```bash
docker run -d -p 80:3000 사용자계정/my-express-app
```

"Dockerfile로 이미지를 굽고, Docker Hub에 저장했다가, 어디서든 컨테이너로 찍어낸다."
Git : GitHub = Docker : Docker Hub
Class : Instance = Image : Container