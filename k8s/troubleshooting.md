### 문제 상황
현재 리소스 서버에서 테스트를 진행할 때, 인가 서버를 함께 띄워야 하므로, 테스트 컨테이너 환경에서 테스트를 진행했다. 아래 코드와 같이 설정하여 테스트를 실행했다.
``` java
static final DockerImageName AUTHORIZATION_IMAGE = DockerImageName.parse("scr.softcamp.co.kr/secaas/ojt-minhyeok-authorization:latest");
// 추가 로직
```

그런데 Jenkins 파이프 라인에서 모두 통과하던 테스트가 어느 날 갑자기 실패했다. 에러 로그를 확인해 보니, 이미지 pull 과정에서 권한이 없다는 오류가 발생했다. 
비즈니스 로직과 테스트 코드는 건드리지 않았고, 원래 문제없이 잘 동작했기 때문에 이유를 알 수 없어 당황스러웠다.  

```
05:25:06.175 [docker-java-stream-1859973121] ERROR com.github.dockerjava.api.async.ResultCallbackTemplate -- Error during callback
com.github.dockerjava.api.exception.InternalServerErrorException: Status 500: {"message":"unauthorized: unauthorized to access repository: secaas/ojt-minhyeok-authorization, action: pull: unauthorized to access repository: secaas/ojt-minhyeok-authorization, action: pull"}
```

### 원인
| NAME | READY | STATUS | RESTARTS | AGE | IP | NODE |
| --- | --- | --- | -- | --- | --- | --- |
| Authorization | 0/4 | ContainerCreating | 0 | 29s | <none> | dev-kubernetes-worker-3 |
| Resource | 6/6 | Running | 0 | 69s | 10.42.2.155 | dev-kubernetes-master-3 |

기존에는 동일한 노드에서 리소스와 인가 서버가 실행되었기 때문에, 이미지가 이미 캐싱되어 있어 문제가 없었다. 하지만 이번에는 서로 다른 노드에서 실행되면서, 인가 서버 컨테이너가 레지스트리에서 이미지를 pull해야 했고, 이 과정에서 권한 문제가 발생했다.

결과적으로, 다른 노드에 배포되면서 이미지가 캐싱되지 않아 레지스트리에서 pull을 시도했고, 이때 권한 부족으로 인해 에러가 발생한 것이다.

### 해결
`config.json` 파일에는 Docker 레지스트리에 대한 인증 정보(주로 인증 토큰이나 자격 증명), 자격 증명 저장소 설정, 클라이언트 설정이 포함되어 있다. 이 파일을 마운트하여 이미지를 pull할 때 인증 문제를 해결했다.

```
volumes: [
        hostPathVolume(hostPath: "/var/run/docker.sock", mountPath: "/var/run/docker.sock"),
        hostPathVolume(hostPath: "/home/iadmin/.docker/config.json", mountPath: "/root/.docker/config.json"), //추가한 로직
        persistentVolumeClaim(mountPath: '/root/.m2/repository', claimName: 'maven-repo-storage', readOnly: false)
]
```

또한, 노드에 캐싱된 이미지가 구버전일 수 있는 문제를 방지하기 위해, 항상 최신 이미지를 pull하도록 설정했다.
``` java
AUTHORIZATION_CONTAINER = new GenericContainer<>(AUTHORIZATION_IMAGE)
         .withImagePullPolicy(PullPolicy.alwaysPull());
```
