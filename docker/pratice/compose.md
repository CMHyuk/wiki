도커 컴포즈
* 도커 컴포즈는 여러 개의 Docker 컨테이너들을 관리하는 도구 
* 도커 데스크탑 설치 시 기본으로 설치 
* 한 번의 명령어로 여러 개의 컨테이너를 한 번에 실행하거나 종료할 수 있음 
* 로컬 개발 환경에서 활용하기 편리

```
docker compose up -d - YAML 파일에 정의된 서비스 생성 및 시작

docker compose ps - 현재 실행중인 서비스 상태 표시

docker compose build - 현재 실행중인 서비스의 이미지만 빌드

docker compose logs - 실행 중인 서비스의 로그 표시

docker compose down - YAML 파일에 정의된 서비스 종료 및 제거

docker compose up -d --build - 컴포즈 실행 전 빌드와 함께 실행
```

공통 변수 선언
```
x-enviroment: &common_environment
```
적용
```
enviroment:
    <<: *common_environment
```