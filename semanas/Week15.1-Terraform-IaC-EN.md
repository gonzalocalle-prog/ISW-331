# Week 15.1 - Infrastructure as Code with Terraform

> **Block 5 Extension: Cloud and Production**
> This optional companion extends Week 15 by introducing Infrastructure as Code (IaC) with Terraform. The goal is not to turn students into Terraform specialists. The goal is to help them see how infrastructure can be defined, reviewed, and repeated as code.

---

## Part 1 - Where Terraform Fits

This extension makes the most sense after students already understand the Week 15 deployment flow:

```text
Source code
   -> infrastructure exists
   -> application is deployed
   -> runtime configuration and secrets are added
   -> logs, health, and URL are verified
```

Terraform focuses on the infrastructure part of that chain.

It helps students separate three different concerns:

- Provisioning infrastructure
- Deploying application code
- Managing runtime configuration and secrets

---

## Part 2 - What Infrastructure as Code Means

Infrastructure as Code means defining infrastructure in version-controlled files instead of creating everything manually through a web console.

With Terraform, students can describe resources such as:

- A virtual machine or app hosting service
- Networking rules
- DNS records
- Storage resources
- Managed databases

### Why this matters

Terraform helps teams:

- Recreate environments more consistently
- Review infrastructure changes before applying them
- Reduce manual setup mistakes
- Document infrastructure as part of the project

---

## Part 3 - What Students Should Understand

Week 15.1 does not need deep Terraform expertise. A beginner-level understanding is enough:

- Terraform files describe the desired infrastructure state
- `terraform plan` shows intended changes before they happen
- `terraform apply` creates or updates the infrastructure
- Variables help avoid hardcoding environment-specific values
- State tracks what Terraform already manages

### A simple mental model

```text
Terraform
   -> creates infrastructure
   -> application is deployed into that infrastructure
   -> runtime configuration and secrets are added
   -> app is verified through logs, health checks, and URL access
```

---

## Part 4 - Good Use Cases

Terraform as an extension of deployment thinking, not as a requirement for every student project.

Terraform is a good fit when:

- Students are repeating the same environment setup many times
- The class wants a provider-agnostic way to discuss provisioning
- You want students to see infrastructure reviewed like code

It is probably too much when students are still struggling with their first basic deployment.

---

## Part 5 - Suggested Optional Exercise and Deliverable

### Suggested extension

Use Terraform to provision part of the infrastructure before deployment, such as:

- A hosting service or virtual machine
- A storage resource
- A DNS record
- A managed database

Then deploy the application and verify that the infrastructure and app work together correctly.

### Deliverable expectations

If Terraform is used, students should include:

- The Terraform files in the repository or a linked infrastructure repository
- A short note describing which resources Terraform manages
- A brief note explaining how infrastructure changes were reviewed or applied
- Evidence that secrets are not committed to the repository
- A warning not to commit sensitive state snapshots carelessly

### Reflection questions

1. Which infrastructure decisions were encoded in Terraform instead of being configured manually?
2. What would be easier to reproduce with Terraform in another environment?
3. Which values should stay outside Terraform because they are secrets or environment-specific?
4. What part of the workflow still requires application-level deployment steps after provisioning is done?

---

## Recommended Reading and Exploration

- [Terraform Documentation - Intro to Infrastructure as Code](https://developer.hashicorp.com/terraform/intro) - A practical starting point for understanding Terraform workflows and concepts
- [Terraform Documentation - State](https://developer.hashicorp.com/terraform/language/state) - Useful background for explaining why state matters and why it must be handled carefully
- [The Twelve-Factor App](https://12factor.net/) - Helpful for reinforcing that infrastructure, config, and application code are related but not the same thing
