[TOC]

# 09: Developer Workflow

## 9.1 General Workflow

1. Write/update code
2. Run changes locally (for development environment)
3. Create a pull request
4. Run tests via continuous integration
5. Deploy to staging via CD (on merge to main)
6. Deploy to production via CD (on release)

\* **Recommendation**: Having a testing schedule on a cron: periodically running just a `terraform plan` within your continuous integration tool and if that plan shows any changes from the deployed state to the current config on your main branch to raise an error.

## 9.2 Multi Account/Project

Different from cloud-to-cloud terminology, it's something like one account for production and one account for staging environments.

- Simplify IAM policies for enforcing controls for different environments (and remote TF backends)
- Isolate environments to protect minimize blast radius
- Reduce naming conflicts for resources
- Con: Adds complexity to TF config (but still worth it + tooling can help)

[Hashicorp - Going Multi-Account with Terraform](https://www.hashicorp.com/resources/going-multi-account-with-terraform-on-aws)

## 9.3 Additional Tools

### 9.3.1 Terragrunt

- Minimizes code repetition
- Enables multi-account separation (improved isolation/security)

### 9.3.2 Cloud Nuke

- Easy cleanup of cloud resources

### 9.3.3 Makefiles (or shellscripts)

- Prevent human error

## 9.4 Continuous Integration/Continuous Deployment

- GitHub Actions
  - https://github.com/hashicorp/setup-terraform
- CircleCI
  - https://circleci.com/developer/orbs/orb/circleci/terraform
- GitLab
  - https://docs.gitlab.com/user/infrastructure/iac
- Atlantis
  - https://www.runatlantis.io/

## 9.5 Potential Gotchas

These gotchas with Terraform can lead you to have a bad day!

- **Name changes when refactoring**:

  If you change the name of a resource within your Terraform config it can lead Terraform to think, "Oh! they want to delete this resource and create a new one."

- **Sensitive data in Terraform state files**:

  The Terraform state file does have sensitive data within it, so be careful in making sure to encrypt and manage permissions.

- **Cloud timeouts**:

  You can issue a command and sometimes that command will take a long time for the server to provision or for the database to be provisioned, etc., and if there are things that take a long time behind the scenes, Terraform sometimes has timeouts, where it will provision half of your infrastructure and then the other things were taking too long and it gives up, so you can configure those timeouts, but something to think about; usually if you just reissue the `terraform apply` command it will fix that, but they can be a little challenging.

- **Naming conflicts**:

  If you are provisioning two things (two resources) within the same AWS account, the second one will fail to create.

- **Forgetting to destroy test-infra**:

  Use Cloud Nuke tool or write some ShellScript.

- **Uni-directional version upgrades**:

  If I run Terraform version 1.0.0 to provision my infrastructure and then my colleague runs Terraform 1.1.0, I can now no longer issue a command with my older version of Terraform, because the state file is associated with the version of Terraform binary that was used with it. So I would then need to upgrade and so if you have a larger team you want to make sure everyone is using the same version of Terraform on their local system as well as matching the version in your CI/CD system.

- **Multiple ways to accomplish same configuration**:

  That's not necessarily a con but it's just as you're thinking about things, there are always multiple ways to do it. What would be the cleanest representation of this infrastructure is important to think about.

- **Some params are immutable**:

  If I provision some specific resource, there are certain fields within that resource that I could change and there are certain fields that I cannot. If I need to make a change associated with one of those immutable parameters, I will need to delete the previous resource and provision a new one and so that's another consideration in terms of thinking about downtime.

- **Out of band changes**:

  Making changes out of the normal Terraform sequence of events.

  That is something that you just want to avoid whenever possible.

## 9.6 Example

We are creating an automation with GitHub Actions.

In `.github/workflows/terraform.yml`:

```yaml
name: "Terraform"

on:
  push:
    branches:
      - main
  release:
    types: [published]
  pull_request:

jobs:
  terraform:
    name: "Terraform"
    runs-on: ubuntu-latest
    env:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    defaults:
      run:
        working-directory: 07-managing-multiple-environments/file-structure/staging
    steps:
      - name: Checkout
        uses: action/checkout@v2

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_version: 1.0.1
          terraform_wrapper: false

      - name: Terraform Format
        id: fmt
        run: terraform fmt -check

      - name: Terraform Init
        id: init
        run: terraform init

      - name: Terraform Plan
        id: plan
        if: github.event_name == 'pull_request'
        # Route 53 zone must already exist for this to succeed!
        run: terraform plan -var db_pass=${{ secrets.DB_PASS }} -no-color
        continue-on-error: true

      - uses: actions/github-script@0.9.0
        if: github.event_name == 'pull_request'
        env:
          PLAN: "terraform\n${{ steps.plan.outputs.stdout }}"
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const output = `#### Terraform Format and Style 🖌 \`${{ steps.fmt.outcome }}\`
            #### Terraform Initialization  \`${{ steps.init.outcome }}\`
            #### Terraform Plan 📖\`${{ steps.plan.outcome }}\`

            <details><summary>Show Plan</summary>

            \`\`\`${process.env.PLAN}\`\`\`

            </details>

            *Pusher: @${{ github.actor }}, Action: \`${{ github.event_name }}\`*`;


            github.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            })
      - name: Terraform Plan Status
        if: steps.plan.outcome == 'failure'
        run: exit 1

      - uses: actions/setup-go@v2
        with:
          go-version: "^1.15.5"

      - name: Terratest Execution
        if: github.event_name == 'pull_request'
        working-directory: 08-testing/tests/terratest
        run: |
          go test . -v timeout 10m

      - name: Check tag
        id: check-tag
        run: |
          if [[ ${{ github.ref }} =~ ^refs\/tags\/v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then echo ::set-output name=environment::production
          elif [[ ${{ github.ref }} == 'refs/heads/main' ]]; then echo ::set-output name=environment::staging
          else echo ::set-output name=environment::unknown
          fi

      - name: Terraform Apply Global
        if: github.event_name == 'push' || github.event_name == 'release'
        working-directory: 07-managing-multiple-environments/file-structure/global
        run: |
          terraform init
          terraform apply -auto-approve

      - name: Terraform Apply Staging
        if: steps.check-tag.outputs.environment == 'staging' && github.event_name == 'push'
        run: terraform apply -var db_pass=${{secrets.DB_PASS }} -auto-approve

      - name: Terraform Apply Production
        if: steps.check-tag.outputs.environment == 'production' && github.event_name == 'release'
        working-directory: 07-managing-multiple-environments/file-structure/production
        run: |
          terraform init
          terraform apply -var db_pass=${{secrets.DB_PASS }} -auto-approve
```

