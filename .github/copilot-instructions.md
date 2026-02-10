# Copilot instructions for this repository

This repo is a minimal static site packaged into a Docker image and built/pushed by a Jenkins pipeline.

Key files
- `Jenkinsfile`: pipeline stages (checkout → set version → build → tag → push). See how `VERSION` is composed as `${BUILD_NUMBER}-${SHORT_SHA}` and `docker` commands run under Windows `bat` steps.
- `Dockerfile`: simple `nginx:alpine` image that copies repository contents into `/usr/share/nginx/html`.

What an AI coding agent should know
- Pipeline runtime: Jenkins uses Windows-style `bat` steps (not `sh`) and expects a Jenkins agent that supports Windows commands.
- Image naming & tagging: environment `IMAGE` is set to `ssm0712/jenkins-production-versioning-practice`. Tags used: `VERSION`, `BUILD_NUMBER`, and `latest`.
- Credentials: `docker push` uses Jenkins credentials with id `dockerhub-creds` (username/password). Do not hardcode secrets in code; reference credentials by id.
- Build context: Docker build runs from repo root. Any change to static assets should be added to the top-level tree to be copied into the image.

Common tasks & reproducible commands
- Build locally (replicates Jenkins build):
  - PowerShell / Windows CMD:
    - `docker build -t ssm0712/jenkins-production-versioning-practice:LOCAL .`
  - Tagging:
    - `docker tag ssm0712/jenkins-production-versioning-practice:LOCAL ssm0712/jenkins-production-versioning-practice:latest`
  - Push (after `docker login`):
    - `docker push ssm0712/jenkins-production-versioning-practice:latest`

Patterns and pitfalls observed
- Versioning: Jenkins composes `VERSION` as `${BUILD_NUMBER}-${SHORT_SHA}` using `git rev-parse --short HEAD`. When simulating locally, compute a short SHA and combine with your CI build number or timestamp.
- Platform assumptions: The pipeline runs Windows `bat` commands; if you change to `sh`, update the `Jenkinsfile` accordingly and ensure the agent environment is Linux-compatible.
- Docker push stage: the pipeline expects `docker login` with credentials from `dockerhub-creds`. Confirm credential id exists in Jenkins before changing push logic.

Where to look next
- To change pipeline behavior: edit `Jenkinsfile` (root).
- To change the built site: edit top-level files (they are copied into the image by the `Dockerfile`).

If you want me to refine this document, tell me what additional workflows to document (e.g., local test commands, branch-release conventions, or how credentials are managed in your Jenkins instance).
