# GitHubActions

# GitHub Actions Tutorial - Complete Guide

## Table of Contents
1. [Introduction to GitHub Actions](#introduction-to-github-actions)
2. [Core Concepts](#core-concepts)
3. [Workflow Structure](#workflow-structure)
4. [Setting Up Your First Workflow](#setting-up-your-first-workflow)
5. [Events and Triggers](#events-and-triggers)
6. [Jobs and Steps](#jobs-and-steps)
7. [Actions and Reusable Workflows](#actions-and-reusable-workflows)
8. [Secrets and Environment Variables](#secrets-and-environment-variables)
9. [Build and Test Workflows](#build-and-test-workflows)
10. [Deployment Workflows](#deployment-workflows)
11. [Matrix Strategies](#matrix-strategies)
12. [Conditional Execution](#conditional-execution)
13. [Caching and Artifacts](#caching-and-artifacts)
14. [Advanced Patterns](#advanced-patterns)
15. [Best Practices](#best-practices)

---

## Introduction to GitHub Actions

**What is GitHub Actions?**

GitHub Actions is a continuous integration/continuous deployment (CI/CD) platform that allows you to automate your build, test, and deployment pipeline. It's built directly into GitHub and executes code in response to events in your repository.

**Key Benefits:**
- Integrated directly into GitHub repositories
- No separate server needed
- Free tier for public repositories
- Generous free tier for private repositories (2000 minutes/month)
- Supports Linux, macOS, Windows runners
- Extensive marketplace of pre-built actions
- YAML-based configuration

**Common Use Cases:**
- Automated testing on pull requests
- Building and publishing packages
- Deploying applications
- Sending notifications
- Code quality checks
- Security scanning
- Release automation

---

## Core Concepts

### Key Terminology

**Workflow**
- Automated process defined in YAML file
- Located in `.github/workflows/` directory
- Runs in response to events
- Can be scheduled or manually triggered

**Event**
- Specific activity that triggers a workflow
- Examples: push, pull_request, schedule, manual trigger

**Job**
- Set of steps that execute on the same runner
- Can run in parallel or sequentially
- Each job runs in a fresh instance of the runner environment

**Step**
- Individual task within a job
- Can run commands or actions
- Steps share environment variables and file system within a job

**Action**
- Reusable unit of code
- Can be created or used from marketplace
- Abstracts complex operations into simple interface

**Runner**
- Machine that executes workflows
- GitHub-hosted runners available
- Self-hosted runners for custom needs

### Visual Workflow Flow

```
Repository Event (push, PR, schedule)
              ↓
         Workflow Triggered
              ↓
    ┌────────┴────────┐
    ↓                 ↓
  Job 1            Job 2
    ↓                 ↓
Step 1.1          Step 2.1
Step 1.2          Step 2.2
Step 1.3          Step 2.3
    ↓                 ↓
  Complete        Complete
```

---

## Workflow Structure

**Basic YAML Structure:**

```yaml
name: Workflow Name                    # Workflow display name
on: [push, pull_request]              # Triggers
env:                                   # Global environment variables
  REGISTRY: ghcr.io

jobs:
  job_id:                             # Unique job identifier
    name: Job Display Name
    runs-on: ubuntu-latest            # Runner OS
    env:                              # Job-level environment variables
      NODE_ENV: production
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Run script
        run: echo "Hello World"
```

**Workflow File Location:**
- Path: `.github/workflows/workflow-name.yml`
- All workflows in this directory are automatically discovered
- Multiple workflows can run independently

---

## Setting Up Your First Workflow

### Example 1: Simple Hello World Workflow

```yaml
name: Hello World Workflow

on:
  push:
    branches: [ main ]

jobs:
  hello-world:
    runs-on: ubuntu-latest
    steps:
      - name: Say Hello
        run: echo "Hello, World!"
      
      - name: Show Date
        run: date
      
      - name: List Files
        run: ls -la
```

**How to create:**
1. Go to your GitHub repository
2. Click "Actions" tab
3. Click "New workflow"
4. Select "set up a workflow yourself"
5. Paste the YAML code
6. Commit the file

### Example 2: Workflow with Checkout

```yaml
name: Repository Info

on:
  push:
    branches: [ main ]

jobs:
  repo-info:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: Get repository info
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit: ${{ github.sha }}"
          echo "Actor: ${{ github.actor }}"
      
      - name: Count files
        run: find . -type f | wc -l
      
      - name: Show directory structure
        run: tree -L 2 || find . -type d -maxdepth 2
```

---

## Events and Triggers

### Push Events

```yaml
on:
  push:
    branches:
      - main
      - 'release/**'              # Wildcard pattern
    paths:
      - 'src/**'                  # Only run if src/ changes
      - 'package.json'
    paths-ignore:
      - 'docs/**'                 # Ignore doc changes
      - '**.md'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: echo "Code pushed!"
```

### Pull Request Events

```yaml
on:
  pull_request:
    branches: [ main, develop ]
    types:
      - opened
      - synchronize
      - reopened
    paths:
      - 'src/**'
      - 'tests/**'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: echo "PR submitted!"
```

### Schedule Events (Cron)

```yaml
on:
  schedule:
    # Daily at 2 AM UTC
    - cron: '0 2 * * *'
    # Every Monday at 9 AM UTC
    - cron: '0 9 * * 1'
    # Every 6 hours
    - cron: '0 */6 * * *'

jobs:
  scheduled-task:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running scheduled task"
```

### Manual Trigger (Workflow Dispatch)

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        type: choice
        options:
          - staging
          - production
      version:
        description: 'Release version'
        required: true
        type: string

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to ${{ inputs.environment }}"
      - run: echo "Version: ${{ inputs.version }}"
```

### Multiple Event Types

```yaml
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 0 * * 0'
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm test
```

---

## Jobs and Steps

### Sequential Jobs

```yaml
name: Sequential Jobs

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build
        run: echo "Building..."
  
  test:
    needs: build              # Depends on build job
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Test
        run: echo "Testing..."
  
  deploy:
    needs: test               # Depends on test job
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: echo "Deploying..."
```

### Parallel Jobs

```yaml
name: Parallel Jobs

on: [push]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: echo "Linting..."
  
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: echo "Testing..."
  
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: echo "Security scan..."
```

### Complex Job Dependencies

```yaml
name: Complex Dependencies

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."
  
  test-unit:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running unit tests..."
  
  test-integration:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running integration tests..."
  
  deploy:
    needs: [test-unit, test-integration]  # Multiple dependencies
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

### Step Details

```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      # Simple command
      - name: Echo message
        run: echo "Hello"
      
      # Multiline script
      - name: Complex script
        run: |
          echo "Line 1"
          echo "Line 2"
          if [ -f "file.txt" ]; then
            cat file.txt
          fi
      
      # Script file
      - name: Run script file
        run: bash scripts/build.sh
      
      # Using specific shell
      - name: PowerShell example
        shell: pwsh
        run: Write-Host "Hello from PowerShell"
      
      # Python example
      - name: Python script
        shell: python
        run: print("Hello from Python")
```

---

## Actions and Reusable Workflows

### Using Existing Actions

```yaml
name: Using Actions

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # Checkout action
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      # Setup Node.js
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      # Install dependencies
      - name: Install dependencies
        run: npm ci
      
      # Run tests
      - name: Run tests
        run: npm test
      
      # Upload coverage
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/coverage-final.json
          flags: unittests
```

### Creating Custom Actions (Simple Shell Action)

**Action file: `.github/actions/hello-world/action.yml`**

```yaml
name: 'Hello World'
description: 'Say hello to someone'
inputs:
  who-to-greet:
    description: 'Who to greet'
    required: true
    default: 'World'
outputs:
  time:
    description: 'The time we greeted'
    value: ${{ steps.hello.outputs.time }}
runs:
  using: 'composite'
  steps:
    - id: hello
      shell: bash
      run: |
        echo "Hello ${{ inputs.who-to-greet }}!"
        echo "time=$(date)" >> $GITHUB_OUTPUT
```

**Using the custom action:**

```yaml
on: [push]

jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Hello from custom action
        uses: ./.github/actions/hello-world
        with:
          who-to-greet: 'GitHub User'
      
      - name: Display time
        run: echo "The time was ${{ steps.hello.outputs.time }}"
```

### Creating Custom Actions (Docker)

**Action file: `.github/actions/docker-action/action.yml`**

```yaml
name: 'Custom Docker Action'
description: 'Run a custom Docker container'
inputs:
  message:
    description: 'Message to display'
    required: true
runs:
  using: 'docker'
  image: 'docker://node:18'
  args:
    - ${{ inputs.message }}
```

**Dockerfile: `.github/actions/docker-action/Dockerfile`**

```dockerfile
FROM node:18
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

**Entrypoint: `.github/actions/docker-action/entrypoint.sh`**

```bash
#!/bin/bash
echo "Message: $1"
echo "Done!"
```

### Reusable Workflows

**Reusable workflow file: `.github/workflows/reusable-test.yml`**

```yaml
name: Reusable Test Workflow

on:
  workflow_call:
    inputs:
      node_version:
        required: true
        type: string
        default: '18'
    secrets:
      npm_token:
        required: false

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ inputs.node_version }}
          cache: 'npm'
          registry-url: 'https://registry.npmjs.org'
      
      - name: Install dependencies
        run: npm ci
        env:
          NODE_AUTH_TOKEN: ${{ secrets.npm_token }}
      
      - name: Run tests
        run: npm test
      
      - name: Generate coverage
        run: npm run coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

**Using reusable workflow:**

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node_version: '18'
    secrets:
      npm_token: ${{ secrets.NPM_TOKEN }}
```

---

## Secrets and Environment Variables

### Working with Secrets

**Creating Repository Secrets:**
1. Go to Repository Settings
2. Security → Secrets and variables → Actions
3. Click "New repository secret"
4. Add name and value

**Using Secrets in Workflow:**

```yaml
name: Using Secrets

on: [push]

env:
  PUBLIC_VAR: public_value

jobs:
  secure:
    runs-on: ubuntu-latest
    environment: production  # Optional: environment protection
    steps:
      - name: Use secret
        run: echo "Secret: ${{ secrets.DATABASE_URL }}"
        env:
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
      
      - name: API call with token
        run: |
          curl -H "Authorization: Bearer ${{ secrets.API_TOKEN }}" \
            https://api.example.com/data
      
      - name: npm auth
        run: npm set //registry.npmjs.org/:_authToken=${{ secrets.NPM_TOKEN }}
```

### Environment Variables

```yaml
name: Environment Variables

on: [push]

env:
  GLOBAL_VAR: global_value

jobs:
  env-example:
    runs-on: ubuntu-latest
    env:
      JOB_VAR: job_value
    steps:
      - name: Display variables
        env:
          STEP_VAR: step_value
        run: |
          echo "Global: $GLOBAL_VAR"
          echo "Job: $JOB_VAR"
          echo "Step: $STEP_VAR"
      
      - name: Built-in variables
        run: |
          echo "GitHub Actor: ${{ github.actor }}"
          echo "GitHub Repository: ${{ github.repository }}"
          echo "GitHub Ref: ${{ github.ref }}"
          echo "GitHub SHA: ${{ github.sha }}"
          echo "Workspace: ${{ github.workspace }}"
          echo "Runner OS: ${{ runner.os }}"
          echo "Runner Arch: ${{ runner.arch }}"
```

### Environment Protection

```yaml
name: Protected Deployment

on:
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    steps:
      - name: Deploy to production
        run: |
          echo "Deploying with approval required"
          curl -X POST https://api.example.com/deploy \
            -H "Authorization: Bearer ${{ secrets.DEPLOY_TOKEN }}"
```

---

## Build and Test Workflows

### Node.js Build and Test

```yaml
name: Node.js CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint code
        run: npm run lint
      
      - name: Run tests
        run: npm test -- --coverage
      
      - name: Build application
        run: npm run build
      
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella
```

### .NET Build and Test

```yaml
name: .NET CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        dotnet-version: ['6.0', '7.0', '8.0']
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup .NET ${{ matrix.dotnet-version }}
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: ${{ matrix.dotnet-version }}
      
      - name: Restore dependencies
        run: dotnet restore
      
      - name: Build
        run: dotnet build --no-restore --configuration Release
      
      - name: Run tests
        run: dotnet test --no-build --verbosity normal --logger "trx" --collect:"XPlat Code Coverage"
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: test-results
          path: '**/TestResults/*.trx'
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: '**/coverage.cobertura.xml'
```

### Python Build and Test

```yaml
name: Python CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        python-version: ['3.8', '3.9', '3.10', '3.11']
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov
      
      - name: Lint with flake8
        run: |
          flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
          flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
      
      - name: Test with pytest
        run: pytest --cov=./ --cov-report=xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

---

## Deployment Workflows

### Deploy to AWS

```yaml
name: Deploy to AWS

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Build Docker image
        run: |
          docker build -t my-app:${{ github.sha }} .
          docker tag my-app:${{ github.sha }} my-app:latest
      
      - name: Push to ECR
        run: |
          aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.us-east-1.amazonaws.com
          docker push ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.us-east-1.amazonaws.com/my-app:${{ github.sha }}
      
      - name: Update ECS service
        run: |
          aws ecs update-service --cluster production --service my-service --force-new-deployment
      
      - name: Wait for deployment
        run: aws ecs wait services-stable --cluster production --services my-service
```

### Deploy to Azure

```yaml
name: Deploy to Azure

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '7.0'
      
      - name: Build
        run: dotnet build --configuration Release
      
      - name: Publish
        run: dotnet publish -c Release -o ./publish
      
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Deploy to App Service
        uses: azure/webapps-deploy@v2
        with:
          app-name: my-app
          package: ./publish
          slot-name: production
      
      - name: Logout from Azure
        run: az logout
```

### Deploy Docker to Docker Hub

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

jobs:
  docker:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ secrets.DOCKERHUB_USERNAME }}/my-app
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=registry,ref=${{ secrets.DOCKERHUB_USERNAME }}/my-app:buildcache
          cache-to: type=registry,ref=${{ secrets.DOCKERHUB_USERNAME }}/my-app:buildcache,mode=max
```

### Deploy to GitHub Pages

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
      
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v2
        with:
          path: './dist'
  
  deploy:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

---

## Matrix Strategies

### Testing Multiple Versions

```yaml
name: Matrix Strategy

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [16.x, 18.x, 20.x]
        include:
          - os: ubuntu-latest
            node-version: 18.x
            coverage: true
        exclude:
          - os: macos-latest
            node-version: 16.x
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Upload coverage
        if: matrix.coverage == true
        uses: codecov/codecov-action@v3
```

### Complex Matrix

```yaml
name: Complex Matrix

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        python-version: ['3.9', '3.10', '3.11']
        django-version: ['3.2', '4.0', '4.2']
        database: [sqlite, postgres, mysql]
        include:
          # Add specific test configuration
          - python-version: '3.11'
            django-version: '4.2'
            database: postgres
            extra-args: '--slow'
        exclude:
          # Skip incompatible combinations
          - python-version: '3.9'
            django-version: '4.2'
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install Django==${{ matrix.django-version }}
          pip install psycopg2-binary  # For PostgreSQL
          pip install mysqlclient      # For MySQL
      
      - name: Run tests
        env:
          DATABASE: ${{ matrix.database }}
        run: pytest ${{ matrix.extra-args || '' }}
```

---

## Conditional Execution

### if Conditions

```yaml
name: Conditional Execution

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # Run only on main branch
      - name: Deploy to production
        if: github.ref == 'refs/heads/main'
        run: echo "Deploying to production"
      
      # Run only on pull requests
      - name: Comment on PR
        if: github.event_name == 'pull_request'
        run: echo "This is a pull request"
      
      # Run only on tags
      - name: Create release
        if: startsWith(github.ref, 'refs/tags/v')
        run: echo "Creating release"
      
      # Run only if previous step failed
      - name: Handle failure
        if: failure()
        run: echo "Previous step failed"
      
      # Run only if previous step succeeded
      - name: Continue on success
        if: success()
        run: echo "All previous steps succeeded"
      
      # Run always, even if cancelled
      - name: Cleanup
        if: always()
        run: echo "Cleanup step"
      
      # Multiple conditions
      - name: Complex condition
        if: github.event_name == 'push' && contains(github.ref, 'release')
        run: echo "Pushing to release branch"

  conditional-job:
    runs-on: ubuntu-latest
    # Job-level condition
    if: github.event_name == 'pull_request'
    steps:
      - run: echo "This job runs only for PRs"
```

### Expression Contexts

```yaml
name: Context Expressions

on:
  push:
  pull_request:
  workflow_dispatch:

jobs:
  context-demo:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: GitHub context
        run: |
          echo "Event: ${{ github.event_name }}"
          echo "Ref: ${{ github.ref }}"
          echo "SHA: ${{ github.sha }}"
          echo "Actor: ${{ github.actor }}"
          echo "Repository: ${{ github.repository }}"
      
      - name: Runner context
        run: |
          echo "OS: ${{ runner.os }}"
          echo "Architecture: ${{ runner.arch }}"
      
      - name: Conditional with contains
        if: contains(github.event.head_commit.message, '[skip-tests]')
        run: echo "Skipping tests based on commit message"
      
      - name: Conditional with format
        if: format('refs/heads/{0}', github.event.repository.default_branch) == github.ref
        run: echo "On default branch"
      
      - name: Conditional with fromJson
        if: fromJson('{"deploy": true}').deploy
        run: echo "Deploying..."
```

---

## Caching and Artifacts

### Caching Dependencies

```yaml
name: Caching Example

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # Node.js caching
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
  
  build-python:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install dependencies
        run: pip install -r requirements.txt
  
  build-manual-cache:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # Manual cache
      - name: Cache Maven packages
        uses: actions/cache@v3
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
          restore-keys: |
            ${{ runner.os }}-maven-
      
      - name: Build with Maven
        run: mvn clean install
```

### Artifacts

```yaml
name: Artifacts Example

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build application
        run: |
          mkdir build
          echo "build artifact" > build/app.jar
      
      - name: Upload build artifact
        uses: actions/upload-artifact@v3
        with:
          name: build-output
          path: build/
          retention-days: 5
  
  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v3
        with:
          name: build-output
      
      - name: Test artifact
        run: |
          ls -la
          cat app.jar
  
  upload-coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run tests with coverage
        run: |
          mkdir coverage
          echo "coverage data" > coverage/report.xml
      
      - name: Upload test coverage
        uses: actions/upload-artifact@v3
        with:
          name: coverage-report
          path: coverage/
      
      - name: Upload to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/report.xml
          flags: unittests
```

---

## Advanced Patterns

### Create Release

```yaml
name: Create Release

on:
  push:
    tags:
      - 'v*'

jobs:
  create-release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body: |
            Changes in this Release
            - First item
            - Second item
          draft: false
          prerelease: false
```

### Publish Package

```yaml
name: Publish Package

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    environment: release
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          registry-url: 'https://registry.npmjs.org'
      
      - name: Publish to npm
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
  
  publish-nuget:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '7.0'
      
      - name: Pack and publish to NuGet
        run: |
          dotnet pack --configuration Release
          dotnet nuget push **/*.nupkg --api-key ${{ secrets.NUGET_API_KEY }} --source https://api.nuget.org/v3/index.json
```

### Code Quality

```yaml
name: Code Quality

on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0  # Full history for SonarQube
      
      - name: Set up JDK
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: SonarQube Scan
        uses: SonarSource/sonarqube-scan-action@master
        env:
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      
      - name: CodeQL Analysis
        uses: github/codeql-action/init@v2
        with:
          languages: 'java'
      
      - name: Build for CodeQL
        run: mvn clean package
      
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2
```

### Notification

```yaml
name: Notifications

on:
  workflow_run:
    workflows: ['CI']
    types: [completed]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Slack Notification
        uses: slackapi/slack-github-action@v1.24.0
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "Workflow ${{ github.workflow }} completed",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Workflow Status*: ${{ github.event.workflow_run.conclusion }}"
                  }
                }
              ]
            }
      
      - name: Send Email Notification
        uses: dawidd6/action-send-mail@v3
        if: failure()
        with:
          server_address: ${{ secrets.EMAIL_SERVER }}
          server_port: ${{ secrets.EMAIL_PORT }}
          username: ${{ secrets.EMAIL_USERNAME }}
          password: ${{ secrets.EMAIL_PASSWORD }}
          subject: Workflow Failed - ${{ github.repository }}
          to: team@example.com
          from: noreply@github.com
          body: |
            Workflow: ${{ github.workflow }}
            Repository: ${{ github.repository }}
            Branch: ${{ github.ref }}
            Commit: ${{ github.sha }}
```

---

## Best Practices

### Security Best Practices

```yaml
name: Security Best Practices

on: [push, pull_request]

jobs:
  secure:
    runs-on: ubuntu-latest
    steps:
      # 1. Use specific action versions (not latest)
      - uses: actions/checkout@v3
        # Don't use: @latest or @main
      
      # 2. Use secrets for sensitive data
      - name: Use secrets
        run: |
          # Good
          curl -H "Authorization: Bearer ${{ secrets.API_TOKEN }}" https://api.example.com
          
          # Bad - Never hardcode
          # curl -H "Authorization: Bearer abc123xyz" https://api.example.com
      
      # 3. Use OIDC for cloud authentication
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::ACCOUNT_ID:role/github-actions-role
          aws-region: us-east-1
      
      # 4. Validate external inputs
      - name: Validate input
        run: |
          if ! [[ "${{ github.ref }}" =~ ^refs/tags/v ]]; then
            echo "Invalid tag format"
            exit 1
          fi
      
      # 5. Use pinned dependencies
      - uses: actions/setup-node@v3
        with:
          node-version: '18.0.0'  # Specific version, not 18.x
      
      # 6. Limit permissions with least privilege
      # See: permissions section at top of workflow
```

### Workflow Structure Best Practices

```yaml
name: Best Practices Workflow

# 1. Clear, descriptive name
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

# 2. Limit job permissions
permissions:
  contents: read
  checks: write
  pull-requests: write

env:
  # 3. Global environment variables
  REGISTRY: ghcr.io

jobs:
  # 4. Logical job names
  lint:
    name: Code Quality Checks
    runs-on: ubuntu-latest
    # 5. Job-specific permissions
    permissions:
      contents: read
    
    steps:
      # 6. Descriptive step names
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      # 7. Use official setup actions
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      # 8. Separate install and run
      - name: Install dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Format check
        run: npm run format:check
  
  test:
    name: Test Suite
    runs-on: ubuntu-latest
    needs: lint  # Explicit dependencies
    
    strategy:
      # 9. Test against multiple versions
      matrix:
        node-version: [16.x, 18.x, 20.x]
      # 10. Fail fast is optional
      fail-fast: false
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test -- --coverage
      
      # 11. Upload artifacts for analysis
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        if: matrix.node-version == '18.x'
  
  build:
    name: Build Application
    runs-on: ubuntu-latest
    needs: test
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
      
      # 12. Upload build artifacts
      - name: Upload build artifact
        uses: actions/upload-artifact@v3
        with:
          name: build-${{ github.sha }}
          path: dist/
          retention-days: 1

# 13. Use concurrency to cancel old runs
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

### Performance Best Practices

```yaml
name: Performance Optimized

on: [push, pull_request]

# Use concurrency to cancel redundant runs
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # 1. Parallelize jobs
  parallel-test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        test-suite: [unit, integration, e2e]
    steps:
      - uses: actions/checkout@v3
      - run: npm test -- --suite=${{ matrix.test-suite }}
  
  # 2. Use matrix for OS/version testing
  multi-os:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    steps:
      - uses: actions/checkout@v3
      - run: ./build.sh
  
  # 3. Cache dependencies
  cached-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'  # Built-in caching
      - run: npm ci      # Faster than npm install
      - run: npm run build
  
  # 4. Skip unnecessary jobs
  deploy:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
      - run: echo "Only deploy on main push"
```

---

## Workflow Debugging Tips

### View Workflow Logs

1. Go to Actions tab
2. Select the workflow run
3. Click on the job to view logs
4. Expand steps to see details

### Enable Debug Logging

```yaml
jobs:
  debug:
    runs-on: ubuntu-latest
    steps:
      - name: Enable debug logging
        env:
          RUNNER_DEBUG: '1'
        run: echo "Debug mode enabled"
```

### Common Issues and Solutions

| Issue | Solution |
|-------|----------|
| Secrets not available | Check secret is added to repo, use correct syntax `${{ secrets.NAME }}` |
| Cache not working | Ensure consistent cache key, check path exists |
| Job timeout | Increase timeout-minutes, optimize workflow |
| Permission denied | Check file permissions, use `chmod +x` for scripts |
| Action not found | Verify action syntax: `owner/repo@ref` |

---

## Real-World Examples

### Full CI/CD Pipeline

```yaml
name: Full CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  release:
    types: [created]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Stage 1: Code Quality
  quality:
    name: Code Quality
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run format:check
  
  # Stage 2: Tests
  test:
    name: Tests
    runs-on: ubuntu-latest
    needs: quality
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm test -- --coverage
      - uses: codecov/codecov-action@v3
  
  # Stage 3: Build
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: test
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    steps:
      - uses: actions/checkout@v3
      - uses: docker/setup-buildx-action@v2
      - uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/metadata-action@v4
        id: meta
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha
      - uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
  
  # Stage 4: Deploy
  deploy:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production
    steps:
      - uses: actions/checkout@v3
      - uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::${{ secrets.AWS_ACCOUNT_ID }}:role/github-actions-role
          aws-region: us-east-1
      - run: |
          aws ecs update-service \
            --cluster production \
            --service my-app \
            --force-new-deployment
```

---

## Conclusion

GitHub Actions is a powerful CI/CD platform that integrates seamlessly with GitHub. Key takeaways:

1. **Start Simple:** Begin with basic workflows and expand gradually
2. **Use Official Actions:** Leverage marketplace and official setup actions
3. **Security First:** Always use secrets for sensitive data
4. **Test Locally:** Use tools like `act` to test workflows locally
5. **Documentation:** Keep workflows documented and readable
6. **Monitor Performance:** Watch for slow jobs and optimize
7. **Reuse Code:** Create reusable workflows and actions
8. **Permissions:** Apply least privilege principle

**Next Steps:**
- Create your first workflow
- Explore the Actions Marketplace
- Set up CI for your project
- Configure automated deployments
- Monitor and optimize workflows

**Resources:**
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax Reference](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)

This comprehensive guide covers GitHub Actions from basics to advanced patterns. Happy automating!
