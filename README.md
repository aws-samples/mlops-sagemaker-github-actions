# MLOps using Amazon SageMaker and GitHub Actions
This is an example of MLOps implementation using Amazon SageMaker and GitHub Actions.

In this example, we will automate a model-build pipeline that includes steps for data preparation, model training, model evaluation, and registration of that model in the SageMaker Model Registry. The resulting trained ML model is deployed from the model registry to staging and production environments upon the approval.


## Architecture Overview
![Amazon SageMaker and GitHub Actions Architecture](/img/Amazon-SageMaker-GitHub-Actions-Architecture.png)


## Prerequisites
The followings are prerequisites to completing the steps in this example:


### Set up a CodeStar Connection
If you don't have a CodeStar Connection to your GitHub account already, follow this [link](https://docs.aws.amazon.com/dtconsole/latest/userguide/connections-create-github.html) to create one.

Your CodeStar Connection ARN will look like this:
```
arn:aws:codestar-connections:us-west-2:account_id:connection/aEXAMPLE-8aad-4d5d-8878-dfcab0bc441f
```
In the above, `aEXAMPLE-8aad-4d5d-8878-dfcab0bc441f` is the unique Id for this connection. We'll be using this Id when we create our SageMaker project later in this example.

### Set up Secret Access Keys for GitHub Token
We need to create a secret in AWS secret Manager that holds our GitHub personal access token. If you do not have a personal access token for GitHub, you need to create one following the instructions here: create personal access token

Then, go to the AWS Secrets Manager, click on Store a new secret, select “Other type of secret” for Choose Secret type, then give a name to your secret in the “key” and add your personal access token to its associated “value”- click next, type a name for your Secret name and click next and then store.

### Create an IAM user for GitHub Actions
In order to give permission to the GitHub Actions to deploy the SageMaker endpoints in your AWS environment, you need to create an IAM user.
Use `iam/GithubActionsMLOpsExecutionPolicy.json` to provide enough permission to this user to deploy your endpoints.

Next, generate an *Access Key* for this user. You'll use this key in the next step when you set up your GitHub Secrets.


### GitHub Setup
The following are the steps to prepare your github account to run this example.


#### a) GitHub Repository
You can reuse an existing github repo for this example. However, it's easier if you create a new repository. This repository is going to contain all the source code for both sagemaker pipeline build and deployments.

Copy the contents of the `seedcode` directory in the root of your github repository. e.g. `.github` directory should be under the root of your github repo.


#### b) Set up a GitHub Secrets containing your IAM user access key
Got your GitHub repository- on the top of repository select Setting- then in the security section go to the Secrets and variables and choose Actions. Choose the New repository secret :
1- Add the name AWS_ACCESS_KEY_ID and for Secret that you created for the IAM user in the [Create an IAM user for GitHub Actions](https://github.com/aws-samples/mlops-sagemaker-github-actions#create-an-iam-user-for-github-actions) step add your AWS_ACCESS_Key, click on add secret.
2- repeat the same process for AWS_SECRET_ACCESS_KEY

#### c) GitHub Environments
In order to create a manual approval step in our deployment pipelines, we use [GitHub Environments](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment).

1. Go to the Settings>Environments menu of your github repository and create a new environment called `production`.
2. In the *Environment protection rules* select the `Required reviewers` and then add the reviewers. You can choose yourself for this example.

>Note: Environment feature is not available in some types of GitHub plans. Check the documentation [here](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment).


### Deploy the Lambda function
Simply zip the `lambda_function.py` and upload it to an S3 bucket.

```sh
cd lambda_functions/lambda_github_workflow_trigger
zip lambda-github-workflow-trigger.zip lambda_function.py
```
Then upload the `lambda-github-workflow-trigger.zip` to a bucket which can be accessed later on by the ServiceCatalog.

#### Create an AWS Lambda layer
Now, let's create a Lambda layer for the dependencies of the lambda_function which we just uploaded.

Create a python virtual environment and install the dependencies.
```sh
mkdir lambda_layer
cd lambda_layer
python3 -m venv .env
source .env/bin/activate
pip install pygithub
deactivate
```
Now let's create our zip file.
```sh
mv .env/lib/python3.9/site-packages/ python
zip -r layer.zip python
```

Publish the layer to AWS.
```sh
aws lambda publish-layer-version --layer-name python39-github-arm64 \
    --description "Python3.9 pygithub" \
    --license-info "MIT" \
    --zip-file fileb://layer.zip \
    --compatible-runtimes python3.9 \
    --compatible-architectures "arm64"
```
Now, all of your functions can refer to this layer to satisfy their dependencies.


For further reading on Lambda Layer, visit this [link](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html).


### Create a Custom Project Template in SageMaker
If the SageMaker-provided templates do not meet your needs (for example, you want to have more complex orchestration in the CodePipeline with multiple stages or custom approval steps), create your own templates.
We recommend starting by using SageMaker-provided templates to understand how to organize your code and resources and build on top of it. https://docs.aws.amazon.com/sagemaker/latest/dg/sagemaker-projects-templates-custom.html

To do this, after you enable administrator access to the SageMaker templates,

 1. log in to the https://console.aws.amazon.com/servicecatalog/

 2. On the AWS **Service Catalog** console, under Administration, choose **Portfolios**.

 3. Choose Create a new portfolio.

 4. Name the portfolio **SageMaker Organization Templates**.

 5. Download the [**template.yml**](https://github.com/aws-samples/mlops-sagemaker-github-actions/blob/main/project/template.yml) to your computer. This template is a Cloud Formation tempalte that provisions all the CI/CD resources we need as configuarion and infrustruce as code. You can study the template in more details to see what resources are deployed as part of it. This template has been customised to integrate with GitHub and GitHub actions.

 6. Choose the new portfolio.

 7. Choose **Upload a new product**.

 8. For Product name¸ enter a name for your template. We chose **build-deploy-template**.

 9. For Description, enter **my custom build and deploy template**.

 10. For Owner, enter your **name**.

 11. Under Version details, for Method, choose **Use a template file**.

 12. Choose **Upload a template**.

 13. Upload the template you downloaded.

 14. For Version title, choose **1.0**.

 15. Choose Review.

 16. Review your settings and choose **Create product**.

 17. Choose Refresh to list the new product.

 18. Choose the product you just created.

 19. On the Tags tab, add the following tag to the product:

  - Key – **sagemaker:studio-visibility**
  - Value – **true**

 20. Back in the portfolio details, you see something similar to the following screenshot (with different IDs). ![alt text](/img/img1.png)

 21. On the Constraints tab, choose **Create constraint**.

 22. For Product, choose build-deploy-template (the product you just created).

 23. For Constraint type, choose Launch.

 24. Under Launch Constraint, for Method, choose Select IAM role.

 25. Choose **AmazonSageMakerServiceCatalogProductsLaunchRole**.

 26. Choose Create.

 27. On the Groups, roles, and users tab, choose Add groups, roles, users.

 28. On the **Roles** tab, select the **role** you used when configuring your SageMaker Studio domain. This is where the SageMaker Domain Role can be found. ![alt text](/img/img2.png)

 29. Choose Add access.


## Launch your project

In the previous sections, you prepared the Custom MLOps project environment. The next step is to create a project using your new template.
TBA


## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

