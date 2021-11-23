# Argo Workflow on Kubernetes - Quick Start

## Installing Argos Workflow

[Installing Argo](https://argoproj.github.io/argo-workflows/quick-start/)

``` cli
kubectl create ns argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/master/manifests/quick-start-postgres.yaml
```

Start argo server https://localhost:2746
``` cli
kubectl -n argo port-forward deployment/argo-server 2746:2746
```

## Introduction

Argo workflow is a container native workflow engine for orchestrating jobs in kubernetes

### Create workflow
```
kubectl -n argo create -f [YAMLFILE] 
```

### Template Definition & Invocators

1. Containers
2. Script
3. Resource
4. Step/Steps
5. DAG
6. Suspend




### Container
``` yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: wf-container-template
spec:
  entrypoint: container-template      #First step to execute when running a workflow
  templates:
  - name: container-template
    container:
      image: python:3.8-slim
      command: [echo, "The container is executed successfully"]

```


### Script
``` yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: wf-script-templates
spec:
  entrypoint: script-templates
  templates:
  - name: script-templates
    script:
      image: python:3.8-slim
      command: [python]
      source: |
        print("this is executed from the python script")

```


### Resource
``` yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: wf-resource-template
spec:
  entrypoint: resource-template
  templates:
  - name: resource-template
    resource:
      action: create
      manifest: |
        apiVersion: argoproj.io/v1alpha1
        kind: Workflow
        metadata:
          generateName: test-template
        spec:
          entrypoint: test-template
          templates:
          - name: test-template
            script:
              image: python:3.8-slim
              command: [python]
              source: |
                print("Workflow created from resource template")

```


### Step/Steps with Suspend
``` yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata: 
  generateName: wf-step-parallel
spec:
  entrypoint: step-parallel
  templates:
  - name: step-parallel
    steps:                              #Runs step or list of steps sequential and parallel
    - - name: step1
        template: task-template
    - - name: step2
        template: task-template
      - name: step3
        template: task-template
    - - name: delay
        template: delay-template
    - - name: step4
        template: task-template
  - name: task-template                 #Workflow template - Reusable workflow for namespace
    script:
      image: python:3.8-slim
      command: [python]
      source: |
        print("Task completed")
  - name: delay-template
    suspend:
      duration: "10s"

```


### DAG
``` yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata: 
  generateName: wf-dag-template
spec:
  entrypoint: dag-template
  templates:
  - name: dag-template
    dag:
      tasks:
      - name: Task1
        template: task-template
      - name: Task2
        template: task-template
        dependencies: [Task1]
      - name: Task3
        template: task-template
        dependencies: [Task1]
      - name: Task4
        template: task-template
        dependencies: [Task2, Task3]
  - name: task-template
    script:
      image: python:3.8-slim
      command: [python]
      source: |
        print("Task completed")

```



