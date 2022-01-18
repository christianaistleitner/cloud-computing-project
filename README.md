# Cloud Computing Project - Team 4

Last updated: 18.01.2021 17:00

Lecture: _Special Topics: Cloud Computing Architectures, Processes and Operations (510.211)_ <br>
University: _Johannes Kepler University Linz_

Project Idea: _Implementation of a Continuous Deployment (CD) Pipeline using Tekton_
(see [PROPOSAL.md](PROPOSAL.md))

## Authors
- [Christian Aistleitner](https://github.com/christianaistleitner)
- [Marcel Brunnbauer](https://github.com/Marcel256)
- [Martin Zwifl](https://github.com/martin-zwifl)

## Research

Tekton is an open-source cloud-native Continuous Integration and Deployment solution running on Kubernetes clusters. It can be separated into the following components:

Building blocks of a workflow:
- [Pipelines](https://tekton.dev/docs/pipelines) execute tasks which themself contain one or multiple steps
- [Triggers](https://tekton.dev/docs/triggers) instantiate pipelines runs and ensure the right timing within workflows

Workflow management:
- [Command-Line-Interface](https://tekton.dev/docs/cli) for workflow management
- [Dashboard](https://tekton.dev/docs/dashboard) in case you prefer a graphical interface

Workflow repository:
- [Catalog](https://tekton.dev/docs/catalog) is a repository containing predefined building blocks
- [Hub](https://hub.tekton.dev/) is a web app used for accessing the Tekton Catalog

Kubernetes integration:
- [Operator](https://tekton.dev/docs/operator) is a Kubernetes extension that allows us to manage Tekton components

## Tutorial

### Step 1: Install

TODO

### Step 2: Deployments

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

**Services**



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

**Ingress**

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

### Step 3: Pipeline

**Pipeline**

```
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: main-pipeline
spec:
  params:
    - name: git-url
      type: string
    - name: git-revision
      type: string
    - name: deployment-name
      type: string
    - name: container-name
      type: string
    - name: image-name
      type: string
  workspaces:
    - name: source-code
  tasks:
    - name: clone-repo
      taskRef:
        name: git-clone
      params:
        - name: url
          value: $(params.git-url)
        - name: revision
          value: $(params.git-revision)
      workspaces:
        - name: output
          workspace: source-code
    - name: build-image
      taskRef:
        name: docker-publish
      params:
        - name: image
          value: $(params.image-name)
      workspaces:
        - name: source
          workspace: source-code
      runAfter:
        - clone-repo
    - name: deploy-app
      taskRef:
        name: kubectl
      params:
        - name: args
          value: set image deployment/$(params.deployment-name) $(params.container-name)=$(params.image-name)
      runAfter:
        - build-image
```

### Step 4: Trigger

**EventListener**

```
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: github-listener
spec:
  triggers:
    - name: git-listener-trigger
      interceptors:
        - ref:
            name: github
          params:
            - name: secretRef
              value:
                secretName: github-webhook-secret
                secretKey: token
            - name: "eventTypes"
              value: ["push"]
      bindings:
        - ref: git-push-binding
        - name: git-url
          value: https://github.com/dstar55/docker-hello-world-spring-boot.git
        - name: deployment-name
          value: servebeer-staging-deployment
        - name: container-name
          value: servebeer-staging
        - name: image-name
          value: crix128/servebeer:latest
      template:
        ref: main-pipeline-trigger-template

```

**TriggerBinding**

```
apiVersion: triggers.tekton.dev/v1alpha1
kind: TriggerBinding
metadata:
  name: git-push-binding
spec:
  params:
  - name: gitrevision
    value: $(body.ref)
```

**TriggerTemplate**

```
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: main-pipeline-trigger-template
spec:
  params:
    - name: git-url
    - name: git-revision
    - name: deployment-name
    - name: container-name
    - name: image-name
  resourcetemplates:
    - apiVersion: tekton.dev/v1beta1
      kind: PipelineRun
      metadata:
        generateName: triggered-pipeline-run-
      spec:
        pipelineRef:
          name: main-pipeline
        params:
          - name: git-url
            value: $(tt.params.git-url)
          - name: git-revision
            value: $(tt.params.git-revision)
          - name: deployment-name
            value: $(tt.params.deployment-name)
          - name: container-name
            value: $(tt.params.container-name)
          - name: image-name
            value: $(tt.params.image-name)
        workspaces:
        - name: source-code
          volumeClaimTemplate:
            spec:
              accessModes:
                - ReadWriteOnce
              resources:
                requests:
                  storage: 1Gi
```

## Lessons-learned

- Item 1
- Item 2
- Item 3
