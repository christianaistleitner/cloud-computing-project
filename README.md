# Cloud Computing Project - Team 4

Last updated: 18.01.2021 17:00

Lecture: _Special Topics: Cloud Computing Architectures, Processes and Operations (510.211)_ <br>
University: _Johannes Kepler University Linz_

see [PROPOSAL.md](PROPOSAL.md)

## Authors
- [Christian Aistleitner](https://github.com/christianaistleitner)
- [Marcel Brunnbauer](https://github.com/Marcel256)
- [Martin Zwifl](https://github.com/martin-zwifl)

## Some threory here

TODO

## Deployment Structure

### Deployments

We have to following deployments...TODO

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: servebeer-staging-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: servebeer-staging
  template:
    metadata:
      labels:
        app: servebeer-staging
    spec:
      containers:
      - name: server
        image: crix128/servebeer:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: servebeer-production-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: servebeer-production
  template:
    metadata:
      labels:
        app: servebeer-production
    spec:
      containers:
      - name: server
        image: crix128/servebeer:stable
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
```

### Services



Note: A Service can map any incoming `port` to a `targetPort`. By default and for convenience, the `targetPort` is set to the same value as the `port` field.

```
apiVersion: v1
kind: Service
metadata:
  name: servebeer-staging-service
spec:
  ports:
  - port: 8080
  selector:
    app: servebeer-staging
  type: ClusterIP
```


```
apiVersion: v1
kind: Service
metadata:
  name: servebeer-production-service
spec:
  ports:
  - port: 8080
  selector:
    app: servebeer-production
  type: ClusterIP
```

### Ingress

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: servebeer-ingress
spec:
  rules:
  - host: staging.servebeer.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: servebeer-staging-service
            port:
              number: 8080
  - host: production.servebeer.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: servebeer-production-service
            port:
              number: 8080
  - http:
      paths:
        - path: /hooks/github
          pathType: Exact
          backend:
            service:
              name: el-github-listener
              port:
                number: 8080
```

## Pipeline Structure

TODO