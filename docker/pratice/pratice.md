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

### 캐싱을 활용한 빌드
* Docker는 각 단계의 결과 레이어를 캐시처리 -> 지시어가 변경되지 않으면 다음 빌드에서 레이어를 재사용 
* COPY, ADD 명령의 경우 빌드 컨텍스트의 파일 내용이 변경되어도 캐시를 사용하지 않음 
* 명령어가 달라지면 그 레이어와 이후의 모든 레이어는 명령어가 동일하더라도, 캐시를 사용하지 않고 새로운 레이어가 만들어짐 
* 잘 변경되지 않는 레이어는 아래로 배치하면 캐시 활용 빈도 높일 수 있음
