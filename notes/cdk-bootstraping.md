
### What is it?
- It is the process of preparing an environment for deployment. It is an one time action you must perform for every environment (Account , and a Region combined)  that you deploy your resources
- The bootstrapping process provisions resources in your environment; 
    - S3 for storing files 
    - AWS IAM that grant permissions needed 
    - These resources are provisioned in an AWS CloudFormation stack called as bootstrap stack
        - It is usually named as CDKToolkit  and deployed like another stack
- Since Environments are independent, if you need to deploy in multiple environments, each of them should be bootstrapped

### The CDKToolkit Stack
- It is the bootstrap stack for other resources to be deployed in the environment. 
- **IMPORTANT** : Do not delete this stack. Protect your bootstrap stack from deletion using `--termination-protection` option with the cdk bootstrap command
    - `cdk bootstrap --termination-protection`
    - `aws cloudformation describe-stacks --stack-name CDKToolkit --query "Stacks[0].EnableTerminationProtection"`
- **IMPORTANT** : If your bootstrapped environment is deleted the AWS resource originally provisioned to support CDK deployment will also be deleted This will cause the pipeline to stop working
- Update the bootstrap stack as needed by running the `cdk bootstrap` command again

### How To Bootstrap

#### Methods ; 
- Use AWS CDK CLI command  cdk bootstrap  
    - simple method that works well if you have few environments to bootstrap
    - example command to bootstrap in one and in 2 environments
    
    ```
        # Multiple ways to bootstrap using CDK 
        cdk bootstrap aws://123456789012/us-east-1
        cdk bootstrap --profile prod
        cdk bootstrap --template TEMPLATE_FILENAME  # this will bootstrap based on an existing CLOUDFORMATION Template

        cdk bootstrap aws://ACCOUNT-NUMBER-1/REGION-1 aws://ACCOUNT-NUMBER-2/REGION-2 ...
        cdk bootstrap 123456789012/us-east-1 123456789012/us-west-1

    ```

- Deploy the template provided by the AWS CDK CLI using a AWS CloudFormation deployment tool
    - flexible to make small modifications and deploy to large scale deployments
    ```
    # Get a copy of the bootstrap-template.yaml file
    cdk bootstrap --show-template > bootstrap-template.yaml

    # Then use the cloudformation service to deploy using AWS CLI
        aws cloudformation create-stack \
        --stack-name CDKToolkit \
        --template-body file://path/to/bootstrap-template.yaml \
        --capabilities CAPABILITY_NAMED_IAM \
        --region us-west-1
    ```

- If you bootstrap more than once, the bootstrap stack will get upgraded.
- Here are the default AWS resources created with default CDK bootstrapping for an account and in an region

|Object Logical ID	| AWS Resource|	Default Values used during deploy	|
|--------------------|-----------|-------------------------------------|
CdkBootstrapVersion	|AWS::SSM::Parameter	|/cdk-bootstrap/hnb659fds/version 	|
CloudFormationExecutionRole	|AWS::IAM::Role	|cdk-hnb659fds-cfn-exec-role-341224537620-us-east-2 	|
DeploymentActionRole|	AWS::IAM::Role|	cdk-hnb659fds-deploy-role-341224537620-us-east-2 	|
FilePublishingRole|	AWS::IAM::Role	|cdk-hnb659fds-file-publishing-role-341224537620-us-east-2 	|
FilePublishingRoleDefaultPolicy|	AWS::IAM::Policy|	CDKTo-FileP-kRppuGoOa6x2	|
ImagePublishingRole|	AWS::IAM::Role|	cdk-hnb659fds-image-publishing-role-341224537620-us-east-2 	|
ImagePublishingRoleDefaultPolicy|	AWS::IAM::Policy|	CDKTo-Image-jJeA0ejiOO60	|
LookupRole	|AWS::IAM::Role	|cdk-hnb659fds-lookup-role-341224537620-us-east-2 	|
StagingBucket	|AWS::S3::Bucket|	cdk-hnb659fds-assets-341224537620-us-east-2 	|
StagingBucketPolicy	|AWS::S3::BucketPolicy|	cdk-hnb659fds-assets-341224537620-us-east-2	|
ContainerAssetsRepository	|AWS::ECR::Repository|	cdk-hnb659fds-container-assets-341224537620-us-east-2 	|


*  when you run cdk bootstrap the CDK CLI synthesizes the CDK app in the current director

### Customizing Bootstrap
- Two ways to customize
    1) cdk bootstrap command line args
    2) modify the default bootstrap template and deploy it yourself
        - to do that use cdk bootstrap --show-template   and get the template for modification
        - after modification use cdk bootstrap --template bootstrap-template.yaml

-  cdk bootstrap customization
    - --bootstrap-bucket-name 
        - AWS CDK app needs to know the bootstrap bucket in order for it to synthesize a stack for deployment
    - --bootstrap-kms-key-id overrides the AWS KMS key used to encrypt the S3 bucket.
    - --cloudformation-execution-policies 
        - specifies the ARNs of managed policies that should be attached to the deployment role assumed by AWS CloudFormation during deployment of your stacks. 
        - By default, stacks are deployed with full administrator permissions using the AdministratorAccess policy.
        - example

        ` --cloudformation-execution-policies "arn:aws:iam::aws:policy/AWSLambda_FullAccess,arn:aws:iam::aws:policy/AWSCodeDeployFullAccess".`


    - --qualifier  - a string added to the name of all resources in the bootstrap stack - This qualifier allows us to provision multiple bootstrap stacks in the same environment 
    - --tags   adda AWS ClloudFormation tags
    - --trust lists the AWS accounts that may deploy into the environment being bootstrapped. Use this flag when bootstrapping an environment that a CDK Pipeline in another environment will deploy into. The account doing the bootstrapping is always trusted.
    - --trust-for-lookup   lists the AWS accounts that may look up context information from the environment being bootstrapped
    - --termination-protection 

### Stack Synthesizers
- The stack synthesizer is an AWS CDK class that controls how the stack's template is synthesized.
- The AWS CDK's built-in stack synthesizers is called DefaultStackSynthesizer. It includes capabilities for cross-account deployments and https://docs.aws.amazon.com/cdk/v2/guide/cdk_pipeline.html deployment
- If you don't provide the synthesizer property, DefaultStackSynthesizer is used.
- When the default synthesizer is not enough you can create your own customer synthesizer using the class that implements IStackSynthesizer 
- Some customization you can do on the default synthesizer class properties
    - for qualifiers  and changing the resource names
# this code can be in app.py

    ```
        MyStack(self, "MyStack",
            synthesizer=DefaultStackSynthesizer(
                qualifier="MYQUALIFIER"


          # Name of the S3 bucket for file assets
          file_assets_bucket_name="cdk-${Qualifier}-assets-${AWS::AccountId}-${AWS::Region}",
          bucket_prefix="",
        
          # Name of the ECR repository for Docker image assets
          image_assets_repository_name="cdk-${Qualifier}-container-assets-${AWS::AccountId}-${AWS::Region}",
        
          # ARN of the role assumed by the CLI and Pipeline to deploy here
          deploy_role_arn="arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-deploy-role-${AWS::AccountId}-${AWS::Region}",
          deploy_role_external_id="",
        
          # ARN of the role used for file asset publishing (assumed from the CLI role)
          file_asset_publishing_role_arn="arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-file-publishing-role-${AWS::AccountId}-${AWS::Region}",
          file_asset_publishing_external_id="",
        
          # ARN of the role used for Docker asset publishing (assumed from the CLI role)
          image_asset_publishing_role_arn="arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-image-publishing-role-${AWS::AccountId}-${AWS::Region}",
          image_asset_publishing_external_id="",
        
          # ARN of the role passed to CloudFormation to execute the deployments
          cloud_formation_execution_role="arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-cfn-exec-role-${AWS::AccountId}-${AWS::Region}",
        
          # ARN of the role used to look up context information in an environment
          lookup_role_arn="arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-lookup-role-${AWS::AccountId}-${AWS::Region}",
          lookup_role_external_id="",
        
          # Name of the SSM parameter which describes the bootstrap stack version number
          bootstrap_stack_version_ssm_parameter="/cdk-bootstrap/${Qualifier}/version",
        
          # Add a rule to every template which verifies the required bootstrap stack version
          generate_bootstrap_version_rule=True,

        
        ))
        )

    ```

    ```
        # qualifiers can also be customized through the cdk.json

        {
        "app": "...",
        "context": {
            "@aws-cdk/core:bootstrapQualifier": "MYQUALIFIER"
        }
        }

    ```

### Bootstrapping Template Contract
- DefaultStack Synthesizer generated template conforms to the bootstrap template contract.
- For custom synthesizer - the class should conform to the template contract



### Security Hub
- AWS Security hub can be used to report on the resources created by the CDK bootstrapping process
- Based on the findings, you need to fix them and redeploy as needed