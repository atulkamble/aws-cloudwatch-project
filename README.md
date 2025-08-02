# **AWS CloudWatch Monitoring Project**

## **Project Overview**

We will:

1. Launch an EC2 instance.
2. Create a CloudWatch Metric Alarm (CPU Utilization).
3. Create an SNS Topic for notifications.
4. Auto Trigger Alarm when CPU exceeds threshold.
5. (Optional) Automate with Terraform.

---

## **1. Pre-requisites**

* AWS CLI configured.
* IAM Role with **CloudWatchFullAccess**, **SNSFullAccess**, **EC2FullAccess**.
* EC2 Instance running.

---

## **2. Project Structure**

```
cloudwatch-monitoring-project/
│
├── cloudwatch_alarm.sh
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
└── README.md
```

---

## **3. CLI Script (cloudwatch\_alarm.sh)**

```bash
#!/bin/bash

INSTANCE_ID="i-xxxxxxxxxxxxxx"  # Update your EC2 Instance ID
SNS_TOPIC_NAME="CPUAlarmTopic"

# Create SNS Topic
aws sns create-topic --name $SNS_TOPIC_NAME

# Get Topic ARN
TOPIC_ARN=$(aws sns list-topics --query "Topics[?contains(TopicArn, '$SNS_TOPIC_NAME')].TopicArn" --output text)

# Subscribe Email to SNS Topic
aws sns subscribe --topic-arn $TOPIC_ARN --protocol email --notification-endpoint your-email@example.com

# Create CloudWatch Alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "EC2-High-CPU-Alarm" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 70 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=$INSTANCE_ID \
  --evaluation-periods 2 \
  --alarm-actions $TOPIC_ARN \
  --ok-actions $TOPIC_ARN \
  --unit Percent

echo "Alarm created and SNS subscription initiated. Please confirm the email subscription."
```

---

## **4. Terraform Automation**

### **terraform/main.tf**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "my_ec2" {
  ami           = "ami-0c02fb55956c7d316"  # Amazon Linux 2 AMI
  instance_type = "t2.micro"
}

resource "aws_sns_topic" "cpu_alarm_topic" {
  name = "CPUAlarmTopic"
}

resource "aws_sns_topic_subscription" "email_subscription" {
  topic_arn = aws_sns_topic.cpu_alarm_topic.arn
  protocol  = "email"
  endpoint  = "your-email@example.com"
}

resource "aws_cloudwatch_metric_alarm" "cpu_high" {
  alarm_name                = "EC2-High-CPU-Alarm"
  comparison_operator       = "GreaterThanThreshold"
  evaluation_periods        = "2"
  metric_name               = "CPUUtilization"
  namespace                 = "AWS/EC2"
  period                    = "300"
  statistic                 = "Average"
  threshold                 = "70"
  alarm_description         = "Alarm when server CPU exceeds 70%"
  dimensions = {
    InstanceId = aws_instance.my_ec2.id
  }
  alarm_actions = [aws_sns_topic.cpu_alarm_topic.arn]
  ok_actions    = [aws_sns_topic.cpu_alarm_topic.arn]
}
```

### **terraform/variables.tf**

```hcl
variable "instance_type" {
  default = "t2.micro"
}

variable "email_address" {
  default = "your-email@example.com"
}
```

### **terraform/outputs.tf**

```hcl
output "instance_id" {
  value = aws_instance.my_ec2.id
}

output "sns_topic_arn" {
  value = aws_sns_topic.cpu_alarm_topic.arn
}
```

---

## **5. Deploy Project Steps**

### **Manual Deployment (Bash Script)**

```bash
chmod +x cloudwatch_alarm.sh
./cloudwatch_alarm.sh
```

### **Terraform Deployment**

```bash
cd terraform
terraform init
terraform apply -auto-approve
```

---

## **6. Testing the Alarm**

* SSH into EC2:

  ```bash
  sudo stress --cpu 4 --timeout 300
  ```
* This will spike CPU Utilization.
* You’ll receive an **email alert** when the alarm triggers.

---

## **7. Cleanup Resources**

```bash
terraform destroy -auto-approve
```

Or delete manually via CLI/Console.

---

## **8. README.md**

```markdown
# AWS CloudWatch EC2 Monitoring Project

## Project Overview
Monitor EC2 instance CPU utilization using AWS CloudWatch with alarms and email notifications via SNS.

## Steps:
1. Launch EC2 Instance.
2. Create CloudWatch Alarm.
3. Configure SNS Topic & Subscription.
4. Test high CPU load.
5. Receive email alerts.

## Deployment Methods:
- Bash Script: `./cloudwatch_alarm.sh`
- Terraform: `terraform apply`

## Cleanup:
- Bash: Delete Alarm & SNS manually.
- Terraform: `terraform destroy`
```

---

### ✅ **Shall I create a ready-to-clone GitHub Repo structure for this project?** (with all files organized)
