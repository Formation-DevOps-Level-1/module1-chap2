# Learning Module – GitHub Actions (CI/CD)

This repository is designed as a learning project to build GitHub Actions pipelines for a Spring Boot (Maven) application.

## Learning goals

- Understand the difference between CI, PR validation, and CD.
- Trigger workflows automatically and manually.
- Use Maven cache to speed up builds.
- Share artifacts between jobs.
- Use environment variables and secrets.

## Workflow structure

- [.github/workflows/ci.yml](.github/workflows/ci.yml): Build + Test
- [.github/workflows/pull_request.yml](.github/workflows/pull_request.yml): PR validation (labels + assignment)
- [.github/workflows/cd.yml](.github/workflows/cd.yml): Deployment (manual / push)

---

## 1) CI – Build and Test

File: [.github/workflows/ci.yml](.github/workflows/ci.yml)

### Triggers

- `push` on `main`
- `pull_request_review` with type `submitted` (as currently configured)

### Jobs

1. **build**
	- Checkout source code
	- Maven cache (`~/.m2/repository`) with key:
	  - `${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}`
	- Maven build: `./mvnw clean package -DskipTests`
	- Upload `target/` as artifact (`maven-target`)

2. **test**
	- Depends on `build` (`needs: build`)
	- Downloads `maven-target` artifact into `target/`
	- Runs tests: `./mvnw test`

### Concepts covered

- Dependency caching
- Multi-job pipeline design
- Artifact sharing across jobs

---

## 2) PR Validation – Labels and Assignment

File: [.github/workflows/pull_request.yml](.github/workflows/pull_request.yml)

### Triggers

- `pull_request_review` (`submitted`)
- `pull_request` (`opened`)

### Permissions

- `pull-requests: write` to add labels and assign users.

### Jobs

1. **add-label**
	- Adds `reviewed` label using `actions-ecosystem/action-add-labels@v1`
	- Attempts to add `approved` label using `ableco/label-when-approved-action@main`

2. **assign-creator**
	- Automatically assigns the PR creator using `actions/github-script@v7`

### Concepts covered

- PR and review event handling
- `GITHUB_TOKEN` usage
- Required minimum permissions

---

## 3) CD – Environment-based Deployment

File: [.github/workflows/cd.yml](.github/workflows/cd.yml)

### Triggers

- `push` on `main`
- `workflow_dispatch` with environment choice (`target_env`)

### Job

- `deploy`
  - `environment: ${{ inputs.target_env }}`
  - Reads `MESSAGE` from variables: `${{ vars.MESSAGE }}`
  - Prints `MESSAGE` (deployment simulation)

### Concepts covered

- Manual workflow execution
- Environment selection
- GitHub environment variables (`vars`)

---

## Variables and secrets

### Where to define them

- **Environment variables**: `Settings > Environments > <env> > Variables`
- **Environment secrets**: `Settings > Environments > <env> > Secrets`
- **Repository secrets**: `Settings > Secrets and variables > Actions`

### Usage in workflows

- Variable: `${{ vars.MY_VARIABLE }}`
- Secret: `${{ secrets.MY_SECRET }}`

Example:

```yaml
env:
  MESSAGE: ${{ vars.MESSAGE }}
  API_TOKEN: ${{ secrets.API_TOKEN }}
```

---

## Running the workflows

### CI

- Push to `main`.

### PR validation

- Open a PR (triggers `pull_request`).
- Submit a review (triggers `pull_request_review`).

### CD

- Go to **Actions** > **Continuous Deployment** > **Run workflow**.
- Choose `target_env`, then run.

---

## Current caveats

- In `cd.yml`, the option is `stagging` (spelling); make sure an environment with that exact name exists in GitHub.
- In `cd.yml`, `environment: ${{ inputs.target_env }}` works for `workflow_dispatch`; on `push`, `inputs.target_env` may be empty.
- In `pull_request.yml`, `if` conditions must match the triggering event to avoid referencing missing event fields.

---

## Expected outcome

By the end of this module, learners should be able to:

- build a CI pipeline with cache + artifacts,
- automate PR actions (labels/assignment),
- drive deployment by environment,
- correctly manage GitHub Actions `vars`, `secrets`, and permissions.