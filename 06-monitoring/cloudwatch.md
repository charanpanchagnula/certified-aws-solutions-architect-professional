# CloudWatch

- Provides services to ingest, store and manage metrics
- It is a public service - provides public space endpoints
- Many services have native management plane integration with CloudWatch, for example EC2. Also, EC2 provides external gathered information only, for metrics from inside an EC2 we can use CloudWatch agent
- CloudWatch can be used from on-premises using the agent or the CloudWatch API
- CloudWatch stores data in a persistent way
- Data can be viewed from the console, CLI or API, but also CloudWatch also provides dashboards and anomaly detection
- CloudWatch Alarms: react to metrics, can be used to notify or to perform actions
- Instances in public subnet can connect to cloudwatch using Internet gateway and the instances in the private subnets can connect to the cloudwatch using Interface Endpoint.
- CloudWatch sits in the AWS Public zone and it stores metrics and various services publish data into CW.

## CloudWatch - Data

- **Namespace**: container for metrics e.g. AWS/EC2 for EC2 NS, AW/Lambda for lambda NS. Its possible to have same metric name for different services, NS helps to segregate them.
- **Data point**: timestamp, value, unit of measure (optional)
- **Metric**: time ordered set of data points. Example of builtin metrics: `CPUUtilization`, `NetworkIn`, `DiskWriteBytes` for EC2
- Every metric has a `MetricName` and a namespace e.g. CPUUtilization for AWS/EC2
- **Dimension**: name/value pair, example: a dimension is the way for `CPUUtilization` metric to be separated from one instance to another. You provide namespace, metric name and zero or more dimensions. Example: AWS/EC2 CPUUtilization Name=InstanceId, Value=i-11111111(cat), AWS/EC2 CPUUtilization Name=InstanceId, Value=i-222222222(dog)
- Dimensions can be used to aggregate data, example aggregate data for all instances for an ASG.
- **Resolution**: standard (60 second granularity) or high (1 second granularity). This means for CPUUtilization, if you published 42% with a 60 second resolution, then it would be viewed as having 42% for that entire 60 second duration.
- **Metric resolution**: minimum period that we can get one particular data point for
- Data retention:
    - sub 60s granularity is retained for 3 hours
    - High resolution can be measured but they cost more. Resolution determines the minimum period which can be specified and get a valid value. Standard (60 Sec) .. High (1 Sec)
    - 60s granularity retained for 15 days
    - 5 min retained for 63 days
    - 1 hour retained for 455 days
- As data ages, its aggregated and stored for longer period of time with less resolution
- Statistics: get data over a period and aggregate it in a certain way (ex Min, Max, Sum, Average)
- Percentile: relative standing of a value within the dataset (p95, p99)

## CloudWatch Alarms

- Alarm: watches a metric over a period of time
- States: `ALARM` or `OK` based on the value of a metric against a threshold over time
- Alarms can be configured with one or more actions, which can initiate actions on our behalf. Actions can be: send notification to an SNS topic, attempt an auto scaling policy modification or use Event Bridge to integrate with other services
- High resolution metrics can have high resolution alarms

## CloudWatch Logs

- CloudWatch Logs provides two type of functionalities: ingestion and subscription
- CloudWatch Logs is a public service designed to store, monitor and provide access logging data
- Can provide logging ingestion for AWS products natively, but also for on-premises, IOT or any application
- CloudWatch Agent: used to provide ingestion for custom applications
- CloudWatch can also ingest log streams from VPC Flow Logs or CloudTrail (account events and AWS API calls)
- CloudWatch Logs is regional service, but certain global services send their logs to `us-east-1`
- Log events consist of 2 parts:
    - **Timestamp**
    - **Raw message**
- Log Events can be collected into Log Streams. Log Streams are sequence of log events sharing the same source
- Log Groups: are collection of Log Streams. We can set retention, permissions and encryption on the log groups. By default log groups store data indefinitely
- Metric Filter: can be defined on the log group and will look for pattern in the log events. Essentially creates a metric from the log streams by looking at occurrences of certain patterns defined by us (example: failed SSH logs in events). From this metric, alarms etc can be created
- Export logs from CloudWatch:
    - **S3 Export**: we can create an export task (`Create-Export-Task`) which will take up to 12 hours. Its not real time. It is encrypted using SSE-S3.
    - **Subscription**: deliver logs in real time. This is done at a per log group basis. We should create a subscription filter for the following destination: Kinesis Data Firehose (near real time) and then to S3 or any firehose destinations, OpenSearch (ElasticSearch) using Lambda or custom Lambda, Kinesis Data Streams (any KCL consumer)
- Subscription filters can be used to create a logging aggregation architecture

## CloudWatch Dashboards

- A great way to setup dashboards for quick access to key metrics
- Dashboards are global
- Dashboards can include graphs from different regions
- We can change the time zone and time range of the dashboards
- We can setup automatic refresh (10s, 1m, 2m, 5m, 15m)
- Pricing:
    - 3 dashboards (up to 50 metrics) for free
    - $3/dashboard/month

## CloudWatch Synthetics Canary

- Synthetics Canary are configurable scripts that will monitor APIs and URLs
- These scripts meant to reproduce what a customer would do in order to find issues before the app is deployed to production
- They can be also used to check the availability and latency of our endpoints
- They can store load time data and screenshots of the UI
- They have integration with CloudWatch Alarms
- The scripts can be written in Node.js or Python
- Provides programmatic access to a headless Chrome browser
- They can be run once or on a regular basis
- Canary Blueprints:
    - Heartbeat Monitor: load URL, store screenshot and an HTTP archive file
    - API Canary: test basic read and write functions of a REST API
    - Broken Link Checker: check all links inside a page
    - Visual Monitoring: compare a screenshot taken during a canary run with a baseline screenshot
    - Canary Recorder: used with CloudWatch Synthetics Recorder - used to record actions on a website and automatically generate a test script for that
    - GUI Workflow Builder: verifies that actions can be taken on a webpage (example: test a webpage with a login form)