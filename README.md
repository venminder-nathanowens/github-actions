# GitHub Actions

## Domain 1: Author and maintain workflows

### Work with events that trigger workflows:

- Configure workflows to run for one or more events

  ```yaml
  name: Event Trigger Workflow
  on: [push, pull_request]
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2
        - name: Run a one-line script
          run: echo "Hello, world!"
  ```

- Configure workflows to run for scheduled events

  ```yaml
  on:
    schedule:
      - cron: "0 0 * * *" # Runs at midnight every day
  ```

- Configure workflows to run for manual events

  ```yaml
  on:
    workflow_dispatch:
  ```

- Configure workflows to run for webhook events (i.e. check_run, check_suite, deployment, etc.)

  ```yaml
  on:
    deployment:
    check_run:
      types: [created, rerequested]
  ```

- Demonstrate a GitHub event to trigger a workflow based on a practical use case

  ```yaml
  on:
    pull_request:
      types: [synchronize]
  jobs:
    test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2
        - name: Run tests
          run: make test
  ```

### Use the components of a workflow:

- Identify the correct syntax for workflow jobs (i.e. indentation and encapsulation of parts of the workflow)

  - Workflows are written in YAML. Proper indentation and structuring are crucial. Jobs are encapsulated within the jobs key, each job can contain multiple steps.

- Use job steps for actions and shell commands

  - Actions are reusable units in workflows, specified with uses. Shell commands are run with 'run'.

- Use conditional keywords for steps

  ```yaml
  steps:
    - name: Checkout code
      uses: actions/checkout@v2
    - name: Run script if not a fork
      if: github.repository_owner == 'your_username'
      run: echo "Running on main repo!"
  ```

- Describe how actions, workflows, jobs, steps, runs, and the marketplace work together

  - **Actions** are individual tasks that you can combine to create jobs and customize your workflows. Actions can be reused across multiple workflows.
  - **Workflows** are automated procedures that are defined by a YAML file in your repository. Workflows are made up of one or more jobs.
  - **Jobs** are a set of steps that execute on the same runner. By default, jobs run in parallel unless specified otherwise.
  - **Steps** are individual tasks that can run commands or actions. A step can either run a script or an action.
  - **Runs** refer to each instance a workflow is executed, which happens automatically upon a triggering event or can be manually triggered.
  - **Marketplace** is where you can find and share actions to use in your workflows, which can be community-built or officially maintained by GitHub.

- Identify scenarios suited for using GitHub- hosted and self- hosted runners

  #### GitHub-Hosted Runners:

  - Suitable for projects with standard development environments.
  - Offers a range of environments such as Ubuntu, Windows, and macOS.
  - Best for projects that do not require heavy customization of the operating system.
  - Use when security and isolation requirements are moderate since each job runs in a fresh instance.

  #### Self-Hosted Runners:

  - Ideal for projects requiring a customized or a specific toolset not provided by GitHub-hosted runners.
  - Necessary when workflows require access to private, internal networks.
  - Better for long-running jobs as they do not have the same time restrictions as GitHub-hosted runners.
  - Use when strict security and compliance policies are in place.

- Implement workflow commands as a run step to communicate with the runner

  ```yaml
  steps:
    - name: Set environment variable
      run: |
      echo "ACTION_STATE=active" >> $GITHUB_ENV
    - name: Use the environment variable
      run: |
      echo "The action state is $ACTION_STATE."
  ```

- Demonstrate the use of dependent jobs
  ```yaml
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2
        - run: make build
    deploy:
      needs: build
      runs-on: ubuntu-latest
      steps:
        - run: make deploy
  ```

### Use encrypted secrets and environment variables as part of a workflow:

- Use encrypted secrets to store sensitive information

  - Set up an encrypted secret: Go to your GitHub repository settings, click on 'Secrets', then 'New repository secret'. Enter the name and value of your secret, such as DEPLOY_KEY.

  ```yaml
  jobs:
    deploy:
      runs-on: ubuntu-latest
      steps:
        - name: Checkout
          uses: actions/checkout@v2
        - name: Use secret
          run: echo "Deploy key is ${{ secrets.DEPLOY_KEY }}"
  ```

- Identify the available default environment variables during the construction of the workflow

  - GitHub provides a set of default environment variables such as GITHUB_SHA, GITHUB_REPOSITORY, and GITHUB_ACTOR. These can be used directly in your workflow scripts.

- Identify the location to set custom environment variables in a workflow

  ```yaml
  jobs:
    build:
      runs-on: ubuntu-latest
      env:
        CUSTOM_VAR: "custom value"
      steps:
        - name: Use custom environment variable
          run: echo $CUSTOM_VAR
  ```

- Identify when to use the GITHUB_TOKEN secret

  - Use GITHUB_TOKEN for operations interacting with GitHub API, like committing code, commenting on issues, or deploying to GitHub Pages.

- Demonstrate how to use workflow commands to set environment variables

  ```yaml
  steps:
    - name: Set the environment variable
      run: echo "ACTION_VAR=some_value" >> $GITHUB_ENV
  ```

### Create a workflow for a particular purpose:

- Add a script to a workflow

  ```yaml
  steps:
    - uses: actions/checkout@v2
    - name: Run script
      run: bash scripts/my_script.sh
  ```

- Demonstrate how to publish to GitHub Packages using a workflow

  ```yaml
  steps:
    # Set up authentication
    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: "14"
        registry-url: "https://npm.pkg.github.com"
    # Publish the package
    - run: npm publish
      env:
        NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  ```

- Demonstrate how to publish to GitHub Container Registry using a workflow

  ```yaml
  steps:
    # Log in to GitHub Container Registry:
    - name: Login to GitHub Container Registry
      uses: docker/login-action@v2
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    # Build and push a container image:
    - name: Build and push
      run: |
        docker build -t ghcr.io/${{ github.repository }}/my-image:latest .
        docker push ghcr.io/${{ github.repository }}/my-image:latest
  ```

- Use database and service containers in a GitHub Actions workflow

  ```yaml
  services:
    postgres:
      image: postgres
      env:
        POSTGRES_PASSWORD: postgres
      ports:
        - 5432:5432
  ```

- Use labels to route workflows to specific runners

  ```yaml
  runs-on: [self-hosted, linux, x64, my-label]
  ```

- Use CodeQL as a step in a workflow

  ```yaml
  steps:
    - name: Checkout repository
      uses: actions/checkout@v2
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v1
    - name: Analyze
      uses: github/codeql-action/analyze@v1
  ```

- Demonstrate how to publish a component as a GitHub release using GitHub Actions

  ```yaml
  name: Create Release
  on:
    push:
      tags:
        - "v*"
  jobs:
    release:
      runs-on: ubuntu-latest
      steps:
        - name: Checkout code
          uses: actions/checkout@v2
        - name: Create Release
          id: create_release
          uses: actions/create-release@v1
          env:
            GITHUB_TOKEN: ${{
  ```

- Deploy a release to a cloud provider using a GitHub Actions workflow

  ```yaml
  name: Deploy to Amazon ECS

  on:
    push:
      branches:
        - main

  env:
    AWS_REGION: MY_AWS_REGION # set this to your preferred AWS region, e.g. us-west-1
    ECR_REPOSITORY: MY_ECR_REPOSITORY # set this to your Amazon ECR repository name
    ECS_SERVICE: MY_ECS_SERVICE # set this to your Amazon ECS service name
    ECS_CLUSTER: MY_ECS_CLUSTER # set this to your Amazon ECS cluster name
    ECS_TASK_DEFINITION:
      MY_ECS_TASK_DEFINITION # set this to the path to your Amazon ECS task definition
      # file, e.g. .aws/task-definition.json
    CONTAINER_NAME:
      MY_CONTAINER_NAME # set this to the name of the container in the
      # containerDefinitions section of your task definition

  jobs:
    deploy:
      name: Deploy
      runs-on: ubuntu-latest
      environment: production

      steps:
        - name: Checkout
          uses: actions/checkout@v4
        - name: Configure AWS credentials
          uses: aws-actions/configure-aws-credentials@0e613a0980cbf65ed5b322eb7a1e075d28913a83
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: ${{ env.AWS_REGION }}
        - name: Login to Amazon ECR
          id: login-ecr
          uses: aws-actions/amazon-ecr-login@62f4f872db3836360b72999f4b87f1ff13310f3a
        - name: Build, tag, and push image to Amazon ECR
          id: build-image
          env:
            ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
            IMAGE_TAG: ${{ github.sha }}
          run: |
            # Build a docker container and
            # push it to ECR so that it can
            # be deployed to ECS.
            docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
            docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
            echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT
        - name: Fill in the new image ID in the Amazon ECS task definition
          id: task-def
          uses: aws-actions/amazon-ecs-render-task-definition@c804dfbdd57f713b6c079302a4c01db7017a36fc
          with:
            task-definition: ${{ env.ECS_TASK_DEFINITION }}
            container-name: ${{ env.CONTAINER_NAME }}
            image: ${{ steps.build-image.outputs.image }}
        - name: Deploy Amazon ECS task definition
          uses: aws-actions/amazon-ecs-deploy-task-definition@df9643053eda01f169e64a0e60233aacca83799a
          with:
            task-definition: ${{ steps.task-def.outputs.task-definition }}
            service: ${{ env.ECS_SERVICE }}
            cluster: ${{ env.ECS_CLUSTER }}
            wait-for-service-stability: true
  ```

### Manage workflow runs:

- Configure caching of workflow dependencies

  ```yaml
  name: Cache Node Modules
  on: push
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2
        - name: Cache node modules
          uses: actions/cache@v1
          with:
            path: ~/.npm
            key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
            restore-keys: |
              ${{ runner.os }}-node-
  ```

- Identify steps to pass data between jobs in a workflow

  ```yaml
  name: Pass Artifacts Between Jobs
  on: push
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - name: Generate artifact
          run: echo "data" > artifact.txt
        - name: Upload artifact
          uses: actions/upload-artifact@v2
          with:
            name: my-artifact
            path: artifact.txt
    use:
      needs: build
      runs-on: ubuntu-latest
      steps:
        - name: Download artifact
          uses: actions/download-artifact@v2
          with:
            name: my-artifact
        - name: Use artifact
          run: cat artifact.txt
  ```

- Remove workflow artifacts from GitHub

  - Currently, artifacts must be deleted manually via GitHub UI or using the GitHub API.

- Add a workflow status badge

  ```md
  [![Example Workflow](https://github.com/github-user/repository/actions/workflows/workflow.yaml/badge.svg)](https://github.com/github-user/repository/actions/workflows/workflow.yaml/badge.svg)
  ```

- Add environment protections

  - Configure environment-specific rules and approvals under repository settings on GitHub.

- Define a matrix of different job configurations

  ```yaml
  name: Strategy Matrix
  on: push
  jobs:
    build:
      runs-on: ${{ matrix.os }}
      strategy:
        matrix:
          os: [ubuntu-latest, windows-latest, macos-latest]
          node-version: [12, 14, 16]
      steps:
        - uses: actions/checkout@v2
        - name: Setup Node.js
          uses: actions/setup-node@v1
          with:
            node-version: ${{ matrix.node-version }}
  ```

- Implement workflow approval gates

  ```yaml
  name: Deployment with Approval
  on: push
  jobs:
    deploy:
      runs-on: ubuntu-latest
      environment:
        name: production
        url: ${{ steps.deploy.outputs.url }}
      steps:
        - name: Deploy
          id: deploy
          run: echo "Deploying to production"
  ```

## Domain 2: Consume workflows

### Interpret the effects of a workflow:

- Identify the event that triggered a workflow from its effects in a repository, issue, or pull request

  - Look at the recent commits, tags, or pull request activities; each of these events can trigger workflows. The GitHub UI shows which event triggered the workflow under the "Actions" tab of a repository.

- Describe a workflow’s effects from reading its configuration file

  - Review the on: section of the YAML configuration to understand what events trigger the workflow. The jobs and steps will detail what actions are taken, such as running tests, deploying code, or tagging a release.

- Diagnose a failed workflow run (i.e. using a workflow run history and its logs, determine why a workflow run may have
  failed)

  - Go to the "Actions" tab of your repository, select the failed run, and inspect the logs for each step. Common causes include failed tests, syntax errors, missing environment variables, or unavailable external resources.

- Identify ways to access the workflow logs from the user interface

  - Navigate to the "Actions" tab, select a specific workflow run, and click on the job to view the logs.

- Identify ways to access the workflow logs from GitHub’s REST API

  - Use the following API endpoint to retrieve logs:

  ```
  # Replace {owner}, {repo}, and {run_id} with the appropriate values.
  GET /repos/{owner}/{repo}/actions/runs/{run_id}/logs
  ```

- Enable step debug logging in a workflow

  - Add the following secrets to your repository: ACTIONS_STEP_DEBUG and ACTIONS_RUNNER_DEBUG both set to true. This will enable more detailed logging during workflow execution.

- Demonstrate how to use default environment variables in a workflow

  ```yaml
  steps:
    - name: Print repository name
      run: echo "This repository is $GITHUB_REPOSITORY"
  ```

- Demonstrate the correct syntax for passing custom environment variables in a workflow step

  ```yaml
  steps:
    - name: Set and use environment variable
      run: |
        echo "CUSTOM_VAR=Hello World" >> $GITHUB_ENV
        echo "Custom variable is $CUSTOM_VAR"
  ```

### Locate a workflow, its logs, and artifacts:

- Describe where to locate a workflow in a repository

  - Workflows are stored in the .github/workflows directory of the repository.

- Explain the difference between disabling and deleting of workflows

  - Disabling a workflow stops it from being triggered but retains the workflow file in the repository. This is reversible.
  - Deleting a workflow removes the workflow file from the repository, permanently stopping its operation unless it is restored from version control history.

- Demonstrate how to download workflow artifacts from the user interface

  - Navigate to the "Actions" tab, select the specific run, scroll down to the "Artifacts" section, and click on the desired artifact to download.

- Describe how to use an organization’s templated workflow

  - If your organization has templated workflows, these can be found under .github/workflows in the organization's .github repository. To use a template, copy it into your own repository's .github/workflows directory and customize as necessary.

### Use actions in a workflow:

- Define the indicators of what makes a trustworthy action

  - Trustworthy actions are usually from verified creators, have a large number of stars on GitHub, regular updates, thorough documentation, and positive community feedback.

- Identify an action’s type, inputs, and outputs

  - Action type can be Docker or JavaScript. Inputs are specified under with: in the workflow file, and outputs can be used from one step in another by referencing them with ${{ steps.<step_id>.outputs.<output_name> }}.

- Demonstrate how to use the specific version of an action in a workflow

  ```yaml
  uses: actions/checkout@v2  # Using a version tag
  uses: actions/checkout@ab1cd234e # Using a commit SHA
  uses: actions/checkout@main  # Using a branch name
  ```

## Domain 3: Author and maintain actions

### Use available action types:

- Identify the type of action required for a given problem (i.e. JavaScript, Docker container, run step)

  - **JavaScript Actions:** Best for when you need a fast execution time and do not require a specific operating system or environment. Ideal for manipulating workflow data, making REST API calls, and handling workflow logic.
  - **Docker Container Actions:** Suitable for tasks that require a specific operating system environment or when you need to package and run a specific software in a containerized environment.
  - **Composite Actions:** Use for executing shell commands directly in the runner's environment without wrapping them in an action.

- Demonstrate how to troubleshoot JavaScript actions

  - Log Output: Add console.log() statements to help trace the execution and understand variable states.
  - Error Handling: Use try-catch blocks to handle errors gracefully and provide meaningful error messages.
  - Local Testing: Test your JavaScript code locally by running it in a Node.js environment to catch errors before deployment.

  #### Example:

  ```js
  const core = require("@actions/core");

  try {
    const input = core.getInput("example-input");
    console.log(`Received input: ${input}`);
    // Process input
    if (!input) {
      throw new Error("Input is required");
    }
  } catch (error) {
    core.setFailed(`Action failed with error: ${error.message}`);
  }
  ```

- Demonstrate how to troubleshoot Docker container actions

  - Build Locally: Build the Docker image locally to ensure there are no issues with the Dockerfile.
  - Run Container Locally: Test the Docker container locally to simulate the action's environment and ensure it behaves as expected.
  - Logs and Debugging: Use docker logs to check for runtime errors and add debugging instructions in your entry script.

  #### Example

  ```docker
  FROM alpine:3.12
  COPY entrypoint.sh /entrypoint.sh
  RUN chmod +x /entrypoint.sh
  CMD ["/entrypoint.sh"]
  ```

  ##### entrypoint.sh

  ```shell
  #!/bin/sh
  echo "Starting action..."
  # Insert commands here
  echo "Action completed."
  ```

### Describe the components of an action

- Identify the files and directory structure needed to create an action

  - **action.yml:** The metadata file that defines inputs, outputs, and main entry point for the action.
  - **Dockerfile** (for Docker actions) or **.js** files (for JavaScript actions): These files contain the executable code for the action.
  - **README.md:** Documentation for users of the action.

  #### Directory structure example:

  ```bash
  /my-action
    |- action.yml
    |- Dockerfile or index.js
    |- README.md
  ```

- Identify the metadata and syntax needed to create an action

  ```yaml
  name: "My Custom Action"
  description: "Performs a custom task"
  inputs:
    example-input:
      description: "Input needed for the action"
      required: true
  outputs:
    example-output:
      description: "Output from the action"
  runs:
    using: "docker"
    image: "Dockerfile"
  ```

- Implement workflow commands within an action to communicate with the runner (Note: this includes exit codes)

### Distribute an action:

- Identify how to select an appropriate distribution model for an action (i.e., public, private, marketplace)

  - **Public:** For actions you want to share with the community. Publish to GitHub Marketplace or a public GitHub repository.
  - **Private:** For internal use within your organization. Store in a private repository that only team members can access.

- Identify the best practices for distributing custom actions

  - Versioning: Use semantic versioning (e.g., v1.0.0) to manage releases of your action.
  - Documentation: Provide clear and detailed usage instructions, including examples and configuration options.
  - Security: Regularly update dependencies to minimize vulnerabilities.

- Demonstrate how to create a release strategy for an action (i.e. versioning)

  - Tag your GitHub repository with a version number using Git tags (e.g., git tag -a v1.0.0 -m "Release version 1.0.0").
  - Push the tags to GitHub (e.g., git push origin v1.0.0).

- Demonstrate how to publish an action to the GitHub marketplace

  - Ensure your repository is public and contains a valid action.yml file.
  - Navigate to the 'Actions' tab in your GitHub repository settings and choose "Publish this action to the GitHub Marketplace."
  - Follow the prompts to set up the action listing, including categorization, pricing (if applicable), and support links

## Domain 4: Manage GitHub Actions in the enterprise

### Distribute actions and workflows to the enterprise:

- Explain reuse templates for actions and workflows

  - Templates can be created by defining actions and workflows in a dedicated repository within your organization. These templates serve as blueprints that can be reused across multiple projects to ensure consistency and speed up workflow setup.

- Define an approach for managing and leveraging reusable components (i.e. repos for storage, naming conventions for files/folders, and plans for ongoing maintenance)

  - **Repositories for Storage:** Use dedicated repositories to store common actions and workflows. This centralizes resources and simplifies updates.
  - **Naming Conventions:** Adopt a consistent naming convention for files and folders to help users easily identify the purpose of each action or workflow. For example, prefix action files with action- and workflows with workflow-.
  - **Ongoing Maintenance:** Plan for regular updates and maintenance of reusable components. Use version tagging to manage releases of actions and track changes over time.

- Define how to distribute actions for an enterprise

  - Actions can be distributed by publishing them to a private organization repository accessible only to the enterprise or by using internal marketplaces if available.

- Define how to control access to actions within the enterprise

  - Use GitHub's built-in access controls to manage who can view and edit the actions. Actions stored in private repositories will be accessible only to members of the organization with the appropriate permissions.

- Configure organizational use policies for GitHub Actions

  - Establish policies on how GitHub Actions should be used within the organization. This includes guidelines on security practices, such as how secrets are handled, and performance considerations, like rate limits on API calls and usage of self-hosted vs. GitHub-hosted runners.

### Manage runners for the enterprise:

- Describe the effects of configuring IP allow lists on GitHub- hosted and self- hosted runners

  - IP allow lists can be used to control which IP addresses are permitted to interact with GitHub-hosted and self-hosted runners. For GitHub-hosted runners, this restricts which actions can run, potentially impacting workflows that interact with external services. For self-hosted runners, this adds a layer of security by restricting access to the runners.

- Describe how to select appropriate runners to support workloads (i.e. using a self- hosted versus GitHub- hosted runner, choosing supported operating systems)

  - **Self-hosted vs. GitHub-hosted Runners:** Choose self-hosted runners for jobs requiring specific hardware, software, or networks not available on GitHub-hosted runners. Use GitHub-hosted runners for jobs that are compatible with standard environments to benefit from GitHub's infrastructure.
  - **Operating Systems:** Select runners that support the operating systems required by your workflows, ensuring compatibility and performance.

- Explain the difference between GitHub- hosted and self- hosted runners

  - GitHub-hosted runners are managed by GitHub and offer a variety of environments. They are easy to set up and scale but have usage limits. Self-hosted runners are managed by your organization, offer full control over the environment, and are suitable for workflows requiring specific configurations.

- Configure self- hosted runners for enterprise use (i.e. including proxies, labels, networking)

  - Include network configurations such as proxies to ensure runners can communicate with the GitHub API and other necessary services. Label runners to specify job routing.

- Demonstrate how to manage self- hosted runners using groups (i.e. managing access, moving runners into and between
  groups)

  - Runners can be organized into groups within an enterprise to manage permissions and segregate jobs by department or project type. Runners can be moved between groups through the GitHub UI or via API.

- Demonstrate how to monitor, troubleshoot, and update self- hosted runners

  - Use tools such as monitoring agents to keep track of the runners’ health and performance. Regularly update the runner software to ensure compatibility with GitHub Actions features. Review logs for troubleshooting any issues that arise.

### Manage encrypted secrets in the enterprise:

- Identify the scope of encrypted secrets

  - Secrets can be scoped at the repository, environment, or organization level, providing flexibility in how sensitive data is managed and accessed.

- Demonstrate how to access encrypted secrets within actions and workflows

  - Use the ${{ secrets.NAME }} syntax to access secrets in your workflows, ensuring sensitive information is not hard-coded.

- Explain how to manage organization- level encrypted secrets

  - Organization-level secrets allow you to share configuration across multiple repositories without duplicating secrets. Manage these via the organization's settings page on GitHub.

- Explain how to manage repository- level encrypted secrets

  - Repository-level secrets are configured within the specific repository settings and are only available to actions running within that repository.
