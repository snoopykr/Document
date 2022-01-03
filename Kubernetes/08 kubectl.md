# Kubernetes Ingress

## Ingress 구동

[ a-namespace.yml ]
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: tkr-system
```

[ ing-configmap.yml ]
```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: tkr-system
  labels:
    app: ingress-nginx
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: tkr-system
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: udp-services
  namespace: tkr-system
```

[ ing-controller-with-rbac.yml ]
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: tkr-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ingress-nginx
  template:
    metadata:
      labels:
        app: ingress-nginx
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.13.0
          args:
            - /nginx-ingress-controller
            - --default-backend-service=$(POD_NAMESPACE)/default-http-backend
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --annotations-prefix=nginx.ingress.kubernetes.io
            - --publish-service=$(POD_NAMESPACE)/nginx-ingress-svc
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
          - name: http
            containerPort: 80
          - name: https
            containerPort: 443
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress-svc
  namespace: tkr-system
  labels:
     app: nginx-ingress-svc
spec:
  type: LoadBalancer
  ports:
  - name: http
    port: 80
    targetPort: http
  - name: https
    port: 443
    targetPort: https
  selector:
    app: ingress-nginx
  externalIPs:
    - 172.16.20.99
```

[ ing-default-backend.yml ]
```yaml
Version: apps/v1
kind: Deployment
metadata:
  name: default-http-backend
  namespace: tkr-system
  labels:
    app: default-http-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: default-http-backend
  template:
    metadata:
      labels:
        app: default-http-backend
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - name: default-http-backend
        image: gcr.io/google_containers/defaultbackend:1.4
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: default-http-backend
  namespace: tkr-system
  labels:
    app: default-http-backend
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: default-http-backend
```

[ ing-rbac.yml ]
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: tkr-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-nisa-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: tkr-system
```

[ vip-configmap.yml ]
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vip-configmap
  namespace: tkr-system
data:
  172.16.20.99: tkr-system/nginx-ingress-svc
```

[ vip-daemonset.yml ]
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-keepalived-vip
  namespace: tkr-system
spec:
  selector:
    matchLabels:
      name: kube-keepalived-vip
  template:
    metadata:
      labels:
        name: kube-keepalived-vip
    spec:
      hostNetwork: true
      serviceAccount: kube-keepalived-vip
      containers:
        - image: k8s.gcr.io/kube-keepalived-vip:0.11
          name: kube-keepalived-vip
          imagePullPolicy: Always
          securityContext:
            privileged: true
          volumeMounts:
            - mountPath: /lib/modules
              name: modules
              readOnly: true
            - mountPath: /dev
              name: dev
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          # to use unicast
          args:
          - --services-configmap=tkr-system/vip-configmap
          - --use-unicast=true
          #- --vrrp-version=3
      volumes:
        - name: modules
          hostPath:
            path: /lib/modules
        - name: dev
          hostPath:
            path: /dev
```

[ vip-rbac.yml ]
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-keepalived-vip
  namespace: tkr-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: kube-keepalived-vip
rules:
- apiGroups: [""]
  resources:
  - pods
  - nodes
  - endpoints
  - services
  - configmaps
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kube-keepalived-vip
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-keepalived-vip
subjects:
- kind: ServiceAccount
  name: kube-keepalived-vip
  namespace: tkr-system
```

```bash
# kubectl apply -f ingress-keepalived/
namespace/tkr-system created
configmap/nginx-configuration created
configmap/tcp-services created
configmap/udp-services created
deployment.apps/nginx-ingress-controller created
service/nginx-ingress-svc created
deployment.apps/default-http-backend created
service/default-http-backend created
serviceaccount/nginx-ingress-serviceaccount created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-clusterrole-nisa-binding created
configmap/vip-configmap created
daemonset.apps/kube-keepalived-vip created
serviceaccount/kube-keepalived-vip created
clusterrole.rbac.authorization.k8s.io/kube-keepalived-vip created
clusterrolebinding.rbac.authorization.k8s.io/kube-keepalived-vip created

# kubectl get pod -n tkr-system -w
NAME                                        READY   STATUS    RESTARTS   AGE
default-http-backend-95c88f4d7-jv8vt        1/1     Running   0          4m23s
kube-keepalived-vip-5s5cr                   1/1     Running   0          4m21s
kube-keepalived-vip-sg9k5                   1/1     Running   0          4m21s
nginx-ingress-controller-6797fbdf4d-qgd7j   1/1     Running   0          4m22s
^C

# kubectl apply -f test-apl/
deployment.apps/hello-world-deployment created
service/hello-world-svc created
ingress.networking.k8s.io/hello-world-ingress created

# kubectl get ing
NAME                  CLASS    HOSTS            ADDRESS   PORTS   AGE
hello-world-ingress   <none>   abc.sample.com             80      84s

# kubectl get pod
NAME                                      READY   STATUS    RESTARTS   AGE
hello-world-deployment-67b5744646-6tndj   1/1     Running   0          114s
hello-world-deployment-67b5744646-g2zt5   1/1     Running   0          114s
hello-world-deployment-67b5744646-sbx4s   1/1     Running   0          114s
hello-world-deployment-67b5744646-t8xz6   1/1     Running   0          114s
hello-world-deployment-67b5744646-tht29   1/1     Running   0          114s

# curl --header "Host: abc.sample.com" http://172.16.20.99/
<html><head><title>HTTP Hello World</title></head><body><h1>Hello from hello-world-deployment-67b5744646-g2zt5</h1></body></html
```

## SSL/TLS 암호화

[ application1.yml ]
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
        - image: "strm/helloworld-http"
          name: hello-world-container
          ports:
            - containerPort: 80
  selector:
    matchLabels:
      app: hello-world
---
apiVersion: v1
kind: Service
metadata:
  name: helloworld-svc
spec:
  type: NodePort
  ports:
     -  port: 8080
        protocol: TCP
        targetPort: 80
        nodePort: 31445
  selector:
    app: hello-world
```

[ application2.yml ]
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx
          name:  nginx
          ports:
            - containerPort: 80
  selector:
    matchLabels:
      app: nginx
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
     -  port: 9080
        targetPort: 80
```

[ application3.yml ]
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: liberty
    spec:
      containers:
        - image: openliberty/open-liberty:javaee8-ubi-min-amd64
          name:  open-liberty
          ports:
            - containerPort: 9080
              name: httpport
  selector:
    matchLabels:
      app: liberty
---
apiVersion: v1
kind: Service
metadata:
  name: java-svc
spec:
  selector:
    app: liberty
  ports:
     -  port: 9080
        targetPort: 9080
```

[ ingress.yml ]
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    kubernetes.io/ingress.class: 'nginx'
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: abc.sample.com
    http:
      paths:
      - path: /
        backend:
          serviceName: helloworld-svc
          servicePort: 8080
      - path: /apl2
        backend:
          serviceName: nginx-svc
          servicePort: 9080
  - host: xyz.sample.com
    http:
      paths:
      - path: /
        backend:
          serviceName: java-svc
          servicePort: 9080
```

[ ingress-iks.yml ]
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    # kubernetes.io/ingress.class: 'nginx'
    # nginx.ingress.kubernetes.io/rewrite-target: /
    # nginx.ingress.kubernetes.io/force-ssl-redirect: 'true'
    ingress.bluemix.net/rewrite-path: "serviceName=nginx-svc rewrite=/;serviceName=java-svc rewrite=/"
    ingress.bluemix.net/redirect-to-https: "True"
spec:
  tls:
  - hosts:
    - iks2.jp-tok.containers.appdomain.cloud
    secretName: iks2
  rules:
  - host: iks2.jp-tok.containers.appdomain.cloud
    http:
      paths:
      - path: /
        backend:
          serviceName: helloworld-svc
          servicePort: 8080
      - path: /apl2
        backend:
          serviceName: nginx-svc
          servicePort: 9080
  - host: xyz.sample.com
    http:
      paths:
      - path: /
        backend:
          serviceName: java-svc
          servicePort: 9080
```

[ ingress-tls.yml ]
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    kubernetes.io/ingress.class: 'nginx'
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/force-ssl-redirect: 'true'

spec:
  tls:
  - hosts:
    - abc.sample.com
    secretName: tls-certificate
    
  rules:
  - host: abc.sample.com
    http:
      paths:
      - path: /
        backend:
          serviceName: helloworld-svc
          servicePort: 8080
      - path: /apl2
        backend:
          serviceName: nginx-svc
          servicePort: 9080
  - host: xyz.sample.com
    http:
      paths:
      - path: /
        backend:
          serviceName: java-svc
          servicePort: 9080
```