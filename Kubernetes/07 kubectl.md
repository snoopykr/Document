# Kubernetes StatefulSet

mysql-sts.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql        ## 이 이름이 k8s내 DNS에 등록됨. 
  labels:
    app: mysql-sts
spec:
  ports:
  - port: 3306
    name: mysql
  clusterIP: None    ## 헤드리스 서비스 설정
  selector:
    app: mysql-sts   ## 후술하는 스테이트풀셋과 연결시키는 라벨
---
## MySQL 스테이트풀셋
#
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql        ## 연결할 서비스의 이름 설정
  replicas: 1               ## 파드 기동 개수
  selector:
    matchLabels:
      app: mysql-sts
  template:
    metadata:
      labels:
        app: mysql-sts
    spec:
      containers:           
      - name: mysql
        image: mysql:5.7    ## Docker Hub MySQL 리포지터리 지정
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: qwerty
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:       ## 컨테이너상의 마운트 경로 설정 
        - name: pvc
          mountPath: /var/lib/mysql
          subPath: data     ## 초기화 시 빈 디렉터리가 필요
        livenessProbe:      ## MySQL 기동 확인 
          exec:
            command: ["mysqladmin","-p$MYSQL_ROOT_PASSWORD","ping"]
          initialDelaySeconds: 60
          timeoutSeconds: 10
  volumeClaimTemplates:     ## 볼륨 요구 템플릿
  - metadata:
      name: pvc
    spec:
      accessModes: [ "ReadWriteOnce" ]
      ## 환경에 맞게 선택하여, sotrage의 값을 편집
      #storageClassName: ibmc-file-bronze   # 용량 20Gi IKS
      #storageClassName: gluster-heketi     # 용량 12Gi GlusterFS
      storageClassName: standard            # 용량 2Gi  Minikube/GKE
      resources:
        requests:
          storage: 2Gi

```