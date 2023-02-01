# MLOps using Amazon SageMaker and GitHub Actions
This is an example of MLOps implementation using Amazon SageMaker and GitHub Actions.

In this example, we will automate a model-build pipeline that includes steps for data preparation, model training, model evaluation, and registration of that model in the SageMaker Model Registry. The resulting trained ML model is deployed from the model registry to staging and production environments upon the approval.


## Architecture Overview
![Amazon SageMaker and GitHub Actions Architecture](/img/Amazon-SageMaker-GitHub-Actions-Architecture.png)


## Prerequisites
The followings are prerequisites to completing the steps in this example:

### Create a Custom Project Template in SageMaker


### Setup a CodeStar Connection


### Secret Access Keys for GitHub Token


### Create an IAM user for GitHub Actions
In order to give permission to the GitHub Actions to deploy the SageMaker endpoints in your AWS environment, you need to create an IAM user.
Use `iam/GithubActionsMLOpsExecutionPolicy.json` to provide enough permission to this user to deploy your endpoints.

Next, generate an *Access Key* for this user. You'll use this key in the next step when you set up your GitHub Secrets.

### GitHub Setup
The following are the steps to prepare your github account to run this example.

#### GitHub Repository
You can reuse an existing github repo for this example. However, it's easier if you create a new repository. This repository is going to contain all the source code for both sagemaker pipeline build and deployments.

Copy the contents of the `seedcode` directory in the root of your github repository. e.g. `.github` directory should be under the root of your github repo.

#### GitHub Secrets


#### GitHub Environments
In order to create a manual approval step in our deployment pipelines, we use [GitHub Environments](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment).

1. Go to the Settings>Environments menu of your github repository and create a new environment called `production`.
2. In the *Environment protection rules* select the `Required reviewers` and then add the reviewers. You can choose yourself for this example.

>Note: Environment feature is not available in some types of GitHub plans. Check the documentation [here](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment).
### Create an AWS Lambda layer


## Launch the SageMaker Project


## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

