# GitHub Copilot License Assignment

This repository contains an Issue Template and a GitHub Actions workflow that automates the assignment of GitHub Copilot licenses to users. The workflow is triggered when an issue is labeled with `copilot-request`.

This template works for **GitHub Copilot for Business** and **GitHub Copilot Enterprise**.

## Objective

The main goal of this repository is to automate the process of assigning GitHub Copilot licenses. Instead of manually assigning licenses, the organization can leverage this solution to simply let the users open an issue using the provided issue template, and the workflow will automatically assign a license to them, when the user opens the issue they are committing to use the GitHub Copilot license.

[![GitHub Copilot Logo](https://upload.wikimedia.org/wikipedia/commons/thumb/8/8a/GitHub_Copilot_logo.svg/1200px-GitHub_Copilot_logo.svg.png)](https://github.com/features/copilot)

## How to configure this repository in your organization

1. Click on [Use this template](https://github.com/new?template_name=GitHubCopilotLicenseAssignment&template_owner=microsoft) to create a new repository in your GitHub organization where you want to automate the GitHub Copilot license assignment.
2. [Create a new label](/../../labels) in that new repository named `copilot-request`.
3. The GitHub Enterprise Owner or GitHub Organization Owner needs to enable GitHub Copilot for the organization where the repository is located. For more details, refer to the [official documentation](https://docs.github.com/copilot/managing-github-copilot-in-your-organization/managing-access-for-copilot-in-your-organization).
4. In the GitHub Organization, create a new GitHub App by going to `Settings`, then `Developer settings` and then `GitHub Apps`, click on `New GitHub App` and provide the `Name`, `Homepage URL` (could be any URL) and the following permissions:
   - `repository:metadata:ready-only`
   - `organization:GitHub Copilot Business:read and write`
5. In the App Settings click on `Install App` in the left menu and select the repository that you created before.   
6. [Store the GitHub App ID in the repository secrets](/../../settings/secrets/actions/new) with the name `APPLICATION_ID`.
7. In the `App Settings`, scroll down to the `Private keys` section and click on `Generate a private key` for the GitHub App and [store the plain text of the .pem file in the repository secrets](/../../settings/secrets/actions/new) with the name `APPLICATION_PRIVATE_KEY`.

> Note: The workflow use the `GITHUB_TOKEN` to comment and close the issue, this token is automatically created by GitHub and is available to use in the workflow, however the `GITHUB_TOKEN` needs to have `Read and write permissions` you can check this in the organization settings, then `Actions`, In the `Workflow permissions` of `General`. Alternatively, you can use the GitHub App that you just created to comment and close the issue but you will need to enable the `Issue` permissions in the `App settings` with the `Read and write` access level and change the line 50 and 78 of the workflow to use the GitHub App token `GITHUB_TOKEN: ${{ steps.get_workflow_token.outputs.token }}` instead of the `GITHUB_TOKEN`. 

## How users can request a GitHub Copilot License

> Note: At this point GitHub Copilot should be enabled in the organization, the label `copilot-request` should be created and the GitHub App configured with the secrets `APPLICATION_ID` and `APPLICATION_PRIVATE_KEY` already stored in the repository.

1. Users can now [open a new issue in this repository using the issue template 'Request GitHub Copilot License'](/../../issues/new?assignees=&labels=copilot-request&projects=&template=github_copilot_request.md&title=Request+GitHub+Copilot+License).
3. The workflow will be triggered automatically and if successful, a GitHub Copilot license will be assigned to the user's account.

## (Optional) Add an approval process to the license assignment

One of the easiest way to add an approval process to the GitHub Copilot license assignment is by following the following steps:

1. Create a new environment in the repository. Go to `Settings`, then `Environments` and click on `New environment`. Provide a name to the environment, for example, `Copilot` or `copilot-request-approval`.
2. In the environment settings, check the `Required reviewers` option and add the users that will be able to approve the requests. (You can add up to 6 users).
3. In the workflow file [assign_github_copilot_license.yml](/.github/workflows/assign_github_copilot_license.yml), add `environment: <name of your environment>`, in the job that assigns the GitHub Copilot (line 8), for example:

```yaml
6. jobs:
7.    assign_github_copilot_license:
8.       environment: Copilot
```

Now, when a user opens an issue with the label `copilot-request`, the workflow will wait for the approval of the users added to the environment before assigning the GitHub Copilot license.

## Repository Content

The repository contains the following files:

### 1. Issue Template

The issue template [Request GitHub Copilot License](/.github/ISSUE_TEMPLATE/github_copilot_request.md) is provided to standardize the license request process. The template uses the label `copilot-request` to trigger the workflow that assigns GitHub Copilot licenses, this label needs to be created in the repository before opening an issue. The template contains a notice to the user about the commitment they're making by requesting the GitHub Copilot license, informing the user about what will happen once their license request is fulfilled, a motivational message for the user, and a thank you note to the user.

### 2. GitHub Action Workflow

The workflow [assign_github_copilot_license.yml](/.github/workflows/assign_github_copilot_license.yml) is triggered when an issue is labeled with `copilot-request`.  The workflow uses the [GitHub API](https://docs.github.com/en/rest/copilot/copilot-user-management?apiVersion=2022-11-28#add-users-to-the-copilot-subscription-for-an-organization) through [octokit](http://octokit.github.io/) to assign the GitHub Copilot license to the user who opened the issue. A token is generated using [Peter Murray's Action](https://github.com/peter-murray/workflow-application-token-action). Upon successful execution, the workflow comments on the issue and closes it. In case of failure, it comments on the issue to notify about the failure and leaves the issue open. The workflow uses the [GITHUB_TOKEN](https://docs.github.com/actions/reference/authentication-in-a-workflow) to comment and close the issue opened by the user. The execution of the workflow takes less than 1 minute.