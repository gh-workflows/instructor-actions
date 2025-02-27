# Deploy on push to `master` or `main`

In this session, we are going to be deploying the application to various production style environments. This will take the built **Docker** Container and deploy it to one or more of the various environments.

### Exercise: Deploy Docker images

> **Note:** Before you add the code below, you will need to setup **Github Secrets** To help hold credentials in environm.

- **DockerHub**
  - `DOCKERHUB_USERNAME` - Username to authenticate to DockerHub
  - `DOCKERHUB_PASSWORD` - Password to authenticate to DockerHub
  - `DOCKER_ORG`         - Your Docker organization

#### Deploy to DockerHub

1. Create a new branch called `Deploy`
1. Add the following file to your repository: `.github/workflows/deploy-prod-docker.yml`


```yaml
# This is a basic workflow to help you get started with Actions

name: Docker Production

# Controls when the action will run.
on:
  push:
    branches:
      - 'master'
      - 'main'

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  docker-prod-release:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest
    # You could use the following lines to help make sure only X people start the workflow
    # if: github.actor == 'admiralawkbar' || github.actor == 'jwiebalk'

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - name: Checkout source code
        uses: actions/checkout@v2

      #########################
      # Install Docker BuildX #
      #########################
      - name: Install Docker BuildX
        uses: docker/setup-buildx-action@v3

      ######################
      # Login to DockerHub #
      ######################
      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_PASSWORD }}

      # Update deployment API
      - name: start deployment
        uses: bobheadxi/deployments@v1
        id: deployment
        with:
          step: start
          token: ${{ secrets.GITHUB_TOKEN }}
          env: Production

      # Create a GitHub Issue with the info from this build
      - name: Create GitHub Issue
        uses: actions/github-script@v7
        id: create-issue
        with:
          github-token: ${{secrets.GITHUB_TOKEN}}
          script: |
            const createIssue = async () => {
              const issue = await github.rest.issues.create({
                owner: context.repo.owner,
                repo: context.repo.repo,
                title: "Deploying to production",
                body: 'Currently deploying...'
              });
              console.log('create', issue);
              return issue.data.number;
            };
            return createIssue();

      ###########################################
      # Build and Push containers to registries #
      ###########################################
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: |
            DOCKER_ORG/demo-action:latest
            DOCKER_ORG/demo-action:v1

      # Update Deployment API
      - name: update deployment status
        uses: bobheadxi/deployments@v1
        if: always()
        with:
          step: finish
          token: ${{ secrets.GITHUB_TOKEN }}
          status: ${{ job.status }}
          env: ${{ steps.deployment.outputs.env }}
          deployment_id: ${{ steps.deployment.outputs.deployment_id }}

      - name: Update issue success
        uses: actions/github-script@v7
        if: success()
        with:
          github-token: ${{secrets.GITHUB_TOKEN}}
          script: |
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: Number(${{ steps.create-issue.outputs.result }}),
              body: "Successfully deployed to production"
            })

      - name: Update issue success
        uses: actions/github-script@v7
        if: success()
        with:
          github-token: ${{secrets.GITHUB_TOKEN}}
          script: |
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: Number(${{ steps.create-issue.outputs.result }}),
              body: "Failed to deploy to production"
            })

```

- Commit the code
- Open Pull request

---
