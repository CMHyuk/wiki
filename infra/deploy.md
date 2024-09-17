### Ubuntu 도커 CI, CD
1. Ec2 Ubuntu 생성
2. Ec2 생성 시 발급한 pem 키 이동`mv ~/Downloads/ .pem ~/.ssh/`
3. `chmod 600 {pem키 이름}.pem`
4. EC2 접속 `ssh -i ~/.ssh/{pem키 이름}.pem ubuntu@{ec2 ip}`
5. docker install
    1. `sudo apt update`
    2. `sudo apt install docker.io`
    3. `docker version`
    4. `sudo usermod -aG docker $USER`
    5. `sudo chmod 666 /var/run/docker.sock`
6. docker compose install
    1. `sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose`
    2. `sudo chmod +x /usr/local/bin/docker-compose`
    3. `docker-compose version`

```yml
name: deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy-dev:
    runs-on: ubuntu-latest
    steps:
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: 17
          distribution: 'adopt'

      - name: 타임존 설정
        uses: szenius/set-timezone@v1.2
        with:
          timezoneLinux: "Asia/Seoul"

      - name: 저장소 Checkout
        uses: actions/checkout@v3
        with:
          submodules: true
          token: ${{ secrets.ACTION_TOKEN }}

      - name: update submodules
        run: git submodule update --remote

      - name: 스프링부트 애플리케이션 빌드
        run: ./gradlew bootJar

      - name: 도커 이미지 빌드
        run: sudo docker build -f ./docker/Dockerfile -t ${{ secrets.DOCKER_IMAGE }} .

      - name: 도커 이미지 push
        run: |
          sudo docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
          sudo docker push ${{ secrets.DOCKER_IMAGE }} 

      - name: scp docker-compose-dev file
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.PRIVATE_KEY }}
          source: "./src/main/resources/yml-config/docker/docker-compose-dev.yml"
          target: "/home/ubuntu"
          strip_components: 6

      - name: 배포
        uses: appleboy/ssh-action@v0.1.6
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.PRIVATE_KEY }}
          script: |
            sudo docker stop $(sudo docker ps -a -q) 
            sudo docker rm -f $(sudo docker ps -qa)
            sudo docker pull ${{ secrets.DOCKER_IMAGE }}
            sudo docker-compose -f docker-compose-dev.yml up -d
            sudo docker image prune -f
            sudo docker volume prune -f
```

### 도메인.co.kr:8080에서 8080 제거 방법
* TCP 80번 포트로 들어오는 모든 트래픽 8080번 포트로 리다이렉트  
  * `sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080`
