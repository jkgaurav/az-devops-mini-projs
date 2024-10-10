### **FAQ: Accessing Variables in Azure Pipelines**

---

#### **1. What is the correct way to access a variable in Azure Pipelines?**

- **Runtime expression**: Use `$(VARIABLE_NAME)` when accessing variables during pipeline execution (runtime).
- **Template expression**: Use `${{ variables.VARIABLE_NAME }}` when accessing variables during pipeline template processing (compile-time).

---

#### **2. When should I use `$(VARIABLE_NAME)`?**

Use the `$(VARIABLE_NAME)` syntax when you need to access variables that are evaluated **during the execution** of the pipeline, such as passing a value into a script, task, or template parameter. This is the most common case for using variables.

Example:
```yaml
pool:
  name: $(agentPool)
```

---

#### **3. When should I use `${{ variables.VARIABLE_NAME }}`?**

Use `${{ variables.VARIABLE_NAME }}` when you need to evaluate the variable **at compile-time**. This is often required when making decisions in the pipeline structure itself, such as conditional steps or stages.

Example:
```yaml
${{ if eq(variables.environment, 'production') }}:
  pool: production-pool
```

---

#### **4. How do I reference a variable from a variable group?**

You can reference a variable from a **variable group** the same way you reference any runtime variable. Use the `$(VARIABLE_NAME)` syntax to ensure the variable is evaluated at runtime.

Example:
```yaml
steps:
- script: echo "The secret value is: $(MY_SECRET)"
```

---

#### **5. What’s the difference between using `$(VARIABLE_NAME)` and `${{ variables.VARIABLE_NAME }}` in my YAML pipeline?**

- **`$(VARIABLE_NAME)`**: This is evaluated at runtime, meaning the variable's value is available when the pipeline runs. Use this for variables that are passed into tasks or scripts.
- **`${{ variables.VARIABLE_NAME }}`**: This is evaluated at compile-time when the pipeline YAML is processed, which happens before the pipeline execution begins. Use this for conditional logic or defining structures in your pipeline.

---

#### **6. What happens if I use `$(VARIABLE_NAME)` instead of `${{ variables.VARIABLE_NAME }}` or vice versa?**

- If you mistakenly use `$(VARIABLE_NAME)` in a place where a template expression is expected (such as in conditions), it may not work as expected because it won't be evaluated at the right time.
- If you use `${{ variables.VARIABLE_NAME }}` in a place meant for runtime expressions, the pipeline might fail because it’s trying to access the variable before it's set.

---

#### **7. How do I access a Personal Access Token (PAT) stored in a variable group?**

If you have stored a PAT in a variable group, you should access it using the runtime expression `$(PAT_VARIABLE)` to ensure it’s evaluated during pipeline execution.

Example:
```yaml
parameters:
  PAT: $(MY_PAT) # Correct for accessing a PAT from a variable group
```

---

#### **8. Can I use variables from a variable group in conditions?**

Yes, but use the **`${{ variables.VARIABLE_NAME }}`** syntax to evaluate the variable during the template compilation phase, especially if you are making decisions about pipeline structure.

Example:
```yaml
stages:
  - ${{ if eq(variables.environment, 'production') }}:
    - stage: DeployToProd
```

---

#### **9. How do I pass a variable from a variable group into a template?**

Pass the variable using the `$(VARIABLE_NAME)` runtime syntax. For example, if you need to pass a PAT into a template, use:

```yaml
- template: my-template.yaml
  parameters:
    personalAccessToken: $(READ_PACKAGES_PAT)
```

---

#### **10. Why am I getting "Variable not found" errors in my pipeline?**

This can happen if:
- You are using the wrong syntax (`$(...)` vs `${{ ... }}`).
- The variable has not been defined in the pipeline, or it is not available at the point of execution.
- The variable is stored in a variable group, but the group isn’t linked to the pipeline.

Ensure you are using the correct syntax for the context (runtime vs compile-time), and that variable groups are correctly linked.

---
