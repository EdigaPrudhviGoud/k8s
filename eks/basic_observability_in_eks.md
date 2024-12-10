Metrics: Time series measurement of system performance,health or business data over a period. typically visualized on graphical charts
Logs: Textual data captured from applications indicating an event happended at that time.
Traces: the record of end-to-end journey of user requests

EKS console and control plane logs:
eks console
Enable control plane logging
explore control plane logs in cloudwatch
cloudwatch insights

control plane log types(Logging in aws console):
api
controlmanager
scheduler
audit-> provide a logs the changes in cluster done by user,administrator or system components
authenticator->these logs represent control plane component that eks uses for k8s RBAC authentication using IAM cred's

Fluentbit log aggregator sends logs of pods on the nodes to cloud watch log group
Pod logs through cloud watch logs
Cluster metrics through container insights

AWS distro for opentelemetry collector-ADOT(pod in eks cluster) -> prometheus -> grafana
AWS distro for opentelemetry collector-ADOT(pod in eks cluster) -> AWS X-RAY
-----------------------------------------------------------------------------------------------------------------------------
By default, AWS EKS does not set up AWS Distro for OpenTelemetry or X-Ray for you. You need to manually deploy and configure the OpenTelemetry Collector and set it up to export traces to AWS X-Ray. However, AWS provides tools like Helm charts to simplify this process.
Steps to Set Up ADOT with AWS X-Ray in EKS:
To integrate OpenTelemetry with AWS X-Ray in EKS, you generally follow these steps:

Install ADOT (AWS Distro for OpenTelemetry) in your EKS Cluster:

You can deploy the OpenTelemetry Collector in your EKS cluster using Helm or directly as a Kubernetes deployment.
ADOT supports both tracing (via AWS X-Ray) and metrics (via Amazon CloudWatch).
Configure the OpenTelemetry Collector:

After deploying the OpenTelemetry Collector, you need to configure it to send trace data to AWS X-Ray.
This involves setting up the OpenTelemetry Collector's configuration to export traces to AWS X-Ray, which requires setting specific endpoint and authentication configurations.
Modify Your Application to Emit Traces:

If your application isn't already instrumented with OpenTelemetry, you need to instrument it so that traces are collected.
OpenTelemetry SDKs are available for many languages, and you can integrate them into your application code.
Verify the Integration:

After deploying the OpenTelemetry Collector and configuring it to export data to X-Ray, you can verify the traces in the AWS X-Ray Console.
