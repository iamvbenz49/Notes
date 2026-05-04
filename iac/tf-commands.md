# Terraform Basic Commands

## 1. `terraform init`
Initializes a working directory. Run this first before anything else.
- Downloads required providers (AWS, Azure, etc.)
- Sets up the backend (where state is stored)
- Only needs to be re-run when providers or backend config changes

```bash
terraform init
```

---

## 2. `terraform plan`
Shows what Terraform *will do* before actually doing it. A dry run.
- Compares your config with the current state
- Shows resources to be created, modified, or destroyed
- Does not make any real changes

```bash
terraform plan

# Save the plan to a file
terraform plan -out=tfplan
```

---

## 3. `terraform apply`
Actually applies the changes to your infrastructure.
- Prompts for confirmation by default
- Use `-auto-approve` to skip the prompt (common in CI/CD)

```bash
terraform apply

# Apply a saved plan
terraform apply tfplan

# Skip confirmation prompt
terraform apply -auto-approve
```

---

## 4. `terraform destroy`
Destroys all resources managed by the current config.
- Prompts for confirmation
- Use carefully — it tears everything down

```bash
terraform destroy

# Skip confirmation
terraform destroy -auto-approve
```

---

## 5. `terraform fmt`
Formats your `.tf` files to the canonical style. Good to run before committing code.

```bash
terraform fmt

# Format recursively in all subdirectories
terraform fmt -recursive
```

---

## 6. `terraform validate`
Checks your config files for syntax errors and internal consistency. Does not connect to any provider.

```bash
terraform validate
```

---

## 7. `terraform show`
Shows the current state or a saved plan in a human-readable format.

```bash
# Show current state
terraform show

# Show a saved plan
terraform show tfplan
```

---

## 8. `terraform state`
Interact with the Terraform state file directly. Useful for debugging or manual fixes.

```bash
# List all resources in state
terraform state list

# Show details of a specific resource
terraform state show aws_instance.my_server

# Remove a resource from state (without destroying it)
terraform state rm aws_instance.my_server
```

---

## 9. `terraform output`
Displays output values defined in your config.

```bash
terraform output

# Get a specific output
terraform output instance_ip
```

---

## 10. `terraform import`
Import existing infrastructure into Terraform state (for resources created outside Terraform).

```bash
terraform import aws_instance.my_server i-1234567890abcdef0
```

---

## Typical Workflow

```bash
terraform init      # 1. initialize
terraform fmt       # 2. format code
terraform validate  # 3. check for errors
terraform plan      # 4. preview changes
terraform apply     # 5. apply changes
```

---

## Key Files

| File | Purpose |
|---|---|
| `main.tf` | Main config — resources defined here |
| `variables.tf` | Input variable declarations |
| `outputs.tf` | Output value declarations |
| `terraform.tfvars` | Variable values |
| `terraform.tfstate` | State file — tracks real infrastructure |
| `.terraform/` | Provider plugins downloaded by init |

> **Note:** The state file (`terraform.tfstate`) is critical — it's how Terraform knows what it manages. In teams, always store it remotely (e.g., S3 + DynamoDB for locking) rather than locally.
