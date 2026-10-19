
# DevOps TD02 & TP02 - Git and GitHub Actions

## 1. Project Overview

This repository contains the work completed for the DevOps Git and GitHub Actions exercises.

The objective is to implement a Continuous Integration and Continuous Delivery (CI/CD) pipeline for a three-tier Docker application.

The project uses:

- Git and GitHub for version control.
- GitHub Actions for CI/CD automation.
- Maven and Java 21 for backend compilation and testing.
- Testcontainers for integration testing.
- SonarQube Cloud for static code analysis and Quality Gate validation.
- Docker Hub for container image publication.

**GitHub repository:**

https://github.com/cz632804-cpu/tp-devops-correction-docker

**Docker Hub account:** `nifdgg`

**SonarQube Cloud organization:** `cz632804-cpu`

## 2. Application Architecture

The application contains three components:

| Component | Technology | Directory |
|---|---|---|
| Backend API | Java / Spring Boot | `simple-api/` |
| Database | PostgreSQL | `database/` |
| HTTP Server | Apache HTTP Server | `http-server/` |

Each component has its own Dockerfile.

The application can be orchestrated using the provided `docker-compose.yaml` file.

### Architecture Diagram

```text
              Client
                 |
                 v
         Apache HTTP Server
                 |
                 v
          Spring Boot API
                 |
                 v
          PostgreSQL Database
```

The backend contains unit and integration tests that are executed automatically by GitHub Actions.

## 3. TD02 - Git and GitHub

### 3.1. Git and Version Control

Git is a distributed Version Control System (VCS).

It allows developers to:

- Track changes to source code.
- Maintain a history of commits.
- Work on separate branches.
- Collaborate through shared repositories.
- Merge changes and resolve conflicts.
- Restore previous versions when necessary.

### 3.2. GitHub Repository

The repository was created by forking the original training repository:

https://github.com/maximepakula/tp-devops-correction-docker

Our fork:

https://github.com/cz632804-cpu/tp-devops-correction-docker

The repository was cloned locally and used to complete the GitHub Actions exercises.

### 3.3. Essential Git Commands

Check the repository status:

```bash
git status
```

View commit history:

```bash
git log --oneline
```

Inspect local modifications:

```bash
git diff
```

Stage changes:

```bash
git add .
```

Create a commit:

```bash
git commit -m "Describe changes"
```

Push commits to GitHub:

```bash
git push origin main
```

Create a development branch:

```bash
git switch -c develop
```

Push the development branch:

```bash
git push -u origin develop
```

Return to the main branch:

```bash
git switch main
```

### 3.4. Branches

The project uses:

- `main`: the main branch used for Continuous Delivery.
- `develop`: a development branch used for Continuous Integration testing.

Branch separation prevents development changes from automatically publishing production-oriented Docker images.

## 4. TP02 - Continuous Integration

### 4.1. GitHub Actions

GitHub Actions is a CI/CD automation service integrated into GitHub.

A workflow is defined in a YAML file stored in the `.github/workflows/` directory.

A workflow contains:

- **Events:** actions that trigger the workflow, such as a push or pull request.
- **Jobs:** groups of steps executed on a runner.
- **Steps:** individual commands or reusable actions.
- **Actions:** reusable automation components.

The initial implementation used a single file:

```text
.github/workflows/main.yml
```

After completing the advanced exercise, the project was reorganized into two workflows:

```text
.github/
└── workflows/
    ├── ci.yml
    └── cd.yml
```

### 4.2. CI Workflow

File: `.github/workflows/ci.yml`

The CI workflow runs automatically when code is pushed to `main` or `develop`, or when a pull request targets these branches.

It performs the following tasks:

1. Checks out the repository.
2. Configures Java 21.
3. Compiles the backend using Maven.
4. Runs unit and integration tests.
5. Performs SonarQube Cloud analysis.
6. Waits for the Quality Gate result.

The workflow runs on an Ubuntu 24.04 GitHub-hosted runner.

### 4.3. Maven Build and Tests

The following command is executed in the `simple-api` directory:

```bash
mvn -B clean verify
```

Explanation:

- `clean` removes generated files from previous builds.
- `verify` compiles the project, runs tests, and verifies that the project meets the configured build requirements.
- `-B` enables batch mode, which is appropriate for CI systems.

The Maven test suite includes unit and integration tests.

During CI validation, the backend tests completed successfully.

### 4.4. Question 2-1: What are Testcontainers?

Testcontainers is a Java library that provides lightweight, disposable Docker containers for automated tests.

It allows integration tests to run with real services, such as PostgreSQL, instead of relying only on mocks or an externally managed test database.

In this project, Testcontainers is used to provide a PostgreSQL database during integration testing.

**Advantages:**

- Tests run in an isolated environment.
- The database environment is reproducible.
- Containers are created automatically for tests.
- Tests can use a real database engine.
- Developers and CI runners do not need to configure a permanent test database manually.

Testcontainers improves integration test reliability by reducing differences between environments.

## 5. TP02 - Continuous Delivery

### 5.1. Docker Hub Integration

Continuous Delivery automates the process of building and publishing the application's Docker images.

Our images are published to Docker Hub.

The following repositories are used:

| Component | Docker Hub image |
|---|---|
| Backend API | `nifdgg/tp-devops-simple-api:latest` |
| Database | `nifdgg/tp-devops-database:latest` |
| HTTP Server | `nifdgg/tp-devops-http-server:latest` |

Repository links:

- https://hub.docker.com/r/nifdgg/tp-devops-simple-api
- https://hub.docker.com/r/nifdgg/tp-devops-database
- https://hub.docker.com/r/nifdgg/tp-devops-http-server

### 5.2. GitHub Secrets

The Docker Hub credentials are stored using GitHub Actions Repository Secrets.

Required secrets:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
SONAR_TOKEN
```

- `DOCKERHUB_USERNAME` identifies the Docker Hub account.
- `DOCKERHUB_TOKEN` authenticates image publication.
- `SONAR_TOKEN` authenticates SonarQube Cloud analysis.

These secret values are never committed to the repository.

### 5.3. Question 2-2: Why do we need secured variables?

Secured variables prevent sensitive information, such as passwords and API tokens, from being stored directly in source code.

Storing credentials in a public GitHub repository could allow unauthorized users to access external services.

GitHub Actions Secrets provide encrypted storage and controlled access to sensitive configuration values.

**Benefits include:**

- Avoiding hardcoded credentials.
- Reducing the risk of exposing access tokens.
- Keeping authentication separate from application code.
- Allowing credentials to be changed without modifying source files.
- Supporting automated authentication inside GitHub Actions.

Secrets must still be handled carefully because workflows with access to them can use these credentials.

### 5.4. Docker Image Building

The CD workflow builds three Docker images using `docker/build-push-action@v6`.

Each Docker image has its own build context:

```text
./simple-api
./database
./http-server
```

The images are tagged using the Docker Hub account name and the `latest` tag.

The workflow authenticates to Docker Hub using `docker/login-action@v3`.

### 5.5. Question 2-3: Why use `needs` between CI and CD jobs?

The `needs` keyword creates an execution dependency between GitHub Actions jobs.

Without this dependency, jobs may execute in parallel.

This could result in Docker images being built and published before the application tests have finished.

Using:

```yaml
needs: test-backend
```

ensures that the Docker publication job only starts after the backend validation job has completed successfully.

This helps prevent invalid or untested application versions from being published.

In our final implementation, CI and CD are separate workflows.

Therefore, the same dependency is implemented using the `workflow_run` event and an explicit check that the CI workflow completed successfully.

### 5.6. Question 2-4: Why do we need to push Docker images?

Pushing Docker images to an online registry makes the built application available to other developers, servers, and deployment environments.

It allows a previously built image to be downloaded and reused without recompiling the source code.

**Advantages include:**

- Sharing images across environments.
- Simplifying application deployment.
- Supporting automated release workflows.
- Reducing repeated manual builds.
- Improving reproducibility.
- Managing image versions through repository tags.

Docker Hub is the image registry used in this project.

## 6. SonarQube Cloud and Quality Gate

### 6.1. Objective

SonarQube Cloud provides automated static code analysis.

It can identify:

- Bugs and reliability issues.
- Security vulnerabilities.
- Maintainability problems.
- Code duplication.
- Technical debt.
- Test coverage information.

The objective is to detect code quality issues before publishing application images.

### 6.2. SonarQube Cloud Configuration

**Organization Key:**

```text
cz632804-cpu
```

**Project Key:**

```text
cz632804-cpu_tp-devops-correction-docker
```

The analysis uses the default `Sonar way` Quality Gate.

Authentication is performed using the `SONAR_TOKEN` GitHub Actions secret.

### 6.3. Maven Analysis

The CI workflow executes the SonarQube Maven plugin using its full coordinates:

```bash
mvn -B org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
  -Dsonar.projectKey=cz632804-cpu_tp-devops-correction-docker \
  -Dsonar.organization=cz632804-cpu \
  -Dsonar.host.url=https://sonarcloud.io \
  -Dsonar.token="$SONAR_TOKEN" \
  -Dsonar.qualitygate.wait=true
```

The full plugin coordinates avoid Maven plugin-prefix resolution problems.

The option:

```text
-Dsonar.qualitygate.wait=true
```

makes the CI workflow wait for the Quality Gate result.

If the Quality Gate fails, the CI job fails, preventing the dependent publication workflow from releasing Docker images.

The token is supplied through an environment variable; the actual secret value is not stored in the repository.

### 6.4. Quality Gate Validation

During implementation, the Maven tests and SonarQube analysis were validated separately.

The first SonarQube integration attempts encountered configuration and Quality Gate issues.

After correcting the Maven plugin invocation and rerunning the analysis, the GitHub Actions CI/CD workflow completed successfully.

The SonarQube project also received an analysis report.

The repository uses the default SonarQube Cloud Quality Gate rather than a custom paid Quality Gate.

## 7. Advanced Exercise - Split Pipelines

### 7.1. Objective

The final exercise separates Continuous Integration and Continuous Delivery into two independent GitHub Actions workflows.

The requirements are:

- CI runs on `main` and `develop`.
- Docker image publication is restricted to `main`.
- CD starts only after successful CI execution.

### 7.2. Continuous Integration: ci.yml

File:

```text
.github/workflows/ci.yml
```

The CI workflow is triggered by:

```yaml
on:
  push:
    branches:
      - main
      - develop
  pull_request:
    branches:
      - main
      - develop
```

It performs Maven compilation, automated tests, SonarQube analysis, and Quality Gate validation.

### 7.3. Continuous Delivery: cd.yml

File:

```text
.github/workflows/cd.yml
```

The CD workflow uses `workflow_run`:

```yaml
on:
  workflow_run:
    workflows:
      - CI DevOps
    types:
      - completed
    branches:
      - main
```

The publishing job requires a successful CI run associated with a push to the `main` branch.

The workflow also checks the CI conclusion and source event before publishing.

This prevents `develop` pushes or pull request runs from publishing Docker images.

### 7.4. Validated Commit

The CD workflow checks out the commit SHA associated with the successful CI workflow.

This ensures that the published Docker images correspond to the same source revision that passed the CI checks.

### 7.5. Workflow Architecture

```text
             GitHub Push
                  |
          +-------+-------+
          |               |
         main           develop
          |               |
          v               v
       CI DevOps       CI DevOps
          |               |
          v               v
      Maven Tests     Maven Tests
          |               |
          v               v
      SonarQube       SonarQube
      Quality Gate    Quality Gate
          |               |
          v               |
      CI Success          |
          |               |
          v               v
       CD DevOps      No publication
          |
          v
    Build 3 Docker Images
          |
          v
       Docker Hub
```

## 8. Validation Results

### 8.1. Initial CI

The initial GitHub Actions pipeline successfully compiled and tested the Java backend.

**Commit:** `d9eb690`

### 8.2. Continuous Delivery

The CI/CD workflow successfully built and published the three application images to Docker Hub.

**Commit:** `6e4b24f`

The published images were verified using Docker manifest inspection.

### 8.3. SonarQube Integration

SonarQube Cloud analysis was integrated into the GitHub Actions pipeline.

A later successful workflow run confirmed the integrated CI/CD execution.


### 8.4. Split Pipelines

The combined CI/CD workflow was split into two independent workflows:

- `.github/workflows/ci.yml`
- `.github/workflows/cd.yml`

The `main` branch successfully runs Maven tests, SonarQube Cloud analysis and the Quality Gate before triggering Docker Hub publication.

The `develop` branch also successfully runs CI. Due to SonarQube Cloud Free plan restrictions, SonarQube analysis is limited to `main`.

The `develop` CI workflow completed successfully in 44 seconds (commit `586d596`). Docker image publication is restricted to successful CI runs triggered by pushes to `main`.

Both CI and CD were successfully validated on `main`.


### 8.5. GitHub Actions

Workflow history and current execution results are available at:

https://github.com/cz632804-cpu/tp-devops-correction-docker/actions

### 8.6. Docker Hub Verification

Published images can be checked with:

```bash
docker manifest inspect nifdgg/tp-devops-simple-api:latest
docker manifest inspect nifdgg/tp-devops-database:latest
docker manifest inspect nifdgg/tp-devops-http-server:latest
```

These commands verify that the image manifests exist in the remote Docker registry.

## 9. Security Considerations

### Secrets Management

All authentication tokens are stored in GitHub repository secrets.

Passwords and access tokens must never be written directly into workflow YAML files or committed to Git.

### Branch Protection

The final CD workflow restricts Docker image publication to validated pushes on `main`.

The development branch is used for integration checks without publishing images.

### Quality Checks

Maven tests and SonarQube analysis run before Docker images are published.

This reduces the risk of publishing builds that fail automated tests or quality validation.

### Reproducibility

GitHub Actions runs the same automated checks on configured Git events.

The CI and CD workflows are versioned alongside the application source code.

## 10. Conclusion

This project demonstrates how GitHub Actions can automate the software development and delivery process.

The final implementation includes:

- Git repository management and branching.
- Automated Maven builds and tests.
- PostgreSQL integration testing with Testcontainers.
- Docker image construction and publication.
- Secure authentication using GitHub Secrets.
- SonarQube Cloud code quality analysis.
- Quality Gate validation.
- Independent CI and CD workflows.
- Deployment restrictions based on branches and CI results.

The completed CI/CD workflow improves reproducibility, automation, and reliability while reducing repetitive manual operations.
