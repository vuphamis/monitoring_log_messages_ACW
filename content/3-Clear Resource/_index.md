---
title: "Clear resource"
date: "`r Sys.Date()`"
weight: 4
chapter : false
pre: " <b> 3. </b> "
---

**Clear Amazon EC2**

Terminate Instance

    aws ec2 describe-instances --filters 'Name=tag:Name,Values=workshopEC2Instance' --query 'Reservations[].Instances[].InstanceId' --output text | select-object {aws ec2 terminate-instances --instance-ids $_}

Delete security group

    aws ec2 describe-security-groups --filters 'Name=group-name,Values=workshopSecurityGroup' --query 'SecurityGroups[].GroupId'  --output text | select-object {aws ec2 delete-security-group --group-id $_}

Delete keypair

    aws ec2 delete-key-pair --key-name 'workshopKeyPair'

**Clear Amazon IAM**

Remove role from instance profile

    aws iam remove-role-from-instance-profile --instance-profile-name 'CWAServerProfile' --role-name 'CWAServer'

Delete instance profile

    aws iam delete-instance-profile --instance-profile-name 'CWAServerProfile'

Detach policy from role

    aws iam list-policies --query "Policies[?PolicyName=='CloudWatchAgentServerPolicy'].Arn" --output text | select-object {aws iam detach-role-policy --role-name 'CWAServer' --policy-arn $_}

Delete role

    aws iam delete-role --role-name 'CWAServer'

**Clear Amazon CloudWatch**

Delete log group

    aws logs delete-log-group --log-group-name 'error_log'

Delete metric alarm

    aws cloudwatch delete-alarms --alarm-name 'WarningAlarm'

**Clear Amazon SNS**

Delete topic

    aws sns list-topics --query "Topics[?contains(TopicArn,'workshopTopic')].TopicArn" --output text | select-object {aws sns delete-topic --topic-arn $_}