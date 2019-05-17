# Problem
The CloudFormation template doesn't support associating log group with KMS key for now.

# Solution
Create a custom resource in the template. Whenever the custom resource is created, updated or deleted, a Lambda function will be triggered. You can do whatever you want right there and then notify CloudFormation to go on or quit.

# Comments
* If you create the key through the KMS console, but not the CloudFormation template. You need to update the policy document manually, since the console doesn't allow adding Service as Principal.
* The alias of key is optional. I just like it to be readable. By the way, the `AliasName` needs to begin with 'alias/'.
* Normally, it's not a good idea to put inline code of Lambda function in the template. Since it's just a demo, I'd like to keep it simple. One extra benefit to do so is that I can directly use parameters, such as ${LogGroupName} and ${MyKey.Arn}.
* If anything exception occurs in the Lambda function, the CloudFormation gets stuck, because it doesn't receive any notification and doesn't know whether to go on or quit. When that happens, I have to delete the whole stack. Even I do that, the CloudFormation will get stuck in `DELETE_IN_PROGRESS` state again, because it needs to call the Lambda function to while deleting the custom resource. I don't know how long will it takes, but it will eventually switch to the `DELETE_FAILED` state, then you can delete it again. This time, it will be quick.
* When the stack is created successfully, you can use the CLI command `aws logs describe-log-groups` to verify the log groups.

# References
* [Custom Resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html)