--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Set a CloudWatch Alarm<a name="how_to_set_cw_alarm"></a>

The **aws\-cloudwatch** package supports setting CloudWatch alarms on CloudWatch metrics\. The syntax is as follows, where *METRIC* is a CloudWatch metric you have created, and the alarm is raised there are more than 100 of the measured metrics in two of the last three seconds:

```
new cloudwatch.Alarm(this, 'Alarm', {
  metric: METRIC,
  threshold: 100,
  evaluationPeriods: 3,
  datapointsToAlarm: 2,
});
```

The syntax for creating a metric for a AWS CloudFormation Resource is as follows, where the *namespace* value should be something like **AWS/SQS** for an Amazon SQS queue\.

```
const metric = new cloudwatch.Metric({
  namespace: 'MyNamespace',
  metricName: 'MyMetric',
  dimensions: { MyDimension: 'MyDimensionValue' }
});
```

Many CDK packages contain an AWS Construct Library construct with functionality to enable setting an alarm based on an existing metric\. For example, you can create an Amazon SQS alarm for the **ApproximateNumberOfMessagesVisible** metric that raises an alarm if the queue has more than 100 messages available for retrieval in two of the last three seconds\.

```
const qMetric = queue.metric('ApproximateNumberOfMessagesVisible');

new cloudwatch.Alarm(this, 'Alarm', {
  metric: qMetric,
  threshold: 100,
  evaluationPeriods: 3,
  datapointsToAlarm: 2,
});
```