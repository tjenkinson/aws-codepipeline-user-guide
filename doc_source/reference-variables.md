# Variables<a name="reference-variables"></a>

This section is a reference only\. For information about creating variables, see [Working with variables](actions-variables.md)\.

Variables allow you to configure your pipeline actions with values that are determined at the time of the action execution\. Variables can be produced by an action execution or be implicitly available at the start of each pipeline execution\.

Some action providers produce a defined set of variables\. You choose from default variable keys for that action provider, such as commit ID\.

To see step\-by\-step examples of using variables:
+ For a tutorial with a Lambda action that uses variables from an upstream action \(CodeCommit\) and generates output variables, see [Tutorial: Using variables with Lambda invoke actions](tutorials-lambda-variables.md)\.
+ For a tutorial with a AWS CloudFormation action that references stack output variables from an upstream CloudFormation action, see [Tutorial: Create a pipeline that uses variables from AWS CloudFormation deployment actions](tutorials-cloudformation-action.md)\.
+ For an example manual approval action with message text that references output variables that resolve to the CodeCommit commit ID and commit message, see [Example: Use variables in manual approvals](actions-variables.md#actions-variables-examples-approvals)\.
+ For an example CodeBuild action with an environment variable that resolves to the GitHub branch name, see [Example: Use a BranchName variable with CodeBuild environment variables](actions-variables.md#actions-variables-examples-env-branchname)\.
+ CodeBuild actions produce as variables all environment variables that were exported as part of the build\. For more information, see [CodeBuild action output variables](#reference-variables-list-configured-codebuild)\.

**Variable Limits**

For limit information, see [Quotas in AWS CodePipeline](limits.md)\.

**Note**  
When you enter output variable syntax in the action configuration fields, do not exceed the 1000\-character limit for the configuration fields\. A validation error is returned when this limit is exceeded\.

**Topics**
+ [Concepts](#reference-variables-concepts)
+ [Configuring variables](#reference-variables-workflow)
+ [Variable resolution](#reference-variables-resolution)
+ [Rules for variables](#reference-variables-rules)
+ [Variables available for pipeline actions](#reference-variables-list)

## Concepts<a name="reference-variables-concepts"></a>

This section lists key terms and concepts related to variables and namespaces\.

### Variables<a name="reference-variables-concepts-variablekeys"></a>

Variables are key\-value pairs that can be used to dynamically configure actions in your pipeline\. There are currently two ways these variables are made available:
+ There is a set of variables that are implicitly available at the start of each pipeline execution\. This set currently includes `PipelineExecutionId`, the ID of the current pipeline execution\.
+ There are action types that produce sets of variables when they are executed\. You can see the variables produced by an action by inspecting the `outputVariables` field that is part of the [ListActionExecutions](https://docs.aws.amazon.com/codepipeline/latest/APIReference/API_ListActionExecutions.html) API\. To see which variables each action type produces, see the CodePipeline [Action structure reference](action-reference.md)\.

To reference these variables in your action configuration, you must use the variable reference syntax with the correct namespace\. 

For a list of available key names by action provider, see [Variables available for pipeline actions](#reference-variables-list)\. For an example variable workflow, see [Configuring variables ](#reference-variables-workflow)\.

### Namespaces<a name="reference-variables-concepts-namespaces"></a>

To ensure that variables can be uniquely referenced, they must be assigned to a namespace\. After you have a set of variables assigned to a namespace, they can be referenced in an action configuration by using the namespace and variable key with the following syntax:

```
#{namespace.variable_key}
```

There are two types of namespaces under which variables can be assigned:
+ **The codepipeline reserved namespace**

  This is the namespace assigned to the set of implicit variables available at the start of each pipeline execution\. This namespace is `codepipeline`\. Example variable reference:

  ```
  #{codepipeline.PipelineExecutionId}
  ```
+ **Action assigned namespace**

  This is a namespace that you assign to an action\. All variables produced by the action fall under this namespace\. To make the variables produced by an action available for use in a downstream action configuration, you must configure the producing action with a namespace\. Namespaces must be unique across the pipeline definition and cannot conflict with any artifact names\. Here is an example variable reference for an action configured with a namespace of `SourceVariables`\.

  ```
  #{SourceVariables.VersionId}
  ```

## Configuring variables<a name="reference-variables-workflow"></a>

You configure an action to produce variables by declaring a namespace for the action\. The action must already be one of the action providers that generates variables\. Otherwise, the variables available are pipeline\-level variables\.

You declare the namespace either by: 
+ On the **Edit action ** page of the console, entering a namespace in **Variable namespace**\.
+ Entering a namespace in the `namespace` parameter field in the JSON pipeline structure\. 

In this example, you add the `namespace` parameter to the CodeCommit source action with the name `SourceVariables`\. This configures the action to produce the variables available for that action provider, such as `CommitId`\. 

```
{
    "name": "Source",
    "actions": [
        {
            "outputArtifacts": [
                {
                    "name": "SourceArtifact"
                }
            ],
            "name": "Source",
            "namespace": "SourceVariables",
            "configuration": {
                "RepositoryName": "MyRepo",
                "BranchName": "mainline",
                "PollForSourceChanges": "false"
            },
            "inputArtifacts": [],
            "region": "us-west-2",
            "actionTypeId": {
                "provider": "CodeCommit",
                "category": "Source",
                "version": "1",
                "owner": "AWS"
            },
            "runOrder": 1
        }
    ]
},
```

Next, you configure the downstream action to use the variables produced by the previous action\. You do this by:
+ On the **Edit action ** page of the console, entering the variable syntax \(for the downstream action\) in the action configuration fields\.
+ Entering the variable syntax \(for the downstream action\) in the action configuration fields in the JSON pipeline structure

In this example, the build action's configuration field shows environment variables that are updated upon the action execution\. The example specifies the namespace and variable for execution ID with `#{codepipeline.PipelineExecutionId}` and the namespace and variable for commit ID with **\#\{SourceVariables\.CommitId\}**\. 

```
{
    "name": "Build",
    "actions": [
        {
            "outputArtifacts": [
                {
                    "name": "BuildArtifact"
                }
            ],
            "name": "Build",
            "configuration": {
                "EnvironmentVariables": "[{\"name\":\"Release_ID\",\"value\":\"#{codepipeline.PipelineExecutionId}\",\"type\":\"PLAINTEXT\"},{\"name\":\"Commit_ID\",\"value\":\"#{SourceVariables.CommitId}\",\"type\":\"PLAINTEXT\"}]",
                "ProjectName": "env-var-test"
            },
            "inputArtifacts": [
                {
                    "name": "SourceArtifact"
                }
            ],
            "region": "us-west-2",
            "actionTypeId": {
                "provider": "CodeBuild",
                "category": "Build",
                "version": "1",
                "owner": "AWS"
            },
            "runOrder": 1
        }
    ]
},
```

## Variable resolution<a name="reference-variables-resolution"></a>

Each time an action is executed as part of a pipeline execution, the variables it produces are available for use in any action that is guaranteed to occur after the producing action\. To use these variables in a consuming action, you can add them to the consuming action's configuration using the syntax shown in the previous example\. Before it performs a consuming action, CodePipeline resolves all of the variable references present in the configuration prior to initiating the action execution\.



![\[Example: Variables for multiple actions\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/images/variables-workflow-example.png)![\[Example: Variables for multiple actions\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/)![\[Example: Variables for multiple actions\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/)

## Rules for variables<a name="reference-variables-rules"></a>

The following rules help you with the configuration of variables:
+ You specify the namespace and variable for an action through a new action property or by editing an action\. 
+ When you use the pipeline creation wizard, the console generates a namespace for each action created with the wizard\.
+ If the namespace isn't specified, the variables produced by that action cannot be referenced in any action configuration\.
+ To reference variables produced by an action, the referencing action must occur after the action that produces the variables\. This means it is either in a later stage than the action producing the variables, or in the same stage but at a higher run order\.

## Variables available for pipeline actions<a name="reference-variables-list"></a>

The action provider determines which variables can be generated by the action\. 

For step\-by\-step procedures for managing variables, see [Working with variables](actions-variables.md)\.

### Actions with defined variable keys<a name="reference-variables-list-defined"></a>

Unlike a namespace which you can choose, the following actions use variable keys that cannot be edited\. For example, for the Amazon S3 action provider, only the `ETag` and `VersionId` variable keys are available\.

Each execution also has a set of CodePipeline\-generated pipeline variables that contain data about the execution, such as the pipeline release ID\. These variables can be consumed by any action in the pipeline\.

**Topics**
+ [CodePipeline execution ID output variable](#w23aac51c29b7b9)
+ [Amazon ECR action output variables](#w23aac51c29b7c11)
+ [AWS CloudFormation StackSets action output variables](#w23aac51c29b7c15)
+ [CodeCommit action output variables](#w23aac51c29b7c19)
+ [CodeStarSourceConnection action output variables](#w23aac51c29b7c21)
+ [GitHub action output variables \(GitHub action version 1\)](#w23aac51c29b7c23)
+ [S3 action output variables](#w23aac51c29b7c25)

#### CodePipeline execution ID output variable<a name="w23aac51c29b7b9"></a>


**CodePipeline execution ID variable**  

| Provider | Variable key | Example value | Example variable syntax | 
| --- | --- | --- | --- | 
| codepipeline | PipelineExecutionId | 8abc75f0\-fbf8\-4f4c\-bfEXAMPLE | \#\{codepipeline\.PipelineExecutionId\} | 

#### Amazon ECR action output variables<a name="w23aac51c29b7c11"></a>


**Amazon ECR variables**  

| Variable key | Example value | Example variable syntax | 
| --- | --- | --- | 
| ImageDigest | sha256:EXAMPLE1122334455 | \#\{SourceVariables\.ImageDigest\} | 
| ImageTag | latest | \#\{SourceVariables\.ImageTag\} | 
| ImageURI | 11111EXAMPLE\.dkr\.ecr\.us\-west\-2\.amazonaws\.com/ecs\-repo:latest | \#\{SourceVariables\.ImageURI\} | 
| RegistryId | EXAMPLE12233 | \#\{SourceVariables\.RegistryId\} | 
| RepositoryName | my\-image\-repo | \#\{SourceVariables\.RepositoryName\} | 

#### AWS CloudFormation StackSets action output variables<a name="w23aac51c29b7c15"></a>


**AWS CloudFormation StackSets variables**  

| Variable key | Example value | Example variable syntax | 
| --- | --- | --- | 
| OperationId | 11111111\-2bbb\-111\-2bbb\-11111example | \#\{DeployVariables\.OperationId\} | 
| StackSetId | my\-stackset:1111aaaa\-1111\-2222\-2bbb\-11111example | \#\{DeployVariables\.StackSetId\} | 

#### CodeCommit action output variables<a name="w23aac51c29b7c19"></a>


**CodeCommit variables**  

| Variable key | Example value | Example variable syntax | 
| --- | --- | --- | 
| AuthorDate | 2019\-10\-29T03:32:21Z | \#\{SourceVariables\.AuthorDate\} | 
| BranchName | development | \#\{SourceVariables\.BranchName\} | 
| CommitId |  exampleb01f91b31  | \#\{SourceVariables\.CommitId\} | 
| CommitMessage | Fixed a bug \(100 KB maximum size\) | \#\{SourceVariables\.CommitMessage\} | 
| CommitterDate | 2019\-10\-29T03:32:21Z | \#\{SourceVariables\.CommitterDate\} | 
| RepositoryName | myCodeCommitRepo | \#\{SourceVariables\.RepositoryName\} | 

#### CodeStarSourceConnection action output variables<a name="w23aac51c29b7c21"></a>


**`CodeStarSourceConnection` variables \(Bitbucket, GitHub, and GitHub Enterprise Repository\)**  

| Variable key | Example value | Example variable syntax | 
| --- | --- | --- | 
| AuthorDate | 2019\-10\-29T03:32:21Z | \#\{SourceVariables\.AuthorDate\} | 
| BranchName | development | \#\{SourceVariables\.BranchName\} | 
| CommitId |  exampleb01f91b31  | \#\{SourceVariables\.CommitId\} | 
| CommitMessage | Fixed a bug \(100 KB maximum size\) | \#\{SourceVariables\.CommitMessage\} | 
| ConnectionArn | arn:aws:codestar\-connections:region:account\-id:connection/connection\-id | \#\{SourceVariables\.ConnectionArn\} | 
| FullRepositoryName | username/GitHubRepo | \#\{SourceVariables\.FullRepositoryName\} | 

#### GitHub action output variables \(GitHub action version 1\)<a name="w23aac51c29b7c23"></a>


**GitHub variables \(GitHub action version 1\)**  

| Variable key | Example value | Example variable syntax | 
| --- | --- | --- | 
| AuthorDate | 2019\-10\-29T03:32:21Z | \#\{SourceVariables\.AuthorDate\} | 
| BranchName | main | \#\{SourceVariables\.BranchName\} | 
| CommitId |  exampleb01f91b31  | \#\{SourceVariables\.CommitId\} | 
| CommitMessage |  Fixed a bug \(100 KB maximum size\)  | \#\{SourceVariables\.CommitMessage\} | 
| CommitterDate | 2019\-10\-29T03:32:21Z | \#\{SourceVariables\.CommitterDate\} | 
| CommitUrl |  | \#\{SourceVariables\.CommitUrl\} | 
| RepositoryName | myGitHubRepo | \#\{SourceVariables\.RepositoryName\} | 

#### S3 action output variables<a name="w23aac51c29b7c25"></a>


**S3 variables**  

| Variable key | Example value | Example variable syntax | 
| --- | --- | --- | 
| ETag | example28be1c3 | \#\{SourceVariables\.ETag\} | 
| VersionId | exampleta\_IUQCv | \#\{SourceVariables\.VersionId\} | 

### Actions with user\-configured variable keys<a name="reference-variables-list-configured"></a>

For CodeBuild, AWS CloudFormation, and Lambda actions, the variable keys are configured by the user\.

**Topics**
+ [CloudFormation action output variables](#w23aac51c29b9b7)
+ [CodeBuild action output variables](#reference-variables-list-configured-codebuild)
+ [Lambda action output variables](#w23aac51c29b9c11)

#### CloudFormation action output variables<a name="w23aac51c29b9b7"></a>


**AWS CloudFormation variables**  

| Variable key | Example variable syntax | 
| --- | --- | 
| For AWS CloudFormation actions, variables are produced from any values designated in the `Outputs` section of a stack template\. Note that the only CloudFormation action modes that generate outputs are those that result in creating or updating a stack, such as stack creation, stack updates, and change set execution\. The corresponding action modes that generate variables are:[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/reference-variables.html)For a tutorial that shows you how to create a pipeline with an AWS CloudFormation deployment action in a pipeline that uses AWS CloudFormation output variables, see [Tutorial: Create a pipeline that uses variables from AWS CloudFormation deployment actions](tutorials-cloudformation-action.md)\. | \#\{DeployVariables\.StackName\} | 

#### CodeBuild action output variables<a name="reference-variables-list-configured-codebuild"></a>


**CodeBuild variables**  

| Variable key | Example variable syntax | 
| --- | --- | 
|  For CodeBuild actions, variables are produced from values generated by exported environment variables\. Set up a CodeBuild environment variable by editing your CodeBuild action in CodePipeline or by adding the environment variable to the build spec\. Add instructions to your CodeBuild build spec to add the environment variable under the exported variables section\. See [env/exported\-variables](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html#build-spec.env.exported-variables) in the *AWS CodeBuild User Guide*\.  |  <pre>#{BuildVariables.EnvVar}</pre>  | 

#### Lambda action output variables<a name="w23aac51c29b9c11"></a>


**Lambda variables**  

| Variable key | Example variable syntax | 
| --- | --- | 
| The Lambda action will produce as variables all key\-value pairs that are included in the outputVariables section of the [PutJobSuccessResult API](https://docs.aws.amazon.com/codepipeline/latest/APIReference/API_PutJobSuccessResult.html) request\. For a tutorial with a Lambda action that uses variables from an upstream action \(CodeCommit\) and generates output variables, see [Tutorial: Using variables with Lambda invoke actions](tutorials-lambda-variables.md)\. | \#\{TestVariables\.testRunId\} | 