# Quotas in AWS CodePipeline<a name="limits"></a>

CodePipeline has quotas for the number of pipelines, stages, actions, and webhooks that an AWS account can have in each AWS Region\. These quotas apply per Region and can be increased\. To request an increase, use the [Support Center console](https://console.aws.amazon.com/support/v1#/case/create?issueType=service-limit-increase)\.

It can take up to two weeks to process requests for a quota increase\.


| Resource | Default | 
| --- | --- | 
|  Length of time before an action times out  |  Approval action: 7 days AWS CloudFormation deployment action: 3 days CodeBuild build action and test action: 8 hours CodeDeploy and CodeDeploy ECS \(blue/green\) deployment actions: 5 days AWS Lambda invoke action: 20 minutes  While the action is running, CodePipeline periodically contacts Lambda for a status\. The Lambda function replies with a status, where the action execution is either successful, failed, or still in progress\. After 20 minutes, if the Lambda function has either sent no reply or replied that the action execution is still in progress, the CodePipeline action times out\. If the action times out, CodePipeline sets the Lambda invoke action state to failed\. Lambda has a separate timeout for Lambda functions that is not related to the CodePipeline action timeout\.  Amazon S3 deployment action: 20 minutes  If the upload to S3 times out during deployment of a large ZIP file, the action fails with a timeout error\. Try breaking up the ZIP file into smaller files\.  Step Functions invoke action: 7 days Custom actions: 24 hours All other actions: 1 hour  The Amazon ECS deployment action timeout is configurable up to one hour \(the default timeout\)\.   | 
|  Maximum number of total pipelines per Region in an AWS account  |  1000  Pipelines configured for either polling or event\-based change detection are counted toward this quota\.   | 
|  Maximum number of pipelines set to polling for source changes, per AWS Region  |  60  We recommend that you configure your pipeline to use event\-based change detection, such as webhooks or Amazon CloudWatch Events\. For more information, see [ Source actions and change detection methods](pipelines-about-starting.md#change-detection-methods)\.1   | 
| Maximum number of webhooks per Region in an AWS account | 300 | 
|  Number of custom actions per Region in an AWS account  |  50  | 
| 1Based on your source provider, use the following instructions to update your polling pipelines to use event\-based change detection:[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/limits.html) | 

The following quotas in AWS CodePipeline apply to Region availability, naming constraints, and allowed artifact sizes\. These quotas are fixed and cannot be changed\.

For information about structural requirements, see [CodePipeline pipeline structure reference](reference-pipeline-structure.md)\.


|  |  | 
| --- |--- |
|  AWS Regions where you can create a pipeline  |  US East \(Ohio\) US East \(N\. Virginia\) US West \(N\. California\) US West \(Oregon\) Canada \(Central\) Europe \(Frankfurt\) Europe \(Ireland\) Europe \(London\) Europe \(Milan\)\* Europe \(Paris\) Europe \(Stockholm\) Asia Pacific \(Hong Kong\)\* Asia Pacific \(Mumbai\) Asia Pacific \(Tokyo\) Asia Pacific \(Seoul\) Asia Pacific \(Singapore\) Asia Pacific \(Sydney\) South America \(São Paulo\) AWS GovCloud \(US\-West\)  | 
| Characters allowed in an action name |  Action names cannot exceed 100 characters\. Allowed characters include: Lowercase letters `a` through `z`, inclusive\. Uppercase letters `A` through `Z`, inclusive\. Numbers `0` through `9`, inclusive\. Special characters `.` \(period\), `@` \(at sign\), `-` \(minus sign\), and `_` \(underscore\)\. Any other characters, such as spaces, are not allowed\.  | 
| Characters allowed in action types |  Action type names cannot exceed 25 characters\. Allowed characters include: Lowercase letters a through z, inclusive\. Uppercase letters A through Z, inclusive\. Numbers 0 through 9, inclusive\. Special characters \. \(period\), @ \(at sign\), \- \(minus sign\), and \_ \(underscore\)\. Any other characters, such as spaces, are not allowed\.  | 
| Characters allowed in partner action names | Partner action names must follow the same naming conventions and restrictions as other action names in CodePipeline\. Specifically, they cannot exceed 100 characters\. Allowed characters include: Lowercase letters a through z, inclusive\.Uppercase letters A through Z, inclusive\.Numbers 0 through 9, inclusive\.Special characters \. \(period\), @ \(at sign\), \- \(minus sign\), and \_ \(underscore\)\.Any other characters, such as spaces, are not allowed\. | 
| Characters allowed in a pipeline name |  Pipeline names cannot exceed 100 characters\. Allowed characters include: Lowercase letters a through z, inclusive\. Uppercase letters A through Z, inclusive\. Numbers 0 through 9, inclusive\. Special characters \. \(period\), @ \(at sign\),\- \(minus sign\), and \_ \(underscore\)\. Any other characters, such as spaces, are not allowed\.   | 
| Characters allowed in a stage name |  Stage names cannot exceed 100 characters\. Allowed characters include: Lowercase letters a through z, inclusive\. Uppercase letters A through Z, inclusive\. Numbers 0 through 9, inclusive\. Special characters \. \(period\), @ \(at sign\), \- \(minus sign\), and \_ \(underscore\)\. Any other characters, such as spaces, are not allowed\.  | 
|  Maximum length of the action configuration key \(for example, the CodeBuild configuration keys are `ProjectName`, `PrimarySource`, and `EnvironmentVariables`\)  | 50 characters | 
|  Maximum length of the action configuration value \(for example, the value of the `RepositoryName` configuration in the CodeCommit action configuration should be less than 1000 characters:  `"RepositoryName": "my-repo-name-less-than-1000-characters"`\)  | 1000 characters | 
| Maximum number of actions per pipeline | 500 | 
| Maximum number of months that pipeline execution history information is retained | 12 | 
| Maximum number of parallel actions in a stage | 50 | 
| Maximum number of sequential actions in a stage | 50 | 
| Maximum size of artifacts in a source stage |  Artifacts stored in Amazon S3 buckets: 5 GB Artifacts stored in CodeCommit or GitHub repositories: 1 GB Exception: If you are using AWS Elastic Beanstalk to deploy applications, the maximum artifact size is always 512 MB\. Exception: If you are using AWS CloudFormation to deploy applications, the maximum artifact size is always 256 MB\. Exception: If you are using the `CodeDeployToECS` action to deploy applications, the maximum artifact size is always 3 MB\.  | 
|  Maximum size of the image definitions JSON file used in pipelines deploying Amazon ECS containers and images  | 100 KB | 
| Maximum size of input artifacts for AWS CloudFormation actions | 256 MB | 
| Maximum size of input artifacts for the CodeDeployToECS action | 3 MB | 
|  Maximum size of the JSON object that can be stored in the `ParameterOverrides` property  | For a CodePipeline deploy action with AWS CloudFormation as the provider, the ParameterOverrides property is used to store a JSON object that specifies values for the AWS CloudFormation template configuration file\. There is a maximum size limit of 1 kilobyte for the JSON object that can be stored in the ParameterOverrides property\. | 
|  Number of actions in a stage  |  Minimum of 1, maximum of 50  | 
| Number of artifacts allowed for each action | For the number of input and output artifacts allowed for each action, see the [Number of input and output artifacts for each action type](reference-pipeline-structure.md#reference-action-artifacts) | 
|  Number of stages in a pipeline  |  Minimum of 2, maximum of 50  | 
| Pipeline tags | Tags are case sensitive\. Maximum of 50 per resource\. | 
| Pipeline tag key names |  Any combination of Unicode letters, numbers, spaces, and allowed characters in UTF\-8 between 1 and 128 characters in length\. Allowed characters are \+ \- = \. \_ : / @ Tag key names must be unique, and each key can have only one value\. A tag cannot:  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/limits.html)  | 
| Pipeline tag values |  Any combination of Unicode letters, numbers, spaces, and allowed characters in UTF\-8 between 1 and 256 characters in length\. Allowed characters are \+ \- = \. \_ : / @ A key can have only one value, but many keys can have the same value\. A tag cannot: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/limits.html)  | 
| Uniqueness of names |  Within a single AWS account, each pipeline you create in an AWS Region must have a unique name\. You can reuse names for pipelines in different AWS Regions\.  Stage names must be unique within a pipeline\. Action names must be unique within a stage\.  | 
| Variable quotas |  There is a maximum size limit of 122880 bytes for all output variables combined for a particular action\. There is a maximum size limit of 100 KB for the total resolved action configuration for a particular action\. Output variable names are case sensitive\. Namespaces are case sensitive\. Allowed characters include: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/limits.html)  | 

\* You must enable this Region before you can use it\.