
### CLI Commands


```
cdk ls   # lists all stacks in the app

cdk bootstrap
# Prepare an AWS environment for CDK deployments by deploying the CDK bootstrap stack, named CDKToolkit, #into the AWS environment.The bootstrap stack is a CloudFormation stack that provisions an Amazon S3 #bucket and Amazon ECR repository in the AWS environment. The AWS CDK CLI uses these resources to store #synthesized templates and related assets during deployment.


cdk synth # emits the synthesized CloudFormation Template
cdk synth --version-reporting false --path-metadata false # will generate CloudFormation Template without                                                           # metadata

cdk deploy # deploys the stack to the default account/region
cdk deploy --hotswap  # NEVER do this in PROD  - only use this in DEV to speed up the dev
                      # It assesses whether a hotswap deployment can be made instead of cloudformation                       # deployment.
                      # In a hotswap deployment, the CDK will be using AWS service API to make change
                      # directly and introduce a drift on CloudFormation
                       
cdk watch # similar to deploy. It monitors your code and assets for changes and deploys only when a change
          # is detected.
          # watch uses cdk.json file "watch" settings. Use the include** and the exclude** to determine
          # which files to watch - refer the AWS Constructs section for details
                              
cdk watch --no-hotswap # By default watch uses hotswap. You have to disable it when doing it in PROD
                                 

cdk diff  # compares deployed stack to the current state

cdk destroy # To remove and then do a CDK deploy - AWS Best practice ** verify this

cdk metadata  # to prevent posting metadata to AWS. Go to the cdk.json file and add
              # "VersionReporting.False"


```