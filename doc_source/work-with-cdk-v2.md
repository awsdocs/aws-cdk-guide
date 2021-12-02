# Migrating to AWS CDK v2<a name="work-with-cdk-v2"></a>

Version 2 of the AWS CDK, now Generally Available, provides an improved development experience that aims to make Infrastructure as Code \(IAC\) even simpler\. See [Migrating to AWS CDK v2](../../v2/guide/migrating-v2.html) in the CDK v1 Developer Guide to learn how to migrate to CDK v2\.

CDK v1 will continue to be fully supported until June 2, 2022, at which time it will enter maintenance\. During the maintenance phase, CDK v1 will receive critical bug fixes and security patches only\. New features will be developed exclusively for CDK v2 during the v1 maintenance phase\. On June 2, 2023, support will end entirely for AWS CDK v1\. For more details, see [AWS CDK Maintenance Policy](https://github.com/aws/aws-cdk-rfcs/blob/master/text/0079-cdk-2.0.md#aws-cdk-maintenance-policy)\.

For information on migrating your apps to AWS CDK v2, see [Migrating to AWS CDK v2](../../v2/migrating_v2.html)\. 

## CDK Toolkit v2 compatibility<a name="work-with-cdk-v2-cli"></a>

CDK v2 requires v2 or later of the CDK Toolkit\. This version is backward\-compatible with CDK v1 apps, so you can use a single globally\-installed version of CDK Toolkit with all your AWS CDK projects, whether they use v1 or v2\. An exception is that CDK Toolkit v2 creates CDK v2 projects\.

If you need to create both v1 and v2 CDK projects, **do not install CDK Toolkit v2 globally\.** \(Remove it if you already have it installed: `npm remove -g aws-cdk`\.\) To invoke the CDK Toolkit, use npx to run v1 or v2 of the CDK Toolkit as desired\.

```
npx cdk init@1.x app --language typescript
npx cdk init@2.x app --language typescript
```

**Tip**  
Set up command line aliases so you can use cdk and cdk2 commands to invoke the desired version of the CDK Toolkit\.  

```
alias cdk=npx aws-cdk@1.x
alias cdk2=npx aws-cdk@2.x
```

```
doskey cdk=npx aws-cdk@1.x $*
doskey cdk2=npx aws-cdk@2.x $*
```