### Azure Pipelines Variable and Template Handling – FAQ

#### 1. **How do I reference a variable coming from a variable group or YAML file in Azure Pipelines?**
   - **Runtime expression**: If you are accessing the variable during pipeline execution (like passing it to tasks, scripts, or templates), use the `$(VARIABLE_NAME)` syntax.
   - **Compile-time expression**: If you are referencing the variable during the YAML parsing phase (like in conditional expressions, stage inclusion, etc.), use `${{ variables.VARIABLE_NAME }}`.

#### 2. **What’s the difference between `$(VARIABLE)` and `${{ variables.VARIABLE }}` in Azure Pipelines?**
   - **`$(VARIABLE)`**: This is a **runtime expression**. It is used when the variable needs to be evaluated while the pipeline is running. Commonly used for passing variables to scripts, tasks, or other templates at execution time.
   - **`${{ variables.VARIABLE }}`**: This is a **compile-time expression**. It is used when the variable is evaluated before the pipeline starts executing. This is typically used for conditional logic or pipeline structure decisions.

#### 3. **If a variable is defined in an external YAML file (like `global.yaml`), which syntax should I use to access it?**
   - If the variable is being passed to another template or task during execution, use the **runtime syntax** `$(VARIABLE_NAME)`.
   - If the variable is used to determine pipeline structure or stages (during the setup phase), use `${{ variables.VARIABLE_NAME }}`.

#### 4. **How do I pass a variable from a YAML file as a parameter to another template?**
   - Use the **runtime expression** `$(VARIABLE_NAME)` to pass the variable as a parameter to another template:
   ```yaml
   steps:
     - template: another-template.yaml
       parameters:
         PAT: $(READ_PACKAGES_PAT)  # Runtime variable passed as parameter
   ```

#### 5. **How do I access a parameter passed to a template inside the template itself?**
   - Use `${{ parameters.PARAMETER_NAME }}` to reference the passed parameter within the template:
   ```yaml
   parameters:
     PAT: ''

   steps:
     - script: echo "Using PAT: ${{ parameters.PAT }}"  # Accessing the parameter inside the template
   ```

#### 6. **When should I use `$(VARIABLE)` vs. `${{ variables.VARIABLE }}` when referencing variables in conditions?**
   - **Use `${{ variables.VARIABLE }}`** in conditional expressions that are evaluated at compile-time (during pipeline preparation). Example:
     ```yaml
     ${{ if eq(variables.environment, 'prod') }}:
       - script: echo "Deploying to production"
     ```
   - **Use `$(VARIABLE)`** if the condition depends on a value determined at runtime (during pipeline execution). Typically, conditions like this are less common.

#### 7. **What happens if I use the wrong syntax (e.g., `$(VARIABLE)` in a compile-time expression)?**
   - If you use `$(VARIABLE)` in a compile-time context (like inside `if` conditions), the pipeline will likely fail or the condition will not evaluate correctly, as `$(...)` is meant to be evaluated during runtime.
   - Similarly, using `${{ variables.VARIABLE }}` in a runtime context will result in errors, as compile-time variables are not evaluated during execution.

#### 8. **Can I pass variables from a variable group or template to a job, task, or another template?**
   - Yes, you can pass variables from a variable group or YAML file as parameters to another template or task using `$(VARIABLE)` syntax.
   - For example, to pass a variable from a YAML template to a job or task:
     ```yaml
     jobs:
       - job: ExampleJob
         steps:
           - template: example-template.yaml
             parameters:
               variableFromFile: $(VAR_FROM_FILE)
     ```

#### 9. **What should I use if I want to set a value conditionally at compile-time?**
   - Use `${{ if }}` and compile-time expressions for setting variables, including or excluding stages, or defining parameters that depend on conditional logic:
     ```yaml
     parameters:
       apiName: ${{ if eq(variables.environment, 'prod') }}: 'productionAPI' 
                 ${{ else }}: 'devAPI'
     ```

#### 10. **Can I define variables in a template (YAML file) and use them in different stages or jobs?**
   - Yes, you can define variables in a YAML template and use them across the pipeline. The syntax to access them depends on where you are using them:
     - For tasks or steps, use `$(VARIABLE_NAME)`.
     - For compile-time conditions, use `${{ variables.VARIABLE_NAME }}`.

---
