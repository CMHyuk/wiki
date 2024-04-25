## 레이어 관리
* Dockerfile에 작성된 지시어 하나당 레이어가 한 개 추가 
* 불필요한 레이어가 많아지면 이미지의 크기가 늘어나고 빌드 속도가 느려질 수 있음 
* 애플리케이션의 크기를 가능한 작게 관리 
  * 베이스 이미지는 가능한 작은 이미지를 사용, alpine OS를 사용하는 것이 좋음 
* .dockerignore 파일을 사용해 불필요한 파일을 제거

```
FROM openjdk:17-oracle

WORKDIR /home/spring

COPY build/libs/*.jar /home/spring/app.jar

CMD ["java", "-Dspring.profiles.active=dev", "-jar", "/home/spring/app.jar"]

CMD를 제외한 지시어가 모두 레이어
```

* RUN 지시어는  &&을 활용해 최대한 하나로 처리

#### 2개 레이어

```
RUN apt-get update
RUN apt-get install -u curl
```
#### 1개 레이어
```
apt-get update && \
apt-get install -y curl
```

