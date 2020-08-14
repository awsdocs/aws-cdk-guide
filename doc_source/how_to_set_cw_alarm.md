# Set a CloudWatch alarm<a name="how_to_set_cw_alarm"></a>

The **aws\-cloudwatch** package supports setting CloudWatch alarms on CloudWatch metrics\. So the first thing you need is a metric\. You can use a predefined metric or you can create your own\. Create your own metric as follows, where the *namespace* value should be something like **AWS/SQS** for an Amazon SQS queue\.

------
#### [ TypeScript ]

```
const metric = new cloudwatch.Metric({
  namespace: 'MyNamespace',
  metricName: 'MyMetric',
  dimensions: { MyDimension: 'MyDimensionValue' }
});
```

------
#### [ JavaScript ]

```
const metric = new cloudwatch.Metric({
  namespace: 'MyNamespace',
  metricName: 'MyMetric',
  dimensions: { MyDimension: 'MyDimensionValue' }
});
```

------
#### [ Python ]

```
metric = cloudwatch.Metric(
    namespace="MyNamespace",
    metric_name="MyMetric",
    dimensions=dict(MyDimension="MyDimensionValue")
)
```

------
#### [ Java ]

```
Metric metric = Metric.Builder.create()
        .namespace("MyNamespace")
        .metricName("MyMetric")
        .dimensions(new HashMap<String, Object>() {{
            put("MyDimension", "MyDimensionValue");
        }}).build();
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

Once you have a metric, either an existing one or one you defined, you can create an alarm\. In this case, the alarm is raised when there are more than 100 of your metric in two of the last three seconds:

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

An alternative way to create an alarm is using the metric's `createAlarm()` method, which takes essentially the same properties as the `Alarm` constructor; you just don't need to pass in the metric, since it's already known\.

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
metric.create_alarm(this, "Alarm",
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

Many AWS Construct Library modules let you set an alarm on an existing metric\. For example, you can create an Amazon SQS alarm for the **ApproximateNumberOfMessagesVisible** metric that raises an alarm if the queue has more than 100 messages available for retrieval in two of the last three seconds\.

------
#### [ TypeScript ]

```
    const qMetric = queue.metric("ApproximateNumberOfMessagesVisible");

    new cloudwatch.Alarm(this, "Alarm", {
      metric: qMetric,
      threshold: 100,
      evaluationPeriods: 3,
      datapointsToAlarm: 2
    });
```

------
#### [ JavaScript ]

```
    const qMetric = queue.metric("ApproximateNumberOfMessagesVisible");

    new cloudwatch.Alarm(this, "Alarm", {
      metric: qMetric,
      threshold: 100,
      evaluationPeriods: 3,
      datapointsToAlarm: 2
    });
```

------
#### [ Python ]

```
q_metric = queue.metric("ApproximateNumberOfMessagesVisible")
        
cloudwatch.Alarm(self, "Alarm",
    metric=q_metric,
    threshold=100,
    evaluation_periods=3,
    datapoints_to_alarm=2
)
```

------
#### [ Java ]

```
Metric qMetric = queue.metric("ApproximateNumberOfMessagesVisible");

Alarm.Builder.create(this, "Alarm")
        .metric(qMetric)
        .threshold(100)
        .evaluationPeriods(3)
        .datapointsToAlarm(2).build();
```

------
#### [ C\# ]

```
var qMetric = queue.Metric("ApproximateNumberOfMessagesVisible");

new Alarm(this, "Alarm", new AlarmProps {
    Metric = qMetric,
    Threshold = 100,
    EvaluationPeriods = 3,
    DatapointsToAlarm = 2
});
```

------