# ASG: Auto Scaling Group

In real-life, the load on your websites and application can change. You can create and get rid of servers very quickly

* The goal of an Auto Scaling Group (ASG) is to:
  * **Scale out** (add EC2 instances) to match an increased load
  * **Scale in** (remove EC2 instances) to match a decreased load
  * Ensure we have a minimum and a maximum number of machines running
  * Automatically Register new instances to a load balancer

**ASGs have the following attributes**

* A launch configuration
  * AMI + InstanceType
  * EC2 User Data
  * EBSVolumes
  * Security Groups
  * SSH Key Pair
* Min Size / Max Size / Initial Capacity (Actual Size / Desired Capacity)
* Network + Subnets Information
* Load Balancer Information
* Scaling Policies

**Auto Scaling Alarms**

* It is possible to scale an ASG based on CloudWatch alarms * An Alarm monitors a metric (such as Average CPU)
* Metrics are computed for the overall ASG instances
* Based on the alarm:
  * We can create scale-out policies (increase the number of instances)
  * We can create scale-in policies (decrease the number of instances)

**Auto Scaling New Rules**

* It is now possible to define ”better” auto scaling rules that are directly managed by EC2
  * Target Average CPU Usage
  * Number of requests on the ELB per instance
  * Average Network In
  * Average Network Out
* These rules are easier to set up and can make more sense

**Auto Scaling Custom Metric**

* We can auto scale based on a custom metric (ex: number of connected users)
* Send custom metric from application on EC2 to CloudWatch (PutMetric API)
* Create CloudWatch alarm to react to low / high values
* Use the CloudWatch alarm as the scaling policy for ASG

**ASG Summary**

* Scaling policies can be on CPU, Network... and can even be on custom metrics or based on a schedule (if you know your visitors patterns)
* ASGs use Launch configurations or Launch Templates (newer)
* To update an ASG, you must provide a new launch configuration / launch template
* IAM roles attached to an ASG will get assigned to EC2 instances
* ASG are free.You pay for the underlying resources being launched
* Having instances under an ASG means that if they get terminated for whatever reason, the ASG will automatically create new ones as a replacement. Extra safety!
* ASG can terminate instances marked as unhealthy by an LB (and hence replace them)

**Auto Scaling Groups – Dynamic Scaling Policies**

* Target Tracking Scaling
  * Most simple and easy to set-up
  * Example: I want the average ASG CPU to stay at around 40%
* Simple / Step Scaling
  * When a CloudWatch alarm is triggered (example CPU > 70%), then add 2 units
  * When a CloudWatch alarm is triggered (example CPU < 30%), then remove 1
* Scheduled Actions
  * Anticipate a scaling based on known usage patterns
  * Example: increase the min capacity to 10 at 5 pm on Fridays
* Predictive scaling
  * continuously forecast load and schedule scaling ahead

**Good metrics to scale on**

* **CPUUtilization**: Average CPU utilization across your instances
* **RequestCountPerTarget**: to make sure the number of requests per EC2 instances is stable
* **Average Network In / Out** (if you’re application is network bound)
* **Any custom metric** (that you push using CloudWatch)

**Auto Scaling Groups - Scaling Cooldowns**

* After a scaling activity happens, you are in the **cooldown period (default 300 seconds)**
* During the cooldown period, the ASG will not launch or terminate additional instances (to allow for metrics to stabilize)
* Advice: Use a ready-to-use AMI to reduce configuration time in order to be serving request fasters and reduce the cooldown period
