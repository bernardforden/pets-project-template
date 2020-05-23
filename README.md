### HOW TO: Run your pets-project in a minute

**1. Use pets-project template repository**

- Click on `Use this template` button or https://github.com/simplify-framework/pets-project-template/generate
- Name your project URL as https://github.com/your_github_account/pets-project
- Click on `Actions` tab to see the predefined CI/CD flow runs with error.

**2. Setup CI/CD flows with your AWS account**

- Setup your [root credentials](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html) to be used to provision a secure Github credentials 
  + A root credentials must have the following permissions to create Github user account:
    + iam:CreateUser
    + iam:CreateRole
    + iam:PutUserPolicy
    + iam:PutRolePolicy

- Prepare a Project Id and AWS Account ID
  + Choose a random number for your project Id: (e.g: `66640738`)
  + Find your AWS Account ID https://docs.aws.amazon.com/IAM/latest/UserGuide/console_account-alias.html#FindingYourAWSId

- Execute the following commands under root credentials to create Github user account:
  + Generate `petsample` with `simplify-codegen` command lines
    + `npm install -g simplify-codegen`
    + `simplify-codegen template -i petsample`
    + `simplify-codegen generate -i openapi.yaml -p your_project_id -a your_aws_account_id`
  
    It will generate the wole project template with an `openapi.yaml` sample and five policy documents
      + policy-assume-role.json: the permission for Github user account can assume a specific action role
      + policy-relationship-role.json a trust relationship between Github account and the `ProjectPetsDemoRole` action role
      + policy-deployment.json is the action policy that allows cloudformation to create resources for pets-project
      + policy-services.json is the action policy that allows cloudformation to setup resources for pets-project
      + policy-execute-api.json is the action policy that allows CI/CD flow can run test-apis with IAM Sigv4 authorization

  + Create a Github user account with Assume Role permission only:
    + `aws iam create-user --user-name ProjectPetsDemo`
    + `aws iam put-user-policy --user-name ProjectPetsDemo --policy-name GithubUserAccessRole --policy-document policy-assume-role.json` 
  + Create a role with limited permission for `pets-project` resources only:
    + `aws iam create-role --role-name ProjectPetsDemoRole --assume-role-policy-document policy-relationship-role.json`
    + `aws iam put-role-policy --role-name ProjectPetsDemoRole --policy-name ProjectPetsDemoRolePolicy --policy-document policy-deployment.json`
    + `aws iam put-role-policy --role-name ProjectPetsDemoRole --policy-name ProjectPetsDemoRolePolicy --policy-document policy-services.json`
    + `aws iam put-role-policy --role-name ProjectPetsDemoRole --policy-name ProjectPetsDemoRolePolicy --policy-document policy-execute-api.json`

**3. Setup Github Actions Secrets**

  + Generate Access Key crediential for Github user account, notes this key to use for the next step
    + `aws iam create-access-key --user-name ProjectPetsDemo`
  + Go to https://github.com/your_github_account/pets-project/settings/secrets
    + `AWS_ACCESS_KEY_ID` with value is the access key ID from step above
    + `AWS_SECRET_ACCESS_KEY` with value is the secret key from step above
    + `AWS_ACCOUNT_ID` with your AWS Account ID number (e.g: 1234567890)
    + `AWS_ROLE_EXTERNAL_ID` with `ProjectPetsDemo-66640738` the same with policy-relationship-role.json
    + `AWS_ROLE_TO_ASSUME` with the ARN of `ProjectPetsDemoRole` created by step 2 with `create-role`
    + `PROJECT_ID` with the choosen number of the first step, it is: `66640738`
      
**4. Go to Github Actions and re-run the failed jobs**

  + You can trigger this CI/CD flows by any pull_request or push to the master branch
  > The CI/CD flow will re-generate code, deploy stack, push function code and run all tests...
