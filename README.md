# vote-app

# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: vote
---
# Resource Quota
apiVersion: v1
kind: ResourceQuota
metadata:
  name: vote-quota
  namespace: vote
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 2Gi
    limits.cpu: "4"
    limits.memory: 4Gi
    pods: "15"
---
# Redis Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis
        resources:
          requests:
            cpu: "0.2"
            memory: 128Mi
          limits:
            cpu: "0.5"
            memory: 256Mi
        ports:
        - containerPort: 6379
---
# Redis Service
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  namespace: vote
spec:
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
  type: ClusterIP
---
# PostgreSQL Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-deployment
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - name: postgres
        image: postgres
        resources:
          requests:
            cpu: "0.5"
            memory: 256Mi
          limits:
            cpu: "1"
            memory: 512Mi
        env:
        - name: POSTGRES_PASSWORD
          value: "mypassword"
        ports:
        - containerPort: 5432
---
# PostgreSQL Service
apiVersion: v1
kind: Service
metadata:
  name: db-service
  namespace: vote
spec:
  selector:
    app: db
  ports:
  - port: 5432
    targetPort: 5432
  type: ClusterIP
---
# Voting App Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote-deployment
  namespace: vote
spec:
  replicas: 2
  selector:
    matchLabels:
      app: vote
  template:
    metadata:
      labels:
        app: vote
    spec:
      containers:
      - name: voting-app
        image: dockersamples/examplevotingapp_vote
        resources:
          requests:
            cpu: "0.1"
            memory: 64Mi
          limits:
            cpu: "0.3"
            memory: 128Mi
        ports:
        - containerPort: 80
---
# Voting App Service
apiVersion: v1
kind: Service
metadata:
  name: vote-svc
  namespace: vote
spec:
  selector:
    app: vote
  ports:
  - port: 80
    targetPort: 80
    nodePort: 31000
  type: NodePort
---
# Result App Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result-deployment
  namespace: vote
spec:
  replicas: 2
  selector:
    matchLabels:
      app: result
  template:
    metadata:
      labels:
        app: result
    spec:
      containers:
      - name: result-app
        image: dockersamples/examplevotingapp_result
        resources:
          requests:
            cpu: "0.1"
            memory: 64Mi
          limits:
            cpu: "0.3"
            memory: 128Mi
        ports:
        - containerPort: 80
---
# Result App Service
apiVersion: v1
kind: Service
metadata:
  name: result-svc
  namespace: vote
spec:
  selector:
    app: result
  ports:
  - port: 80
    targetPort: 80
    nodePort: 31001
  type: NodePort
---
# Worker App Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: worker
  template:
    metadata:
      labels:
        app: worker
    spec:
      containers:
      - name: worker
        image: dockersamples/examplevotingapp_worker
        resources:
          requests:
            cpu: "0.2"
            memory: 128Mi
          limits:
            cpu: "0.5"
            memory: 256Mi
        env:
        - name: REDIS
          value: "redis-service"
        - name: DB
          value: "db-service"
