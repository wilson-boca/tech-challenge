# Kubernetes Deployment

## Manifests for Kubernetes

To run the applcation on Kubernetes we are gonna need the manitests below.
You can find about scaling in the end of the document.

## Secrets for API and consumer
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: api-secrets
  namespace: challenge-cluster
type: Opaque
data:
  # PostgreSQL
  POSTGRES_USER: base64secret
  POSTGRES_PASSWORD: base64secret
  POSTGRES_DB: base64secret
  POSTGRES_PORT: base64secret
  POSTGRES_HOST: base64secret
  # RabbitMQ
  RABBITMQ_USER: base64secret
  RABBITMQ_PASSWORD: base64secret
  RABBITMQ_PORT: base64secret
  RABBITMQ_URI: base64secret
  # Redis
  REDIS_USER: base64secret
  REDIS_PASSWORD: base64secret
  REDIS_PORT: base64secret
  REDIS_HOST: base64secret
  REDIS_URI: base64secret

  # API Configuration
  API_PORT: base64secret
  ---
apiVersion: v1
kind: Secret
metadata:
  name: consumer-secrets
  namespace: challenge-cluster
type: Opaque
data:
  # PostgreSQL
  POSTGRES_USER: base64secret
  POSTGRES_PASSWORD: base64secret
  POSTGRES_DB: base64secret
  POSTGRES_PORT: base64secret
  POSTGRES_HOST: base64secret
  # RabbitMQ
  RABBITMQ_USER: base64secret
  RABBITMQ_PASSWORD: base64secret
  RABBITMQ_PORT: base64secret
  RABBITMQ_URI: base64secret
  ```

## Service for the API
For the consumer services are not needed.
```yaml
kind: Service
apiVersion: v1
metadata:
  name: api
spec:
  selector:
    service: api
  ports:
    - name: http
      port: 80
      targetPort: 5590
      protocol: TCP
```

# Deployment file for API and consumer
We are using busybox to wait for all databases before start the POD.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: challenge-cluster
spec:
  replicas: 3
  template:
    metadata:
      labels:
        service: api
      name: api
    spec:
      revisionHistoryLimit: 2
      initContainers:
      - name: wait-for-postgres
        image: busybox
        command: ['sh', '-c', 'until nc -vz postgres 5432; do echo waiting for postgres; sleep 2; done;']
      - name: wait-for-rabbitmq
        image: busybox
        command: ['sh', '-c', 'until nc -vz rabbitmq 5672; do echo waiting for rabbitmq; sleep 2; done;']        
      containers:
      - name: api
        image: container-registry-address/api:latest
        imagePullPolicy: Always
        envFrom:
          - secretRef:
              name: api-secrets
        ports:
        - containerPort: 5590
        resources:
          requests:
            cpu: 300m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 256Mi
  selector:
    matchLabels:
      service: api
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: consumer
  namespace: challenge-cluster
spec:
  replicas: 3
  selector:
    matchLabels:
      service: consumer
  template:
    metadata:
      labels:
        service: consumer
      name: consumer
    spec:
      initContainers:
      - name: wait-for-postgres
        image: busybox
        command: ['sh', '-c', 'until nc -vz postgres 5432; do echo waiting for postgres; sleep 2; done;']
      - name: wait-for-rabbitmq
        image: busybox
        command: ['sh', '-c', 'until nc -vz rabbitmq 5672; do echo waiting for rabbitmq; sleep 2; done;']
      containers:
      - name: consumer
        image: contaier-registry-adddress/consumer:latest
        imagePullPolicy: Always
        envFrom:
          - secretRef:
              name: consumer-secrets
        ports:
        - containerPort: 5672
        resources:
          requests:
            cpu: 150m
            memory: 128Mi
          limits:
            cpu: 300m
            memory: 256Mi      
```
# Infrastructure Components

The application requires PostgreSQL, Redis, and RabbitMQ as infrastructure components. Instead of providing Kubernetes manifests we can use the Helm charts to install them.

- PostgreSQL: `bitnami/postgresql`
- Redis: `bitnami/redis`
- RabbitMQ: `bitnami/rabbitmq`

## Scalling
The application is configured to scale horizontally based on CPU and MEMORY utilization.
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api
  namespace: challenge-cluster
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

We previsously set the resources to be used in all PODs as you can see below:
```yaml
    resources:
        requests:
        cpu: 150m
        memory: 128Mi
        limits:
        cpu: 300m
        memory: 256Mi   
```

Details about how it will be scalled:

- Maintains the minimum of 3 replicas
- Can scale up to a maximum of 10 replicas
- Scales up when CPU utilization reaches 75%
- Scales down when CPU utilization drops below 75%
- Scales up when MEMORY utilization reaches 80%
- Scales down when MEMORY utilization drops below 80%
