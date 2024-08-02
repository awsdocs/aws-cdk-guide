# Set a CloudWatch alarm<a name="how_to_set_cw_alarm"></a>

Use the [aws\-cloudwatch](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_cloudwatch-readme.html) package to set up Amazon CloudWatch alarms on CloudWatch metrics\. You can use predefined metrics or create your own\.

**Topics**
+ [Use an existing metric](#how_to_set_cw_alarm_use_metric)
+ [Create your own metric](#how_to_set_cw_alarm_new_metric)
+ [Create the alarm](#how_to_set_cw_alarm_create)

## Use an existing metric<a name="how_to_set_cw_alarm_use_metric"></a>

Many AWS Construct Library modules let you set an alarm on an existing metric by passing the metric's name to a convenience method on an instance of an object that has metrics\. For example, given an Amazon SQS queue, you can get the metric **ApproximateNumberOfMessagesVisible** from the queue's [metric\(\)](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_sqs.Queue.html#metricmetricname-props) method:

------
#### [ TypeScript ]

```
const metric = queue.metric("ApproximateNumberOfMessagesVisible");
```

------
#### [ JavaScript ]

```
const metric = queue.metric("ApproximateNumberOfMessagesVisible");
```

------
#### [ Python ]

```
metric = queue.metric("ApproximateNumberOfMessagesVisible")
```

------
#### [ Java ]

```
Metric metric = queue.metric("ApproximateNumberOfMessagesVisible");
```

------
#### [ C\# ]

```
var metric = queue.Metric("ApproximateNumberOfMessagesVisible");
```

------

## Create your own metric<a name="how_to_set_cw_alarm_new_metric"></a>

Create your own [metric](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_cloudwatch.Metric.html) as follows, where the *namespace* value should be something like **AWS/SQS** for an Amazon SQS queue\. You also need to specify your metric's name and dimension:

------
#### [ TypeScript ]

```
const metric = new cloudwatch.Metric({
  namespace: 'MyNamespace',
  metricName: 'MyMetric',
  dimensionsMap: { MyDimension: 'MyDimensionValue' }
});
```

------
#### [ JavaScript ]

```
const metric = new cloudwatch.Metric({
  namespace: 'MyNamespace',
  metricName: 'MyMetric',
  dimensionsMap: { MyDimension: 'MyDimensionValue' }
});
```

------
#### [ Python ]

```
metric = cloudwatch.Metric(
    namespace="MyNamespace",
    metric_name="MyMetric",
    dimensionsMap=dict(MyDimension="MyDimensionValue")
)
```

------
#### [ Java ]

```
Metric metric = Metric.Builder.create()
        .namespace("MyNamespace")
        .metricName("MyMetric")
        .dimensionsMap(java.util.Map.of(    // Java 9 or later
            "MyDimension", "MyDimensionValue"))
        .build();
```

------
#### [ C\# ]

```
var metric = new Metric(this, "Metric", new MetricProps
{
    Namespace = "MyNamespace",
    MetricName = "MyMetric",
    Dimensions = new Dictionary<string, object>
    {
        { "MyDimension", "MyDimensionValue" }
    }
});
```

------

## Create the alarm<a name="how_to_set_cw_alarm_create"></a>

Once you have a metric, either an existing one or one you defined, you can create an alarm\. In this example, the alarm is raised when there are more than 100 of your metric in two of the last three evaluation periods\. You can use comparisons such as less\-than in your alarms via the `comparisonOperator` property\. Greater\-than\-or\-equal\-to is the AWS CDK default, so we don't need to specify it\.

------
#### [ TypeScript ]

```
const alarm = new cloudwatch.Alarm(this, 'Alarm', {
  metric: metric,
  threshold: 100,
  evaluationPeriods: 3,
  datapointsToAlarm: 2,
});
```

------
#### [ JavaScript ]

```
const alarm = new cloudwatch.Alarm(this, 'Alarm', {
  metric: metric,
  threshold: 100,
  evaluationPeriods: 3,
  datapointsToAlarm: 2
});
```

------
#### [ Python ]

```
alarm = cloudwatch.Alarm(self, "Alarm",
    metric=metric,
    threshold=100,
    evaluation_periods=3,
    datapoints_to_alarm=2
)
```

------
#### [ Java ]

```
import software.amazon.awscdk.services.cloudwatch.Alarm;
import software.amazon.awscdk.services.cloudwatch.Metric;

Alarm alarm = Alarm.Builder.create(this, "Alarm")
        .metric(metric)
        .threshold(100)
        .evaluationPeriods(3)
        .datapointsToAlarm(2).build();
```

------
#### [ C\# ]

```
var alarm = new Alarm(this, "Alarm", new AlarmProps
{
    Metric = metric,
    Threshold = 100,
    EvaluationPeriods = 3,
    DatapointsToAlarm = 2
});
```

------

An alternative way to create an alarm is using the metric's [createAlarm\(\)](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_cloudwatch.Metric.html#createwbralarmscope-id-props) method, which takes essentially the same properties as the `Alarm` constructor\. You don't need to pass in the metric, because it's already known\.

------
#### [ TypeScript ]

```
metric.createAlarm(this, 'Alarm', {
  threshold: 100,
  evaluationPeriods: 3,
  datapointsToAlarm: 2,
});
```

------
#### [ JavaScript ]

```
metric.createAlarm(this, 'Alarm', {
  threshold: 100,
  evaluationPeriods: 3,
  datapointsToAlarm: 2,
});
```

------
#### [ Python ]

```
metric.create_alarm(self, "Alarm",
    threshold=100,
    evaluation_periods=3,
    datapoints_to_alarm=2
)
```

------
#### [ Java ]

```
metric.createAlarm(this, "Alarm", new CreateAlarmOptions.Builder()
        .threshold(100)
        .evaluationPeriods(3)
        .datapointsToAlarm(2)
        .build());
```

------
#### [ C\# ]

```
metric.CreateAlarm(this, "Alarm", new CreateAlarmOptions
{
    Threshold = 100,
    EvaluationPeriods = 3,
    DatapointsToAlarm = 2
});
```

------