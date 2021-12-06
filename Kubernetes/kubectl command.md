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
| `create` | deployment를 추가해야 생성가능 (apply...???) |

## wordpress-docker.yml
```yaml
version: "3"

services:
  wordpress:
    image: wordpress:5.5.3-apache
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD: password
    ports:
      - "30000:80"

  mysql:
    image: mariadb:10.7
    environment:
      MYSQL_ROOT_PASSWORD: password
```

docker-compose 실행 `(wordpress-docker.yml가 있는 디렉토리에서 실행)`
```bash
$ docker-compose up
```

Web browser
```url
http://localhost:30000/
```

## wordpress-k8s.yml

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
        - image: mariadb:10.7
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

## apply
```bash
$ kubectl apply -f wordpress-k8s.yml
```

minikube에서 실행 (ip, port 확인)
```bash
$ kubectl apply -f wordpress-k8s.yml
deployment.apps/wordpress-mysql created
service/wordpress-mysql created
deployment.apps/wordpress created
service/wordpress created

$ minikube ip
192.168.49.2

$ kubectl get all
NAME                                  READY   STATUS    RESTARTS   AGE
pod/wordpress-5f59577d4d-vkv7x        1/1     Running   0          107s
pod/wordpress-mysql-545d9c6dc-6vj6b   1/1     Running   0          107s

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP        3d
service/wordpress         NodePort    10.98.201.113   <none>        80:31604/TCP   107s
service/wordpress-mysql   ClusterIP   10.100.162.62   <none>        3306/TCP       107s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wordpress         1/1     1            1           107s
deployment.apps/wordpress-mysql   1/1     1            1           107s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/wordpress-5f59577d4d        1         1         1       107s
replicaset.apps/wordpress-mysql-545d9c6dc   1         1         1       107s

$ kubectl delete -f wordpress-k8s.yml
deployment.apps "wordpress-mysql" deleted
service "wordpress-mysql" deleted
deployment.apps "wordpress" deleted
service "wordpress" deleted
```

Web browser
```url
http://192.168.49.2:31604/
```

## get
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

## delete
```bash
$ kubectl delete pod/wordpress-5f59577d4d-8t2dg
```

## logs
```bash
$ kubectl logs wordpress-5f59577d4d-8t2dg
$ kubectl logs -f wordpress-5f59577d4d-8t2dg          // 실시간 로그
```

## exec
```bash
$ kubectl exec -it wordpress-5f59577d4d-8t2dg -- bash     // bash 실행
```

## config
```bash
$ kubectl config current-context          // 현재 컨텍스트 확인
$ kubectl config use-context minikube     // 컨텍스트 설정
```

기타
```bash
$ kubectl api-resources       // 전체 오브젝트 확인
$ kubectl explain pod         // 오브젝트 설명 보기
```