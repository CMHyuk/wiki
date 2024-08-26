### ConfigMap
애플리케이션의 설정 값을 컨테이너에 주입하고 싶을 때 사용하는 리소스이다. `ConfigMap`을 사용하면 Pod에서 직접 환경 변수를 관리하지 않고, ConfigMap을 분리할 수 있다. 이를 통해 애플리케이션을 빌드하고 배포할 때 코드와 설정을 분리할 수 있어 더 유연한 구성 관리가 가능하다.

### 주요 개념 및 특징

1. **구성 데이터의 외부화**
    - 애플리케이션의 구성 데이터를 컨테이너 이미지에서 분리하여 관리할 수 있다. 이렇게 하면 동일한 이미지를 사용하면서도 다른 환경에 대해 다른 설정을 적용할 수 있다.
2. **구조**
    - `ConfigMap`은 키-값 쌍의 형태로 데이터를 저장한다. 각 키는 문자열이고, 각 값은 문자열 또는 파일의 내용일 수 있다.
    - 예를 들어, 데이터베이스 연결 문자열, 외부 API 엔드포인트, 또는 애플리케이션 설정 파일 등을 저장할 수 있다.

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: authorization-minhyeok-config
  namespace: KUBE_NAMESPACE
data:
#  SPRING_PROFILES_ACTIVE: "dev"
  ELASTIC_HOST: "10.30.10.19"
  ELASTIC_PORT: "9200"
  KUBERNETES_CLUSTER_NAMESPACE: ""
  JAVA_OPTS: "-Xms2G -Xmx2G"
  ELASTIC_EXPECTED_HTTP_RESPONSE_TIME_MS: "50"
  ELASTIC_EXPECTED_REQUEST_PER_SEC: "500"
  ELASTIC_SOCKET_TIMEOUT_SEC: "60"
  ELASTIC_CONNECTION_TIMEOUT_SEC: "30"
  ELASTIC_CONNECTION_REQUEST_TIMEOUT_SEC: "30"
  ELASTIC_KEEP_ALIVE_STRATEGY_TIME_MIN: "5"
```

- **`metadata`**
    - `name`: `ConfigMap`의 이름으로, 이 `ConfigMap`을 참조할 때 사용된다.
    - `namespace`: `ConfigMap`이 속한 namespace를 지정한다.
- **`data`**
    - 이 섹션에는 애플리케이션이 필요로 하는 설정 데이터가 키-값 쌍의 형태로 저장된다. 각 키는 문자열이고, 각 값은 해당 키에 대응하는 설정 값이다.

### Pod

쿠버네티스에서 가장 기본적인 배포 단위로, 컨테이너를 포함하는 단위이다. 쿠버네티스의 특징 중 하나는 컨테이너를 개별적으로 하나씩 배포하는 것이 아니라 Pod라는 단위로 배포하는데, Pod는 하나 이상의 컨테이너를 포함한다.

**특징**

- Pod내의 컨테이너는 IP와 Port를 공유한다. 두개의 컨테이너가 하나의 Pod를 통해 배포되었을 때, localhost를 통해서 통신이 가능하다. 예를 들어 컨테이너 A가 8080, 컨테이너 B가 7001로 배포가 되었을 때, B에서 A를 호출할 때는 localhost:8080으로 호출하면 되고, 반대로 A에서 B를 호출할 때는 localhost:7001로 호출하면 된다.
- Pod내의 배포된 컨테이너 간 디스크 볼륨을 공유할 수 있다.

### Service

Pod와 볼륨을 이용해 컨테이너들을 정의하고, Pod를 서비스로 제공할 때 일반적인 분산환경에서는 하나의 Pod로 서비스 하는 경우는 드물고, 여러 개의 Pod를 서비스하면서, 이를 로드밸런서를 이용해 하나의 IP와 포트로 묶어서 서비스를 제공한다.

Pod의 경우 동적으로 생성이 되고, 장애가 생기면 자동으로 리스타트 되면서 그 IP가 바뀌기 때문에, 로드 밸런서에는 Pod의 목록을 지정할 때는 IP주소를 이용하는 것은 어렵다. 또한 오토 스케일링으로 인해 Pod가 동적으로 추가, 삭제되기 때문에 로드밸런서가 유연하게 선택해줘야 한다.

그래서 사용하는 것이 라벨과 라벨 셀렉터라는 개념이다. 서비스를 정의할 때 어떤 Pod를 서비스로 묶을 것인지를 정의하는데, 이를 라벨 셀렉터라고 한다. 각 Pod를 생성할 때 메타데이터 정보 부분에 라벨을 정의할 수 있다. 서비스는 라벨 셀렉터에서 특정 라벨을 가지고 있는 Pod만 선택해 서비스에 묶게 된다.
