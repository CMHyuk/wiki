### 명령어
* Pod 상태 확인
```
kubectl describe pod <pod-name> -n <namespace>
```

* 특정 Pod 상세 정보
```
kubectl describe pod ojt-minhyeok-authorization-84564db5bf-zghpz -n dev
```

* 특정 Pod 환경 변수 조회
```
kubectl exec -it <pod-name> -n dev -- /bin/sh
printenv | grep {조회할 값 변수}
```

* 특정 Deployment 상세 정보
```
kubectl describe deployment <deployment-name> -n <namespace>
```

* Pod 목록 조회
```
kubectl get pods -n <namespace>
```

* 특정 Pod YAML 확인
```
kubectl get pod <pod-name> -n <namespace> -o yaml
```

* Ingress 조회
```
kubectl get ingress -n <namespace>
```

* 포트 포워딩
```
kubectl port-forward deployment/<deployment-name> 8080:9000 -n dev --kubeconfig="C:\Users\Security365\Desktop\softcamp-dev.yaml”
```

* 로그 실시간 스트링
```
kubectl logs -f <pod-name> -n <namespace>
```
