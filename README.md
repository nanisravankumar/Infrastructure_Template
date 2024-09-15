# file template/steps/tf_apply.yml


This `pipeline.yaml` file is designed for automating Terraform operations in Azure DevOps. It dynamically handles multiple environments (`dev`, `qa`, and `prod`) and allows the user to execute different Terraform commands (like `apply`, `plan`, etc.) by passing parameters to the pipeline.

## Overview

The pipeline uses two key parameters to determine the environment and the Terraform command to run:
- **Environment (`env`)**: Specifies the environment (dev, qa, or prod) in which the Terraform command should be executed.
- **Terraform Command (`tf_command`)**: Specifies the Terraform command to execute (e.g., `apply`, `plan`, `destroy`).

## Parameters

### 1. **Environment Parameter (`env`)**

The `env` parameter allows the user to select which environment (`dev`, `qa`, `prod`) the Terraform command will target. The environment-specific settings, such as the service connection, will be applied based on this parameter.

```yaml
parameters:
- name: env
  displayName: Environment
```

### 2. **Terraform Command Parameter (`tf_command`)**

The `tf_command` parameter defines the Terraform command to be run. Common commands include `apply`, `plan`, and `destroy`. This allows the pipeline to be flexible and reusable for different Terraform operations.

```yaml
parameters:
- name: tf_command
  displayName: tf_command
```

## Steps

### 1. **Terraform Task**

The main step of the pipeline is executing a Terraform task using the `TerraformTaskV1` Azure DevOps task. This task takes inputs for the provider (`azurerm`), the command to run, command options, and the environment service connection.

```yaml
steps:
  - task: TerraformTaskV1@0
    displayName: 'Apply'
    inputs:
      provider: 'azurerm'
      command: ${{parameters.tf_command}}
      commandOptions: '-input=false -auto-approve -var-file=./env/"${{parameters.env}}.auto.tfvars" '
```

### 2. **Environment-Specific Service Connections**

The pipeline dynamically sets the Azure Resource Manager service connection based on the selected environment using conditionals. Here's how the service connection is assigned for each environment:

- **dev** and **qa** use the `Common-NonProduction-Subscription`.
- **prod** uses the `Ael-prod-svc` subscription.

```yaml
      ${{ if eq(parameters.env, 'dev') }}:
        environmentServiceNameAzureRM: "Common-NonProduction-Subscription"
      ${{ if eq(parameters.env, 'qa') }}:
        environmentServiceNameAzureRM: "Common-NonProduction-Subscription"
      ${{ if eq(parameters.env, 'prod') }}:
        environmentServiceNameAzureRM: "Ael-prod-svc"
```

This conditional logic ensures that the correct Azure subscription is used for the deployment based on the environment.

### 3. **Working Directory**

The `workingDirectory` input specifies where the Terraform code is located. It defaults to `$(System.DefaultWorkingDirectory)`, which is the root of the pipeline workspace.

```yaml
      workingDirectory: '$(System.DefaultWorkingDirectory)'
```

## Usage

To use this pipeline, specify the environment and the Terraform command when running the pipeline. For example, you can use `apply` for the `prod` environment:

```yaml
parameters:
  - env: prod
  - tf_command: apply
```

### Example Commands

- To **plan** the infrastructure for the `dev` environment:
  
  ```yaml
  parameters:
    - env: dev
    - tf_command: plan
  ```

- To **apply** the infrastructure changes for the `qa` environment:
  
  ```yaml
  parameters:
    - env: qa
    - tf_command: apply
  ```

- To **destroy** the infrastructure for the `prod` environment:
  
  ```yaml
  parameters:
    - env: prod
    - tf_command: destroy
  ```
