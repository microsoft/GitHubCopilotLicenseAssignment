# GitHub Copilot License Management

This repository contains **Issue Templates** and a **GitHub Actions Workflow** that automates the management of GitHub Copilot licenses to users. The workflow is triggered when an issue is labeled with `copilot-request` or `copilot-remove`.

This template works for **GitHub Copilot for Business** and **GitHub Copilot Enterprise**.

## Objective

The main goal of this solution is to automate the process of assigning or removing GitHub Copilot licenses. Instead of manually managing licenses, the organization can leverage a self-service solution to simply let the users open an issue using the provided issue templates. The workflow will automatically assign or remove the license to them, when the user opens an issue to request a license they are committing to use GitHub Copilot.

[![GitHub Copilot](/images/GitHubCopilot.png)](https://github.com/features/copilot)

## Configure this solution in your organization

1. Click on [Use this template](https://github.com/new?template_name=GitHubCopilotLicenseAssignment&template_owner=microsoft) to create a new repository in your GitHub organization where you want to automate the GitHub Copilot license management solution.

2. [Create two new labels](/../../labels) in the new repository named `copilot-request` and `copilot-remove` to trigger the workflow that assigns or removes the GitHub Copilot license.

![GitHub Repository Label Issues](/images/LabelIssues.jpg)

3. The GitHub Enterprise Owner or GitHub Organization Owner needs to enable GitHub Copilot for the organization where the repository is located. For more details, refer to the [official documentation](https://docs.github.com/copilot/managing-github-copilot-in-your-organization/managing-access-for-copilot-in-your-organization).

![Enable GitHub Copilot](/images/EnableGitHubCopilot.jpg)

4. In the GitHub Organization, create a new **GitHub App** by going to `Settings`, then `Developer settings` and then `GitHub Apps`, click on `New GitHub App` and provide the `Name`, `Homepage URL` (could be any URL) and the following permissions:
   - `repository:metadata:ready-only`
   - `organization:GitHub Copilot Business:read and write`

![GitHub App](/images/GitHubApp.jpg)

5. [Copy the GitHub App ID and store it in the repository secrets](/../../settings/secrets/actions/new) with the name **`APPLICATION_ID`**.

![GitHub App ID](/images/GitHubAppID.jpg)

6. In the `App Settings`, scroll down to the `Private keys` section and click on `Generate a private key` for the GitHub App and [store the plain text of the .pem file in the repository secrets](/../../settings/secrets/actions/new) with the name **`APPLICATION_PRIVATE_KEY`**.

![GitHub App Private Key](/images/PrivateKey.jpg)

The private key should be stored in the repository secrets in the following format:

```
-----BEGIN RSA PRIVATE KEY-----
// The private key
-----END RSA PRIVATE KEY-----
```

7. In the App Settings click on `Install App` in the left menu and select the repository that you created before.   

![Install GitHub App](/images/InstallGitHubApp.jpg)

> [!TIP]
> The workflow use the `GITHUB_TOKEN` to comment and close the issue, this token is automatically created by GitHub and is available to use in the workflow, however the `GITHUB_TOKEN` needs to have `Read and write permissions`, you can check this out in the `Organization settings`, then `Actions` and scroll down to the the `Workflow permissions` section of `General`.

![GitHub Actions Workflow Permission](/images/GitHubTokenPermissions.jpg)

## How users can request an assignment or removal of a GitHub Copilot

> [!NOTE]
> At this point GitHub Copilot should be enabled in the organization, the labels `copilot-request` and `copilot-removal` should be created and the GitHub App configured with the secrets `APPLICATION_ID` and `APPLICATION_PRIVATE_KEY` already stored in the repository.

1. Users can now [open a new issue in this repository using the issue template 'Request GitHub Copilot License'](/../../issues/new?assignees=&labels=copilot-request&projects=&template=github_copilot_request.md&title=Request+GitHub+Copilot+License) or [open a new issue in this repository using the issue template 'Remove GitHub Copilot License'](/../../issues/new?assignees=&labels=copilot-remove&projects=&template=github_copilot_remove.md&title=Remove+GitHub+Copilot+License).

2. The workflow will be triggered automatically and if successful, a GitHub Copilot license will be assigned or removed in the user's account.

## (Optional) Add an approval process to the license assignment or removal

One of the easiest way to add an approval process to the GitHub Copilot license assignment is by doing the following steps:

1. Create a new environment in the repository. Go to `Settings`, then `Environments` and click on `New environment`. Provide a name to the environment, for example, `Copilot` or `copilot-request-approval`.

2. In the environment settings, check the `Required reviewers` option and add the users that will be able to approve the requests. (You can add up to 6 users).

3. In the workflow file [assign_github_copilot_license.yml](/.github/workflows/github_copilot_license_management.yml), add `environment: <name of your environment>`, in the jobs that assigns or removes the GitHub Copilot (line 15 and 99), for example:

```yaml
13. jobs:
14.    assign_github_copilot_license:
15.       environment: Copilot
//...
98.    remove_github_copilot_license:
99.       environment: Copilot
```

Now, when a user opens an issue with the label `copilot-request` or , the workflow will wait for the approval of the users added to the environment before assigning the GitHub Copilot license.

## Repository Content

The repository contains the following files:

### 1. Issue Templates

There are two issue templates provided with this repository:

   - [Assign GitHub Copilot](/.github/ISSUE_TEMPLATE/assign_github_copilot.md) is provided to standardize the license request process. The template uses the label `copilot-request` to trigger the workflow that assigns GitHub Copilot licenses, this label needs to be created in the repository before opening an issue. The template contains a notice to the user about the commitment they're making by requesting the GitHub Copilot license, informing the user about what will happen once their license request is fulfilled, a motivational message for the user, and a thank you note to the user.

   - [Remove GitHub Copilot](/.github/ISSUE_TEMPLATE/remove_github_copilot.md) is provided to standardize the license removal process. The template uses the label `copilot-remove` to trigger the workflow that removes GitHub Copilot licenses, this label needs to be created in the repository before opening an issue. The template contains a notice to the user about the commitment they're making by requesting the GitHub Copilot license removal, informing the user about what will happen once their license removal request is fulfilled, a motivational message for the user, and a thank you note to the user.

### 2. GitHub Action Workflow

The workflow [github_copilot_license_management.yml](/.github/workflows/github_copilot_license_management.yml) is triggered when an issue is labeled with `copilot-request` or `copilot-remove`.  The workflow uses the [GitHub API](https://docs.github.com/rest/copilot/copilot-user-management?apiVersion=2022-11-28) through [octokit](http://octokit.github.io/) to assign the GitHub Copilot license to the user who opened the issue. A token is generated using [Peter Murray's Action](https://github.com/peter-murray/workflow-application-token-action). Upon successful execution, the workflow comments on the issue and closes it. In case of failure, it comments on the issue to notify about the failure and leaves the issue open. The workflow uses the [GITHUB_TOKEN](https://docs.github.com/actions/reference/authentication-in-a-workflow) to comment and close the issue opened by the user. The execution of the workflow takes less than 1 minute.

### 3. README & index.html (web page)

The repository contains an `index.html` file that is used in GitHub Pages as informational website about this solution. The `index.html` file can be removed and the `README.md` file can be updated with a custom description for the users of your GitHub Organization.

## Feedback

If you have any feedback, please [create an issue in this repository](https://github.com/microsoft/GitHubCopilotLicenseAssignment/issues/new). We would love to hear from you! 🚀
