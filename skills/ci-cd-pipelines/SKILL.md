---
mode: auto
name: ci-cd-pipelines
description: CI/CD pipeline configuration and automation for storage interfaces
tags: [cicd, github-actions, gitlab-ci, jenkins, automation]
version: 1.0.0
---

# CI/CD Pipelines for Storage Interface Drivers

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Expert knowledge for setting up comprehensive CI/CD pipelines for storage interface development, testing, and deployment.

## Technology Stack

- **CI/CD**: GitHub Actions, GitLab CI, Jenkins
- **Testing**: Go test, csi-sanity
- **Security**: Trivy, Snyk, Cosign
- **Deployment**: Helm, Kustomize, ArgoCD

## GitHub Actions

### Complete CI Pipeline
```yaml
# @author <AUTHOR_NAME>
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - uses: actions/setup-go@v5
      with:
        go-version: '1.21'
    
    - name: golangci-lint
      uses: golangci/golangci-lint-action@v3
      with:
        version: latest
  
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - uses: actions/setup-go@v5
      with:
        go-version: '1.21'
    
    - name: Run tests
      run: |
        go test -v -race -coverprofile=coverage.out ./...
        go tool cover -func=coverage.out
    
    - name: Check coverage
      run: |
        COVERAGE=$(go tool cover -func=coverage.out | grep total | awk '{print $3}' | sed 's/%//')
        if (( $(echo "$COVERAGE < 80" | bc -l) )); then
          echo "Coverage $COVERAGE% is below 80%"
          exit 1
        fi
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        files: ./coverage.out
  
  security:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
        severity: 'CRITICAL,HIGH'
        exit-code: '1'
  
  build:
    needs: [lint, test, security]
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - uses: actions/setup-go@v5
      with:
        go-version: '1.21'
    
    - name: Build binary
      run: |
        CGO_ENABLED=0 go build -ldflags="-w -s" \
          -o bin/storage-interface \
          ./cmd/storage-interface-driver
    
    - name: Upload artifact
      uses: actions/upload-artifact@v3
      with:
        name: storage-interface
        path: bin/storage-interface
```

### Docker Build and Push
```yaml
# @author <AUTHOR_NAME>
# .github/workflows/docker.yml
name: Docker

on:
  push:
    tags:
      - 'v*'

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up QEMU
      uses: docker/setup-qemu-action@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Login to Quay.io
      uses: docker/login-action@v3
      with:
        registry: quay.io
        username: ${{ secrets.QUAY_USERNAME }}
        password: ${{ secrets.QUAY_PASSWORD }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: registry.example.com/storage-interface
        tags: |
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
          type=sha
    
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        platforms: linux/amd64,linux/arm64
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
    
    - name: Sign image
      run: |
        cosign sign --key env://COSIGN_KEY \
          registry.example.com/storage-interface:${{ steps.meta.outputs.version }}
      env:
        COSIGN_KEY: ${{ secrets.COSIGN_PRIVATE_KEY }}
```

### Release Automation
```yaml
# @author <AUTHOR_NAME>
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - name: Generate changelog
      id: changelog
      run: |
        PREVIOUS_TAG=$(git describe --tags --abbrev=0 HEAD^)
        git log ${PREVIOUS_TAG}..HEAD --pretty=format:"- %s" > CHANGELOG.md
    
    - name: Create release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: ${{ github.ref }}
        release_name: Release ${{ github.ref }}
        body_path: CHANGELOG.md
    
    - name: Package Helm chart
      run: |
        helm package deploy/helm-chart \
          --version ${GITHUB_REF#refs/tags/v}
    
    - name: Upload Helm chart
      uses: actions/upload-release-asset@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        upload_url: ${{ steps.create_release.outputs.upload_url }}
        asset_path: ./storage-interface-driver-*.tgz
        asset_content_type: application/gzip
```

## GitLab CI/CD

### Complete Pipeline
```yaml
# @author <AUTHOR_NAME>
# .gitlab-ci.yml
stages:
  - lint
  - test
  - build
  - security
  - deploy

variables:
  GO_VERSION: "1.21"
  DOCKER_DRIVER: overlay2

lint:
  stage: lint
  image: golangci/golangci-lint:latest
  script:
    - golangci-lint run

test:
  stage: test
  image: golang:${GO_VERSION}
  script:
    - go test -v -race -coverprofile=coverage.out ./...
    - go tool cover -func=coverage.out | tee coverage.txt
    - |
      COVERAGE=$(awk '/total:/ {print $3}' coverage.txt | sed 's/%//')
      if (( $(echo "$COVERAGE < 80" | bc -l) )); then
        echo "Coverage ${COVERAGE}% below 80%"
        exit 1
      fi
  coverage: '/total:\s+\(statements\)\s+(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.out

build:
  stage: build
  image: golang:${GO_VERSION}
  script:
    - CGO_ENABLED=0 go build -ldflags="-w -s" -o bin/storage-interface ./cmd/storage-interface-driver
  artifacts:
    paths:
      - bin/storage-interface

security-scan:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy fs --severity HIGH,CRITICAL --exit-code 1 .

docker-build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHA} .
    - docker push ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHA}
  only:
    - main
    - tags

deploy-dev:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/storage-system-controller storage-interface=${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHA}
  environment:
    name: development
  only:
    - main
```

## Jenkins Pipeline

### Declarative Pipeline
```groovy
// @author <AUTHOR_NAME>
pipeline {
    agent any
    
    environment {
        GO_VERSION = '1.21'
        REGISTRY = 'quay.io/storage-system'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Lint') {
            steps {
                sh 'golangci-lint run'
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                    go test -v -race -coverprofile=coverage.out ./...
                    COVERAGE=$(go tool cover -func=coverage.out | grep total | awk '{print $3}' | sed 's/%//')
                    if (( $(echo "$COVERAGE < 80" | bc -l) )); then
                        echo "Coverage $COVERAGE% below 80%"
                        exit 1
                    fi
                '''
            }
            post {
                always {
                    publishCoverage adapters: [coberturaAdapter('coverage.out')]
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'CGO_ENABLED=0 go build -o bin/storage-interface ./cmd/storage-interface-driver'
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'trivy fs --severity HIGH,CRITICAL .'
            }
        }
        
        stage('Docker Build') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.build("${REGISTRY}/storage-interface:${env.BUILD_NUMBER}")
                }
            }
        }
        
        stage('Deploy to Dev') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                    kubectl set image deployment/storage-system-controller \
                        storage-interface=${REGISTRY}/storage-interface:${BUILD_NUMBER}
                '''
            }
        }
    }
    
    post {
        failure {
            slackSend color: 'danger', message: "Build failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        success {
            slackSend color: 'good', message: "Build successful: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
    }
}
```

## Best Practices

### 1. Automated Testing
```yaml
# Run multiple test types
- unit tests (go test)
- integration tests (with testcontainers)
- e2e tests (csi-sanity)
- security tests (trivy, snyk)
```

### 2. Quality Gates
```bash
# @author <AUTHOR_NAME>
# Fail pipeline on:
- Coverage < 80%
- Linter errors
- Security vulnerabilities (HIGH/CRITICAL)
- Failed tests
```

### 3. Artifact Management
```yaml
# Version artifacts
- Docker images: semantic versioning
- Binaries: include git SHA
- Helm charts: match app version
```

## Reference Files

- **Docker Builder**: `.ai-config/agents/docker-builder.md`
- **Release Manager**: `.ai-config/agents/release-manager.md`
- **Container Security**: `.ai-config/skills/container-security/SKILL.md`
