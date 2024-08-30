### Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-test
  labels:
    app: jenkins-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins-test
  template:
    metadata:
      labels:
        app: jenkins-test
    spec:
      containers:
      - name: jenkins-test
        image: harbor.softcamp.co.kr/jenkins-test/jenkins-test:BUILD_NUMBER
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
      imagePullSecrets:
      - name: regcred
```
**spec**
- **replicas: 1**: 이 `Deployment`가 관리할 파드 수
- **selector**
    - **matchLabels**: `app: jenkins-test` 라벨이 있는 파드를 선택, `Deployment`에서 관리할 파드를 식별하는 데 사용
- **template**
    - **metadata**
        - **labels**: `app: jenkins-test` 라벨이 새로 생성되는 파드에 적용
    - **spec**:
        - **containers**
            - **name: jenkins-test**: 컨테이너의 이름
            - **image: harbor.softcamp.co.kr/jenkins-test/jenkins-test**: 컨테이너에 사용할 도커 이미지를 지정, `BUILD_NUMBER`는 Jenkins 파이프 라인에서 빌드 번호로 치환
            - **imagePullPolicy: Always**: 항상 최신 이미지를 풀(pull)
            - **ports**:
                - **containerPort: 8080**: 컨테이너가 수신할 포트 번호
        - **imagePullSecrets**:
            - **name: regcred**: 프라이빗 Docker 레지스트리에서 이미지를 가져오기 위해 사용할 시크릿의 이름

#### HPA
```
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: test-hpa
  namespace: KUBE_NAMESPACE
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: test
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        targetAverageUtilization: 70
```
- **minReplicas: 2**: 스케일링할 때 유지할 최소 파드 수를 지정합니다. 여기서는 최소 2개의 파드를 유지
- **maxReplicas: 5**: 스케일링할 때 허용되는 최대 파드 수를 지정합니다. 여기서는 최대 5개의 파드까지 스케일링
- **metrics**: 스케일링 결정을 내리기 위해 사용하는 메트릭을 정의
    - **type: Resource**: 리소스 사용량을 기반으로 한 메트릭을 사용
    - **resource**
        - **name: cpu**: 모니터링할 리소스의 이름, 여기서는 CPU 사용량이 기준
        - **targetAverageUtilization: 70**: 평균 CPU 사용량이 70%에 도달할 때 스케일링 이벤트가 발생
     
### Service
```
apiVersion: v1
kind: Service
metadata:
  name: authorization-minhyeok-service
spec:
  selector:
    app: authorization-minhyeok
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: LoadBalancer
```
**`spec`**
- **`selector`**
    - 이 서비스가 트래픽을 전달할 파드를 선택하기 위한 라벨 셀렉터
    - **`app: authorization-minhyeok`**: `authorization-minhyeok`이라는 라벨을 가진 파드들이 이 서비스에 연결
- **`ports`**
    - 서비스가 사용할 포트와 해당 포트가 어떤 프로토콜을 사용할지를 정의
    - **`protocol: TCP`**: TCP 프로토콜을 사용
    - **`port: 8080`**: 서비스가 클러스터 외부에서 노출할 포트, 클라이언트는 이 포트를 통해 서비스에 접근합니다.
    - **`targetPort: 8080`**: 실제 파드 내부에서 동작하는 애플리케이션이 사용하는 포트, 서비스는 이 포트로 트래픽을 전달
- **`type: LoadBalancer`**
    - `LoadBalancer` 타입은 클러스터 외부에서 접근할 수 있는 IP 주소를 자동으로 생성하여 부여, 클라우드 제공자(예: AWS, GCP, Azure) 환경에서는 이 설정을 통해 외부 IP 주소가 할당되어 트래픽을 받아들이고, 이를 클러스터 내부의 파드로 전달
 
### Ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-nginx-minhyeok
  namespace: dev
spec:
  ingressClassName: ingress-nginx-minhyeok
  rules:
    - host: ojt-minhyeok-authorization.softcamp.co.kr
      http:
        paths:
          - path: /authorization
            pathType: Prefix
            backend:
              service:
                name: ojt-minhyeok-authorization
                port:
                  number: 80
  tls:
    - hosts:
        - minhyeok-authorization.softcamp.co.kr
      secretName: ca-key-pair
```
