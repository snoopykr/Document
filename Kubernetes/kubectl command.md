# kubectl 기본 명령

| 값 | 설명 |
|---|:---:|
| `apply`	| 원하는 상태를 적용합니다. 보통 -f 옵션으로 파일과 함께 사용 |
| `get` | 리소스 목록을 출력 |
| `describe` | 리소스의 상태를 자세하게 출력 |
| `delete` | 리소스를 제거 |
| `exec` | 컨테이너에 명령어를 전달. 컨테이너에 접근할 때 주로 사용 |
| `logs` | 컨테이너의 로그 출력 |
| `config` | kubectl 설정을 관리 |
| `run` | kubectl run nginx-pod --image=nginx (apply...???) |
| `create` | deployment를 추가해야 생성가능 apply...??? |

### wordpress-k8s.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
        - image: mysql:5.6
          name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: password
          ports:
            - containerPort: 3306
              name: mysql

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
        - image: wordpress:5.5.3-apache
          name: wordpress
          env:
            - name: WORDPRESS_DB_HOST
              value: wordpress-mysql
            - name: WORDPRESS_DB_PASSWORD
              value: password
          ports:
            - containerPort: 80
              name: wordpress

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  type: NodePort
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
```

### apply
```bash
$ kubectl apply -f wordpress-k8s.yml
```

### get
```bash
$ kubectl get pod
$ kubectl get pods            // 복수형
$ kubectl get po              // 줄임말
$ kubectl get pod,service     // 다수 Type
$ kubectl get po,svc          // 줄임말
$ kubectl get all             // 전체
```

결과 포맷 변경
```bash
$ kubectl get pod -o wide
$ kubectl get pod -o yaml
$ kubectl get pod -o json
```

Label, 상세 정보 조회
```bash
$ kubectl get pod --show-labels                       // Label 조회
$ kubectl describe pod/wordpress-5f59577d4d-8t2dg     // 상세 정보 조회
```

### delete
```bash
$ kubectl delete pod/wordpress-5f59577d4d-8t2dg
```

### logs
```bash
$ kubectl logs wordpress-5f59577d4d-8t2dg
$ kubectl logs -f wordpress-5f59577d4d-8t2dg          // 실시간 로그
```

### exec
```bash
$ kubectl exec -it wordpress-5f59577d4d-8t2dg -- bash     // bash 실행
```

### config
```bash
$ kubectl config current-context          // 현재 컨텍스트 확인
$ kubectl config use-context minikube     // 컨텍스트 설정
```

### 기타
```bash
$ kubectl api-resources       // 전체 오브젝트 확인
$ kubectl explain pod         // 오브젝트 설명 보기
```