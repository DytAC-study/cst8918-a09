# CST8918 – A01: Terraform Pre-Commit and GitHub Actions Pipeline

## Project Overview

This project is part of CST8918 – DevOps: Infrastructure as Code. It demonstrates the use of Husky for local pre-commit checks and GitHub Actions for remote CI/CD validation. The goal is to ensure Terraform code quality and correctness using automation.

## Objectives

* Use Husky to enforce local code quality with pre-commit hooks
* Create a GitHub Actions workflow to run `terraform fmt` and `terraform validate`
* Demonstrate Terraform automation best practices

## Directory Structure

```
.
├── .github/workflows
│   └── action-terraform-verify.yml
├── .husky
│   └── pre-commit
├── infrastructure
│   └── storage.tf
├── package.json
├── terraform.lock.hcl
└── README.md
```

## Step-by-Step Setup

### 1. Initialize Terraform Project

```bash
mkdir infrastructure && cd infrastructure
touch storage.tf
terraform init
```

Add a basic `azurerm_storage_account` Terraform script to `storage.tf`.

### 2. Initialize Node Project and Husky

```bash
npm init -y
npm install husky --save-dev
npx husky install
npx husky-init
```

### 3. Setup Pre-Commit Hook

```bash
echo "terraform fmt -check -recursive" > .husky/pre-commit
echo "terraform validate" >> .husky/pre-commit
echo "tflint" >> .husky/pre-commit
chmod +x .husky/pre-commit
```

> Note: Ensure `.husky` is in a Git repository and `git init` has been run.

### 4. Test Pre-Commit Hook

Create a syntax or format error in `storage.tf`, attempt to commit, and verify commit is blocked.

```bash
git add infrastructure/storage.tf
git commit -m "Test: bad formatting"
```

Correct the issue and retry commit:

```bash
git commit -m "Fix: formatting issue"
```

### 5. Create GitHub Actions Workflow

Create `.github/workflows/action-terraform-verify.yml` with the following content:

```yaml
name: Terraform Format and Validate

on:
  pull_request:
    branches:
      - main
      - master

permissions:
  contents: read

jobs:
  terraform:
    name: Terraform Check
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.2.4

      - name: Terraform Format Check
        run: terraform fmt -check -recursive

      - name: Terraform Init
        run: terraform init
        working-directory: ./infrastructure

      - name: Terraform Validate
        run: terraform validate
        working-directory: ./infrastructure
```

### 6. Test CI/CD Workflow

1. Create a feature branch

```bash
git checkout -b test-formatting-error
```

2. Make a formatting error and push the branch

```bash
git add .
git commit -m "Intentional formatting error"
git push --set-upstream origin test-formatting-error
```

3. Create a pull request to `main`. GitHub Actions will fail.
4. Fix the error and push again. GitHub Actions should succeed.

## Final Notes

This project ensures that:

* Developers follow Terraform formatting rules locally with Husky
* Pull requests are validated using GitHub Actions for consistent quality
* Both local and remote checks catch errors early in the CI/CD pipeline

