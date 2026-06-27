# 18 CI/CD Pipelines

This guide covers CI/CD concepts, GitHub Actions mechanics, workflow syntax, parallelism, secret management, Docker integration, and automated deployments.

## CI/CD Concepts

### Definition
Continuous Integration (CI) is the practice of automating the integration of code changes from multiple contributors into a single shared repository. Continuous Delivery (CD) automates release preparation, while Continuous Deployment automatically deploys every passing build directly to production.

### Real-world Analogy
Imagine a bakery. CI is checking ingredients: every time a chef brings fruit (code), a scale checks the weight and freshness automatically (automated tests). Continuous Delivery is boxing the cakes and placing them on the shipping dock, waiting for the store manager to sign the release manifest. Continuous Deployment is loading the boxes directly onto a conveyor belt that carries them straight to the storefront counter.

### Code Example
```
// Workflow sequence mapping:
// Developer pushes branch -> CI runs tests -> Build compiles -> Deploy to staging (CD Delivery) -> Deploy to production (CD Deployment)
```

### Common Interview Questions
- Contrast Continuous Delivery (CD) and Continuous Deployment (CD).
- What are the business and technical benefits of maintaining short-feedback CI pipelines?
- What role do integration and smoke tests play in CD pipelines?

### Reference Links
- [Martin Fowler: Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html)
- [Atlassian: CI vs CD vs Continuous Deployment](https://www.atlassian.com/continuous-delivery/continuous-integration/comparison)

## GitHub Actions Fundamentals

### Definition
GitHub Actions is an API-integrated CI/CD automation platform. Workflows are defined inside YAML files in the .github/workflows directory and are composed of Workflows (processes), Triggers (events), Jobs (runner execution blocks), Steps (tasks), and Actions (reusable plugins).

### Real-world Analogy
Imagine a warehouse checklist. The checklist dictates: "On delivery arrival (Trigger), open the doors (Workflow). Task 1 (Job): Clean the floor. Task 2 (Job): Unpack boxes. Under Task 2, step 1 is reading the shipping manifest, step 2 is scanning labels (Steps). We use a barcode scanner bought from a supplier (Action) to speed up scans."

### Code Example
```yaml
# .github/workflows/ci.yml
name: Node CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test
```

### Common Interview Questions
- Explain the role of the actions/checkout action in workflows.
- What are GitHub Runners and how do host-managed runners differ from self-hosted runners?
- How do steps communicate or share files within a single Job?

### Reference Links
- [GitHub Docs: GitHub Actions Quickstart](https://docs.github.com/en/actions/quickstart)
- [GitHub Docs: Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

## Workflow Triggers

### Definition
Workflow triggers are specific repository events that prompt GitHub Actions to execute workflows. Examples include code events (push, pull_request), cron schedules (schedule), manual triggers (workflow_dispatch), or external API dispatches (repository_dispatch).

### Real-world Analogy
Imagine a house security alarm. Triggers are events: someone walks through the front door (push), someone rings the doorbell (pull_request), the clock hits midnight (schedule), or you press the "Test Alarm" button on the wall (manual workflow_dispatch).

### Code Example
```yaml
# Workflow trigger configurations
on:
  # Schedule trigger (every day at midnight)
  schedule:
    - cron: '0 0 * * *'
  # Manual trigger
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target deployment environment'
        required: true
        default: 'staging'
```

### Common Interview Questions
- How do you configure a cron trigger in GitHub Actions and what is the timezone default?
- What parameters does the workflow_dispatch trigger accept?
- Explain how repository_dispatch can trigger workflows from external webhooks.

### Reference Links
- [GitHub Docs: Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

## Jobs and Parallelism

### Definition
Jobs represent execution blocks that run on separate virtual machines. By default, jobs execute in parallel. The 'needs' keyword configures sequential dependencies. Matrix builds allow you to duplicate a job configurations across multiple variables (e.g. Node versions, OS systems).

### Real-world Analogy
Imagine a car wash. Job 1 is vacuuming the inside. Job 2 is washing the outer chassis. These run in parallel because one worker vacuums while another washes. Job 3 is blow-drying: it needs (dependencies) Job 2 to finish first. A Matrix build is washing 3 cars (red, blue, green) at the same time using the exact same instructions.

### Code Example
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20] # Run build on both versions in parallel
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm run build

  deploy:
    runs-on: ubuntu-latest
    needs: build # Runs only if build job succeeds
    steps:
      - run: echo "Deploying..."
```

### Common Interview Questions
- How do you pass build artifacts (e.g. static assets) between different jobs (e.g. upload-artifact)?
- How does the matrix strategy prevent code duplication in CI configuration files?
- What happens if one job in a parallel matrix array fails?

### Reference Links
- [GitHub Docs: Using a Matrix Strategy](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)
- [GitHub Docs: Storing Workflow Data as Artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)

## Managing Secrets in CI

### Definition
Secrets in CI are encrypted credentials (API keys, database passwords, SSH keys) configured in GitHub repository settings. Workflows access secrets using the secret context, preventing sensitive credentials from being committed to source code files.

### Real-world Analogy
Imagine a safety lockbox in a workspace room. The company password is kept inside the lockbox. When workers write instruction guides, they don't print the actual password on the page. They write: "Get password from the vault (secret reference)". The system opens the vault and reads the password during the task, locking it back immediately.

### Code Example
```yaml
steps:
  - name: Deploy to Vercel
    run: vercel --token ${{ secrets.VERCEL_TOKEN }} --prod
    env:
      DATABASE_URL: ${{ secrets.PROD_DB_URL }}
```

### Common Interview Questions
- Why are secrets masked (replaced with ***) in GitHub Actions console logs?
- How do Environment-specific secrets differ from Repository-wide secrets?
- What are the security risks of accessing secrets inside pull_request workflows from forked repositories?

### Reference Links
- [GitHub Docs: Encrypted Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

## Linting and Testing in CI

### Definition
Linting and testing in CI verify code quality and functionality before merging. Pipelines cache node_modules folders between runs using cache actions to reduce dependency install times.

### Real-world Analogy
Imagine submitting an article to a newspaper. The first desk editor checks spelling and font rules (linting). The second reviewer fact-checks every address and quote (testing). To speed up the reviews, the editors keep a drawer of dictionary catalogs (caching) instead of buying fresh dictionaries on every article submission.

### Code Example
```yaml
steps:
  - uses: actions/checkout@v4
  
  # Caching node_modules based on lock file hash
  - name: Cache Node Modules
    uses: actions/cache@v4
    with:
      path: ~/.npm
      key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
      restore-keys: |
        ${{ runner.os }}-node-

  - run: npm ci
  - run: npm run lint
  - run: npm test
```

### Common Interview Questions
- How does actions/cache decide if it should perform a cache hit or cache miss?
- What is the difference between npm install and npm ci regarding package lock files in CI?
- Why should linting checks execute before testing steps in a workflow?

### Reference Links
- [GitHub Docs: Caching Dependencies](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)

## Building and Pushing Docker Images in CI

### Definition
Building and pushing Docker images in CI automates container deployment. Workflows log into registries (Docker Hub, AWS ECR), compile images, tag them with branch name or commit SHA, and push them to storage repositories.

### Real-world Analogy
Imagine an automated packaging center. The center logs into the shipping catalog (Docker registry authentication), builds a shipping container box from local files (Docker build), labels it with a serial shipping tracking number (commit SHA tag), and slides it onto the loading dock truck (push).

### Code Example
```yaml
# Build and Push Docker image using GitHub Actions
jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Build and Push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: user/myapp:latest,user/myapp:${{ github.sha }}
```

### Common Interview Questions
- Why should you tag production Docker images with unique commit SHAs rather than relying on 'latest'?
- Explain how buildx optimizes Docker builds in CI environments.
- How do you configure multi-platform builds (e.g. arm64 and amd64) in GitHub Actions?

### Reference Links
- [Docker Docs: Build Images in CI](https://docs.docker.com/build/ci/github-actions/)

## Scenario-based Interview Questions for CI/CD

### Scenario 1
Your team's pull request workflow takes 25 minutes to complete, slowing down developers. The slow steps are: installing npm dependencies (5 mins), running unit tests (10 mins), and running integration tests (10 mins). How do you optimize the pipeline?

*Expected Approach:*
1. Implement caching for npm dependencies (`actions/cache` or `actions/setup-node` caching configuration) to reduce install times from 5 minutes to under 1 minute.
2. Split the pipeline into parallel jobs: run unit tests and integration tests in parallel on separate runners, cutting down total test execution time from 20 minutes to 10 minutes.
3. Use a faster test runner config (e.g. Vitest instead of Jest or enabling swc compilers).

### Scenario 2
You are configuring a deployment pipeline for an API service. When new code is merged, you want to push the change directly to production. How do you deploy automatically without causing user downtime if a bad update crashes the API?

*Expected Approach:*
1. Implement a Rolling Update or Blue-Green Deployment strategy during the CD deployment step.
2. If deploying to AWS or Kubernetes, use health probes (liveness/readiness checks): the server router directs traffic only to the new containers if they pass verification checks, keeping the old service online if the new version fails to boot.
3. Add a rollback step in the workflow to trigger automatic deployment rollbacks to the last stable commit SHA tag if the deployment script logs errors.
