# Create an App with Multiple Stacks<a name="stack_how_to_create_multiple_stacks"></a>

The following example shows one solution to parameterizing how you create a stack\. It defines a property set that extends the `cdk.StackProps` class to add one additional property, `enc`, which specifies whether to set encryption on the Amazon S3 bucket in the stack\.

```
import core = require("@aws-cdk/core");
import s3 = require("@aws-cdk/aws-s3");

interface MyStackProps extends core.StackProps {
  enc: boolean;
}

export class MyStack extends core.Stack {
  constructor(scope: core.App, id: string, props: MyStackProps) {
    super(scope, id, props);

    if (props.enc) {
      new s3.Bucket(this, "MyGroovyBucket", {
        encryption: s3.BucketEncryption.KMS_MANAGED
      });
    } else {
      new s3.Bucket(this, "MyGroovyBucket");
    }
  }
}
```

The file `MyApp.ts` creates two stacks\. One with an unencrypted bucket in the `us-west-2` region; the other with an encrypted bucket in the `us-east-1` region\.

```
import core = require("@aws-cdk/core");
import { MyStack } from "../lib/MyApp-stack";

const app = new core.App();

new MyStack(app, "MyWestCdkStack", {
  env: {
    region: "us-west-2"
  },
  enc: false
});

new MyStack(app, "MyEastCdkStack", {
  env: {
    region: "us-east-1"
  },
  enc: true
});
```

The following example shows how to deploy a stack with an encrypted bucket to the `us-east-1` region\.

```
cdk deploy MyEastCdkStack
```

If you look at the stack using cdk synth MyEastCdkStack, you should see a bucket similar to the following:

```
  MyGroovyBucketFD9882AC:
    Type: AWS::S3::Bucket
    Properties:
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: aws:kms
```