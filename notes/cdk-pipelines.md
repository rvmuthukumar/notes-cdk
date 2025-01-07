### General
- CDK Pipelines are high level structure that allows you to quickly define a complete delivery pipeline for your application.
- The https://docs.aws.amazon.com/cdk/latest/guide/cdk_pipeline.html construct makes that process easy and streamlined from within your existing CDK infrastructure design. 
- The pipelines consist of “stages” that represent the phases of your deployment process from how the source code is managed, to how the fully built artifacts are deployed. 
    - A Stage in a CDK pipeline represents a set of one or more CDK Stacks that should be deployed together, to a particular environment. 
 
- Pipeline is to deploy our application stack; we no longer need the main CDK application to deploy our app 
- The pipelines use Cloud formation for the entire deployment and NOT the code deploy - everything is happening with the cloud formation 


### CDK Pipeline Structure
- Source - fetch from the repo and trigger the pipeline every time you push the new commits to it
- Build - Compile, Synth, Output Cloud Assembly