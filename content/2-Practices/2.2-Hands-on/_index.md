---
title: "Hands-on"
date: "`r Sys.Date()`"
weight: 5
chapter : false
pre: " <b> 2.2 </b> "
---

#### Configure AWS CLI

    aws configure

![](/images/awscli-2.png?width=90pc)

#### Create Keypair

**Create key pair for ec2.**

    aws ec2 create-key-pair --key-name 'workshopKeyPair' --query 'KeyMaterial' --output 'text' | Out-file ./workshopKeyPair.pem

![](/images/aws-keypair-0.png?width=90pc)

**Check key pair**

    aws ec2 describe-key-pairs --filters 'Name=key-name,Values=workshopKeyPair'--output 'table'

![](/images/aws-keypair-1.png?width=90pc)

#### Security group

**Create security group**

    aws ec2 create-security-group --group-name 'workshopSecurityGroup' --description 'Security Group for FCJ-Workshop'

![](/images/aws-securitygroup-0.png?width=90pc)

**Bind security group to environment variable**

    $SecurityGroupID=aws ec2 describe-security-groups --filters 'Name=group-name,Values=workshopSecurityGroup' --query 'SecurityGroups[].GroupID' --output 'text'

![](/images/aws-securitygroup-1.png?width=90pc)

**Add inbound rule**

    aws ec2 authorize-security-group-ingress --group-id $SecurityGroupID --protocol 'tcp' --port '22' --cidr '0.0.0.0/0'
    aws ec2 authorize-security-group-ingress --group-id $SecurityGroupID --protocol 'tcp' --port '443' --cidr '0.0.0.0/0'

![](/images/aws-securitygroup-2.png?width=90pc)

#### Launch EC2 Instance

**Bind SubnetID, ImageId and mapping device to environment variable**

{{% notice info %}}
Linux distro is **SUSE Linux Enterprise Server 15 SP6** \
Way to find image-id [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html)
{{% /notice %}}

    $SubnetID=aws ec2 describe-subnets --filters 'Name=availabilityZone,Values=ap-southeast-1a' --query 'Subnets[].SubnetId' --output 'text'
    $ImageID='ami-0945845b39d75e25f'
    $mapping=@{
 	'DeviceName' = '/dev/xvda'
 	'Ebs' = @{
 		'VolumeSize' = '10'
		'VolumeType' = 'gp3'
		}
 	}

![](/images/aws-subnet-0.png?width=90pc)
![](/images/aws-image-0.png?width=90pc)
![](/images/aws-mappingdevice-0.png?width=90pc)

**Launch EC2 Instance**

    ConvertTo-Json $mapping | aws ec2 run-instances `
        --image-id $ImageID `
        --count '1' `
        --instance-type 't2.micro' `
        --key-name 'workshopKeyPair' `
        --security-group-ids $SecurityGroupID `
        --subnet-id $SubnetID `
        --associate-public-ip-address `
        --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=workshopEC2Instance}]' `
        --query 'Instances[0].InstanceId'`
        --block-device-mappings $_

![](/images/aws-launchinstance-0.png?width=90pc)

#### Add profile to Instance

**Create role policy document file**

    $rolePolicyDocument=[PSCustomObject]@{
 	'Version'='2012-10-17'
 	'Statement'=@(@{
 		'Effect'='Allow'
 		'Principal'=@{
 			'Service'='ec2.amazonaws.com'
 			}
 		'Action'='sts:AssumeRole'
 		})
 	} | ConvertTo-Json -Depth 3| Out-file -Encoding ASCII -FilePath ./rolePolicyDocument.json

**Create role**

    aws iam create-role --role-name 'CWAServer' --assume-role-policy-document file://rolePolicyDocument.json

![](/images/aws-createrole-0.png?width=90pc)

**Add profile to instance**

Find CloudWatchAgentServerPolicy arn

    $PolicyARN = aws iam list-policies --query "Policies[?PolicyName=='CloudWatchAgentServerPolicy'].Arn" --output text | write-host

Attach policy to role

	aws iam attach-role-policy --role-name 'CWAServer' --policy-arn $PolicyArn

Create IntanceProfile

	aws iam create-instance-profile --instance-profile-name 'CWAServerProfile'

Add role for instance profile

    aws iam add-role-to-instance-profile --instance-profile-name 'CWAServerProfile' --role-name 'CWAServer'

Find InstanceId and bind to environment variable

    $InstanceId = aws ec2 describe-instances --filters 'Name=tag:Name,Values=workshopEC2Instance' --query 'Reservations[].Instances[].InstanceId' --output 'text'

Attach role to Instance Profile

	aws ec2 associate-iam-instance-profile --instance-id $InstanceId --iam-instance-profile Name="CWAServerProfile"

![](/images/aws-attachprofile-0.png?width=90pc)

#### Setup and configure EC2

**Connect to EC2 instance with ssh**

Get a public ip address

    aws ec2 describe-instances --filters 'Name=tag:Name,Values=workshopEC2Instance' --query 'Reservations[].Instances[].PublicIpAddress' --output 'text'

Connect ssh

    ssh -i .\workshopKeyPair.pem ec2-user@<Public-ip-address>

**Install Amazone CloudWatch Agent**

Update package

    sudo zypper update

![](/images/aws-update-0.png?width=90pc)

Install collectd

    sudo zipper install collectd

![](/images/aws-installcollectd-0.png?width=90pc)

Install apache2

	sudo zipper install apache2

![](/images/aws-installapache2-0.png?width=90pc)

Install CloudWatchAgent

    wget https://amazoncloudwatch-agent-ap-southeast-1.s3.ap-southeast-1.amazonaws.com/suse/amd64/latest/amazon-cloudwatch-agent.rpm
    sudo rpm -UvH ./amazon-cloudwatch-agent.rpm

![](/images/aws-installcloudwatchagent-0.png?width=90pc)

**Setup Amazon CloudWatch Agent**

Start web service

    sudo systemctl start apache2
    sudo cat /var/log/apache2/error_log

![](/images/aws-startapache2-0.png?width=90pc)

Setup amazon coudwatch agent with wizard

    sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

![](/images/aws-cwas-0.png?width=90pc)

Run amazone cloudwatch agent service

    sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
	sudo systemctl start amazon-cloudwatch-agent.service

![](/images/aws-runcwas.png?width=90pc)

#### Create Amazone SNS topic

**Create SNS topic**

    aws sns create-topic --name 'workshopTopic' --region 'ap-southeast-1'

![](/images/aws-createsnstopic-0.png?width=90pc)

**Add subcription**

Add subcription

    aws sns subscribe --topic-arn 'arn:aws:sns:ap-southeast-1:316323671320:workshopTopic' --protocol 'email' --notification-endpoint 'unknownvfs@gmail.com'

![](/images/aws-addsubcription-0.png?width=90pc)

Confirm subcription

![](/images/aws-mailconfirm-0.png?width=70pc)
![](/images/aws-confirmcomplete-0.png?width=40pc)

Recheck subcriptions

    aws sns list-subscriptions-by-topic --topic-arn 'arn:aws:sns:ap-southeast-1:316323671320:workshopTopic'

**Create CloudWatch metric**

Create a cloudwatch metric

    aws logs put-metric-filter --log-group-name 'error_log' --filter-name 'SSLWarnFilter' --filter-pattern '"[ssl:warn]"' --metric-transformations 'metricName=SSLWarning,metricNamespace=test,metricValue=1,defaultValue=0,unit=None'
	aws logs describe-metric-filters

![](/images/aws-createmetric-0.png?width=90pc)

Create a cloudwatch alarm

    ws cloudwatch put-metric-alarm --alarm-name 'WarningAlarm' --alarm-description 'ssl:warn missconfigure detection' --alarm-action 'arn:aws:sns:ap-southeast-1:316323671320:workshopTopic' --comparison-operator 'GreaterThanThreshold' --threshold '0' --evaluation-periods '1' --metric-name 'SSLWarning' --period '60' --namespace 'test' --statistic 'Sum' --treat-missing-data 'notBreaching'
	aws cloudwatch describe-alarms

![](/images/aws-createmetricalarm-0.png?width=90pc)