## **Azure Pipelines Variables FAQ: Compile-Time vs. Runtime Variable Access**

### 1. **What is the difference between `${{ variables.NAME }}` and `$(NAME)` in Azure Pipelines?**

- **`${{ variables.NAME }}`**: This syntax is used for **compile-time** evaluation. Variables referenced with `${{ variables.NAME }}` are evaluated when the pipeline is processed and the YAML is being parsed. Use this syntax for:
  - Conditional expressions.
  - Including or excluding stages, jobs, or steps based on variables.
  - Defining pipeline logic during the initial pipeline setup.

- **`$(NAME)`**: This syntax is used for **runtime** evaluation. Variables referenced with `$(NAME)` are evaluated during the pipeline execution, at the moment the tasks or scripts are run. Use this syntax when:
  - Passing variables into tasks or scripts.
  - Referencing variables from the environment or variable groups during the execution.

### 2. **How do I access variables from a variable group in Azure Pipelines?**

If you're pulling a variable (e.g., `READ_PACKAGES_PAT`) from a **variable group**, you should typically use the **runtime expression**:

- **Syntax**: `$(READ_PACKAGES_PAT)`
  
This allows the variable to be accessed when the pipeline is running, as variable groups are typically resolved at runtime.

**Example**:
```yaml
apiCheckoutExtRepo:
  PAT: $(READ_PACKAGES_PAT)  # Correct syntax for accessing variable from a variable group
```

### 3. **How do I reference variables from a YAML template in Azure Pipelines?**

When you define variables in a separate YAML template (e.g., `../../variables/_envspecific/global.yaml`), those variables are accessible based on the context:

- **Compile-Time (using `${{ variables.NAME }}`)**: If you are referencing the variable in a conditional statement, or to control pipeline structure (such as stages or jobs), use `${{ variables.NAME }}`.

- **Runtime (using `$(NAME)`)**: If you are using the variable in a task, script, or any other step that is executed while the pipeline runs, use `$(NAME)`.

**Example (using template variables)**:

```yaml
# Referencing variables from a template in conditional logic (compile-time)
stages:
  - ${{ if eq(variables.environmentName, 'dev') }}:
    - stage: DeployToDev
      jobs:
        - job: Deploy
          steps:
            - script: echo "Deploying to $(environmentName)"  # Correct syntax for runtime use
```

### 4. **How do I access a Personal Access Token (PAT) securely in Azure Pipelines?**

If you are passing a **Personal Access Token (PAT)**, it is typically stored securely in a **variable group** or a **YAML template**. The correct way to access it depends on whether you're passing it at compile-time or runtime:

- **Compile-Time**: Rarely used for PAT, but can be referenced via `${{ variables.PAT }}` if needed in the pipeline structure.
- **Runtime**: This is the recommended method for passing secrets like PAT tokens. Use `$(PAT)`.

**Example**:
```yaml
apiCheckoutExtRepo:
  PAT: $(READ_PACKAGES_PAT)  # Correct runtime syntax for PAT
```

### 5. **When should I use compile-time vs. runtime variables?**

- **Use Compile-Time (`${{ variables.NAME }}`)** when:
  - You are controlling the structure of the pipeline (e.g., determining whether a stage or job should run based on the value of a variable).
  - You need to include or exclude templates or steps based on variables.
  
**Example**:
```yaml
# Use compile-time variable for conditional stage inclusion
stages:
  - ${{ if eq(variables.deployAPI, true) }}:
    - stage: DeployAPI
      jobs:
        - job: Deploy
```

- **Use Runtime (`$(NAME)`)** when:
  - You need to pass a variable to a task, script, or command that is executed during the pipeline.
  - You are dealing with secrets or sensitive values, such as tokens or passwords, pulled from variable groups.

**Example**:
```yaml
# Use runtime variable for passing secret values like tokens to a task
steps:
  - script: echo "Using PAT: $(READ_PACKAGES_PAT)"
```

### 6. **What happens if I mix `${{ variables.NAME }}` and `$(NAME)` incorrectly?**

- If you use `$(NAME)` in a place where a compile-time variable is expected, the variable will not resolve correctly during pipeline parsing, and the pipeline might fail or behave unexpectedly.
- If you use `${{ variables.NAME }}` in a task that runs at runtime, it will fail to retrieve the correct value, because the variable will not be evaluated at the right time.

### 7. **Can I use variables defined in YAML templates to control pipeline flow?**

Yes! Variables defined in YAML templates can be used to control the pipeline flow, such as including or excluding stages or jobs based on the value of those variables.

**Example**:
```yaml
# Template variable controlling whether a stage is included
stages:
  - ${{ if eq(variables.environmentName, 'production') }}:
    - stage: DeployToProduction
      jobs:
        - job: Deploy
```

### 8. **How do I define variables in YAML templates and use them in the main pipeline?**

You can define variables in a template and reference them in your main pipeline YAML file:

**Template (`global.yaml`)**:
```yaml
variables:
  environmentName: "dev"
  apiKey: "12345"
```

**Main Pipeline YAML**:
```yaml
- template: ../../variables/_envspecific/global.yaml@self

stages:
  - stage: Deploy
    jobs:
      - job: DeployJob
        steps:
          - script: echo "Environment: $(environmentName)"  # Accessing runtime variable
```

### 9. **What if I need to use both compile-time and runtime variables in the same pipeline?**

You can use both compile-time and runtime variables in the same pipeline, but be careful to apply the correct syntax in the appropriate context.

**Example (mixed usage)**:
```yaml
stages:
  - ${{ if eq(variables.environmentName, 'dev') }}:
    - stage: DeployToDev
      jobs:
        - job: Deploy
          steps:
            - script: echo "Environment: $(environmentName)"  # Runtime access
```

---
