# CI/CD 流水线指南

通过持续集成和交付流水线自动化 Ralph Orchestrator 的部署。

## 概述

本指南涵盖如何为以下场景设置自动化流水线：
- 代码测试和验证
- Docker 镜像构建和推送
- 文档部署
- 自动化发布
- 多环境部署

## GitHub Actions

### 完整的 CI/CD 工作流

创建 `.github/workflows/ci-cd.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
    tags: [ 'v*' ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # 测试任务
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.9', '3.10', '3.11']

    steps:
    - uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install uv
      run: |
        curl -LsSf https://astral.sh/uv/install.sh | sh
        echo "$HOME/.cargo/bin" >> $GITHUB_PATH

    - name: Install dependencies
      run: |
        uv venv
        uv pip install -e .
        uv pip install pytest pytest-cov pytest-asyncio

    - name: Run tests
      run: |
        source .venv/bin/activate
        pytest tests/ -v --cov=ralph_orchestrator --cov-report=xml

    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml
        fail_ci_if_error: true

  # 代码检查和安全扫描
  quality:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'

    - name: Install tools
      run: |
        pip install ruff black mypy bandit safety

    - name: Run ruff
      run: ruff check .

    - name: Check formatting
      run: black --check .

    - name: Type checking
      run: mypy ralph_orchestrator.py

    - name: Security scan
      run: |
        bandit -r . -ll
        safety check

  # 构建 Docker 镜像
  build:
    needs: [test, quality]
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
    - uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
          type=sha,prefix={{branch}}-

    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
        platforms: linux/amd64,linux/arm64

  # 部署文档
  docs:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'

    - name: Install MkDocs
      run: |
        pip install mkdocs mkdocs-material pymdown-extensions

    - name: Build documentation
      run: mkdocs build

    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      if: github.ref == 'refs/heads/main'
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./site

  # 部署到预发布环境
  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging

    steps:
    - uses: actions/checkout@v4

    - name: Deploy to Kubernetes (Staging)
      env:
        KUBE_CONFIG: ${{ secrets.STAGING_KUBE_CONFIG }}
      run: |
        echo "$KUBE_CONFIG" | base64 -d > kubeconfig
        export KUBECONFIG=kubeconfig
        kubectl set image deployment/ralph-orchestrator \
          ralph=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:develop \
          -n ralph-staging
        kubectl rollout status deployment/ralph-orchestrator -n ralph-staging

  # 部署到生产环境
  deploy-production:
    needs: build
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    environment: production

    steps:
    - uses: actions/checkout@v4

    - name: Deploy to Kubernetes (Production)
      env:
        KUBE_CONFIG: ${{ secrets.PROD_KUBE_CONFIG }}
      run: |
        echo "$KUBE_CONFIG" | base64 -d > kubeconfig
        export KUBECONFIG=kubeconfig
        VERSION=${GITHUB_REF#refs/tags/}
        kubectl set image deployment/ralph-orchestrator \
          ralph=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:$VERSION \
          -n ralph-production
        kubectl rollout status deployment/ralph-orchestrator -n ralph-production

    - name: Create Release
      uses: softprops/action-gh-release@v1
      with:
        generate_release_notes: true
        files: |
          dist/*.whl
          dist/*.tar.gz
```

### 文档工作流

创建 `.github/workflows/docs.yml`:

```yaml
name: Deploy Documentation

on:
  push:
    branches: [ main ]
    paths:
      - 'docs/**'
      - 'mkdocs.yml'
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0  # 完整历史记录用于 git 信息

    - name: Configure Git
      run: |
        git config user.name github-actions
        git config user.email github-actions@github.com

    - uses: actions/setup-python@v4
      with:
        python-version: '3.11'

    - name: Install dependencies
      run: |
        pip install mkdocs-material mkdocs-git-revision-date-localized-plugin
        pip install mkdocs-minify-plugin mkdocs-redirects

    - name: Build and Deploy
      run: |
        mkdocs gh-deploy --force --clean --verbose
```

### 发布工作流

创建 `.github/workflows/release.yml`:

```yaml
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

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'

    - name: Build distribution
      run: |
        pip install build
        python -m build

    - name: Publish to PyPI
      uses: pypa/gh-action-pypi-publish@release/v1
      with:
        password: ${{ secrets.PYPI_API_TOKEN }}

    - name: Create GitHub Release
      uses: softprops/action-gh-release@v1
      with:
        files: dist/*
        generate_release_notes: true
        body: |
          ## Docker Image
          \`\`\`bash
          docker pull ghcr.io/${{ github.repository }}:${{ github.ref_name }}
          \`\`\`

          ## Changelog
          See [CHANGELOG.md](CHANGELOG.md) for details.
```

## GitLab CI/CD

创建 `.gitlab-ci.yml`:

```yaml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: ""
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

# 测试阶段
test:unit:
  stage: test
  image: python:3.11
  script:
    - pip install uv
    - uv venv && source .venv/bin/activate
    - uv pip install -e . pytest pytest-cov
    - pytest tests/ --cov=ralph_orchestrator
  coverage: '/TOTAL.*\s+(\d+%)$/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml

test:lint:
  stage: test
  image: python:3.11
  script:
    - pip install ruff black
    - ruff check .
    - black --check .

# 构建阶段
build:docker:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
    - |
      if [ "$CI_COMMIT_BRANCH" == "main" ]; then
        docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:latest
        docker push $CI_REGISTRY_IMAGE:latest
      fi

# 部署阶段
deploy:staging:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: staging
    url: https://staging.ralph.example.com
  only:
    - develop
  script:
    - kubectl set image deployment/ralph ralph=$IMAGE_TAG -n staging
    - kubectl rollout status deployment/ralph -n staging

deploy:production:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: production
    url: https://ralph.example.com
  only:
    - tags
  when: manual
  script:
    - kubectl set image deployment/ralph ralph=$CI_REGISTRY_IMAGE:$CI_COMMIT_TAG -n production
    - kubectl rollout status deployment/ralph -n production
```

## Jenkins 流水线

创建 `Jenkinsfile`:

```groovy
pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_IMAGE = 'mikeyobrien/ralph-orchestrator'
        DOCKER_CREDENTIALS = 'docker-hub-credentials'
        KUBECONFIG_STAGING = credentials('kubeconfig-staging')
        KUBECONFIG_PROD = credentials('kubeconfig-production')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh '''
                            python -m venv venv
                            . venv/bin/activate
                            pip install -e . pytest pytest-cov
                            pytest tests/ --junitxml=test-results.xml
                        '''
                        junit 'test-results.xml'
                    }
                }

                stage('Lint') {
                    steps {
                        sh '''
                            pip install ruff
                            ruff check .
                        '''
                    }
                }

                stage('Security Scan') {
                    steps {
                        sh '''
                            pip install bandit safety
                            bandit -r . -f json -o bandit-report.json
                            safety check --json
                        '''
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", DOCKER_CREDENTIALS) {
                        def image = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                        image.push()
                        if (env.BRANCH_NAME == 'main') {
                            image.push('latest')
                        }
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh '''
                    export KUBECONFIG=${KUBECONFIG_STAGING}
                    kubectl set image deployment/ralph ralph=${DOCKER_IMAGE}:${BUILD_NUMBER} -n staging
                    kubectl rollout status deployment/ralph -n staging
                '''
            }
        }

        stage('Deploy to Production') {
            when {
                tag pattern: "v\\d+\\.\\d+\\.\\d+", comparator: "REGEXP"
            }
            input {
                message "Deploy to production?"
                ok "Deploy"
            }
            steps {
                sh '''
                    export KUBECONFIG=${KUBECONFIG_PROD}
                    kubectl set image deployment/ralph ralph=${DOCKER_IMAGE}:${TAG_NAME} -n production
                    kubectl rollout status deployment/ralph -n production
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            slackSend(
                color: 'good',
                message: "Build Successful: ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Build Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
            )
        }
    }
}
```

## CircleCI 配置

创建 `.circleci/config.yml`:

```yaml
version: 2.1

orbs:
  python: circleci/python@2.1.1
  docker: circleci/docker@2.2.0
  kubernetes: circleci/kubernetes@1.3.1

jobs:
  test:
    docker:
      - image: cimg/python:3.11
    steps:
      - checkout
      - python/install-packages:
          pkg-manager: pip
      - run:
          name: Run tests
          command: |
            pip install pytest pytest-cov
            pytest tests/ --cov=ralph_orchestrator
      - store_test_results:
          path: test-results
      - store_artifacts:
          path: coverage

  build-and-push:
    executor: docker/docker
    steps:
      - setup_remote_docker
      - checkout
      - docker/check
      - docker/build:
          image: mikeyobrien/ralph-orchestrator
          tag: ${CIRCLE_SHA1}
      - docker/push:
          image: mikeyobrien/ralph-orchestrator
          tag: ${CIRCLE_SHA1}

  deploy:
    docker:
      - image: cimg/base:stable
    steps:
      - kubernetes/install-kubectl
      - run:
          name: Deploy to Kubernetes
          command: |
            echo $KUBE_CONFIG | base64 -d > kubeconfig
            export KUBECONFIG=kubeconfig
            kubectl set image deployment/ralph ralph=mikeyobrien/ralph-orchestrator:${CIRCLE_SHA1}
            kubectl rollout status deployment/ralph

workflows:
  main:
    jobs:
      - test
      - build-and-push:
          requires:
            - test
          filters:
            branches:
              only: main
      - deploy:
          requires:
            - build-and-push
          filters:
            branches:
              only: main
```

## Azure DevOps 流水线

创建 `azure-pipelines.yml`:

```yaml
trigger:
  branches:
    include:
    - main
    - develop
  tags:
    include:
    - v*

pool:
  vmImage: 'ubuntu-latest'

variables:
  dockerRegistry: 'your-registry.azurecr.io'
  dockerImageName: 'ralph-orchestrator'
  kubernetesServiceConnection: 'k8s-connection'

stages:
- stage: Test
  jobs:
  - job: TestJob
    steps:
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.11'

    - script: |
        pip install uv
        uv venv && source .venv/bin/activate
        uv pip install -e . pytest pytest-cov
        pytest tests/ --junitxml=test-results.xml
      displayName: 'Run tests'

    - task: PublishTestResults@2
      inputs:
        testResultsFiles: 'test-results.xml'
        testRunTitle: 'Python Tests'

- stage: Build
  dependsOn: Test
  jobs:
  - job: BuildJob
    steps:
    - task: Docker@2
      inputs:
        containerRegistry: $(dockerRegistry)
        repository: $(dockerImageName)
        command: buildAndPush
        Dockerfile: Dockerfile
        tags: |
          $(Build.BuildId)
          latest

- stage: Deploy
  dependsOn: Build
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployJob
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: Kubernetes@1
            inputs:
              connectionType: 'Kubernetes Service Connection'
              kubernetesServiceEndpoint: $(kubernetesServiceConnection)
              command: 'set'
              arguments: 'image deployment/ralph ralph=$(dockerRegistry)/$(dockerImageName):$(Build.BuildId)'
              namespace: 'production'
```

## ArgoCD GitOps

创建 `argocd/application.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ralph-orchestrator
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/mikeyobrien/ralph-orchestrator
    targetRevision: HEAD
    path: k8s
    helm:
      valueFiles:
        - values.yaml
      parameters:
        - name: image.tag
          value: latest
  destination:
    server: https://kubernetes.default.svc
    namespace: ralph-production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - CreateNamespace=true
      - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

## Tekton 流水线

创建 `tekton/pipeline.yaml`:

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: ralph-ci-pipeline
spec:
  params:
    - name: repo-url
      type: string
    - name: revision
      type: string
      default: main
  workspaces:
    - name: shared-workspace
  tasks:
    - name: fetch-source
      taskRef:
        name: git-clone
      workspaces:
        - name: output
          workspace: shared-workspace
      params:
        - name: url
          value: $(params.repo-url)
        - name: revision
          value: $(params.revision)

    - name: run-tests
      runAfter:
        - fetch-source
      taskRef:
        name: pytest
      workspaces:
        - name: source
          workspace: shared-workspace

    - name: build-image
      runAfter:
        - run-tests
      taskRef:
        name: buildah
      workspaces:
        - name: source
          workspace: shared-workspace
      params:
        - name: IMAGE
          value: ghcr.io/mikeyobrien/ralph-orchestrator

    - name: deploy
      runAfter:
        - build-image
      taskRef:
        name: kubernetes-actions
      params:
        - name: script
          value: |
            kubectl set image deployment/ralph ralph=ghcr.io/mikeyobrien/ralph-orchestrator:$(params.revision)
            kubectl rollout status deployment/ralph
```

## 监控和通知

### Slack 通知

在任何 CI/CD 平台上添加：

```yaml
# GitHub Actions 示例
- name: Slack Notification
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: 'Deployment ${{ job.status }}'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
  if: always()
```

### 健康检查

```yaml
# 部署后验证
- name: Health Check
  run: |
    for i in {1..30}; do
      if curl -f http://ralph.example.com/health; then
        echo "Service is healthy"
        exit 0
      fi
      sleep 10
    done
    echo "Health check failed"
    exit 1
```

## 最佳实践

1. **版本管理**: 使用语义化版本标记发布
2. **自动化测试**: 每次提交都运行测试
3. **安全扫描**: 包含 SAST 和依赖扫描
4. **渐进式部署**: 使用预发布环境
5. **回滚策略**: 在失败时实现自动回滚
6. **密钥管理**: 永远不要提交密钥，使用 CI/CD 密钥
7. **制品存储**: 存储构建制品以实现可重现性
8. **监控**: 跟踪部署指标和成功率
9. **文档**: 每次发布时更新文档
10. **合规性**: 所有部署的审计跟踪

## 下一步

- [生产部署指南](production.md) - 生产环境最佳实践
- [监控设置](../advanced/monitoring.md) - 可观测性配置
- [安全指南](../advanced/security.md) - 安全加固
