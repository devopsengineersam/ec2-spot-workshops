+++
title = "Configure Dynamic scaling"
weight = 150
+++

Now the predictive scaling policy is in place, you notice that the auto scaling group scales out successfully. However, you notice that the auto scaling group doesn't scale in when the predicted capacity is less than the actual capacity.

## Use predictive scaling with dynamic scaling

{{% notice info %}}
When using predictive scaling, Amazon EC2 Auto Scaling scales the number of instances at the **beginning of each hour**. Scales out if actual capacity is less than the predicted, **however** if actual capacity is greater than the predicted capacity, EC2 Auto Scaling **doesn't scale in** capacity. That's why we need to combine predictive scaling with another scaling policy to scale in capacity when it's not needed.
{{% /notice %}}

**Dynamic scaling** is used to automatically scale capacity in response to real-time changes in resource utilization. Using it with predictive scaling helps you follow the demand curve for your application closely, **scaling in** during periods of low traffic and scaling out when traffic is higher than expected.

{{% notice note %}}
When **multiple** scaling policies are active, each policy determines the desired capacity independently, and the desired capacity is set to the maximum of those.
{{% /notice %}}

Now you are going to configure the Auto Scaling group to automatically scale out and scale in as your **application load** fluctuates. We will be using a predefined metric that uses average CPU utilization across the Auto Scaling group instances.


1. Review this command to understand the options, then go ahead and run it 

```
cat <<EoF > asg-automatic-scaling.json
{
  "AutoScalingGroupName": "ec2-workshop-asg",
  "PolicyName": "automaticScaling",
  "PolicyType": "TargetTrackingScaling",
  "EstimatedInstanceWarmup": 300,
  "TargetTrackingConfiguration": {
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 75,
    "DisableScaleIn": false
  }
}
EoF
```

2. Go ahead and apply the scaling policy:

```
aws autoscaling put-scaling-policy --cli-input-json file://asg-automatic-scaling.json
```

Command should return policy ARN and target tracking alarms that have been created.

```json
{
    "PolicyARN": "arn:aws:autoscaling:ap-southeast-2:115751184547:scalingPolicy:04b7b7eb-6d65-40fb-946d-e5d2a1a55747:autoScalingGroupName/ec2-workshop-asg:policyName/automaticScaling",
    "Alarms": [
        {
            "AlarmName": "TargetTracking-ec2-workshop-asg-AlarmHigh-6c60b9c6-b7e8-4fcf-9733-d9c390754b99",
            "AlarmARN": "arn:aws:cloudwatch:ap-southeast-2:115751184547:alarm:TargetTracking-ec2-workshop-asg-AlarmHigh-6c60b9c6-b7e8-4fcf-9733-d9c390754b99"
        },
        {
            "AlarmName": "TargetTracking-ec2-workshop-asg-AlarmLow-70cbbd68-5540-4293-a4c5-3ab2d8aa72bb",
            "AlarmARN": "arn:aws:cloudwatch:ap-southeast-2:115751184547:alarm:TargetTracking-ec2-workshop-asg-AlarmLow-70cbbd68-5540-4293-a4c5-3ab2d8aa72bb"
        }
    ]
}
```

3. Navigate to the [Auto Scaling console](https://console.aws.amazon.com/ec2/autoscaling/home#AutoScalingGroups:view=details) and check out your newly created scaling policy in the **Scaling Policies** tab.
4. Now you have predictive and dynamic scaling polices in place, you have successfully completed the first task you've been asked to do.