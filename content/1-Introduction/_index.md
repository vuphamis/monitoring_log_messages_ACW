---
title: "Introduction"
date: "`r Sys.Date()`"
weight: 2
chapter : false
pre: " <b> 1. </b> "
---

#### Introdution

**Monitoring log messages** is a crucial aspect of information security. It provides valuable insights into system behavior, helps detect potential security threats, and allows for timely response to incidents. By analyzing log data, security teams can identify unusual patterns, track suspicious activities, and gain a better understanding of the overall security posture of their systems. Log monitoring also aids in compliance with various industry regulations and standards that require organizations to maintain audit trails and evidence of security measures.

**In this workshop**, we'll be looking at a system with a web server that's been misconfigured. We'll be monitoring an error log for the service using Amazon CloudWatch and sending an alarm to the user via an Amazone SNS service triggered by an Amazon CloudWatch Alarm.

#### Amazon Simple Notifications Service

**Amazon SNS** is a managed messaging service that lets you send notifications to lots of different devices and platforms. Amazon SNS was introduced in 2009 and has come a long way since then. It started out as a tool for mobile app notifications but has since expanded to support a wide range of use cases, including email alerts, SMS messages, and integration with other AWS services. With its ability to scale seamlessly and deliver messages to millions of recipients, Amazon SNS has become a cornerstone for building real-time communication and notification systems.

<div style="text-align: right">
<a href="https://docs.aws.amazon.com/sns/">More informations about Amazon Simple Notifications Service</a>
</div>

#### Amazon CloudWatch

Launched in 2008, **Amazon CloudWatch** is a monitoring service that provides insights into the performance, health, and usage of your applications, systems, and infrastructure. It tracks data from your AWS resources and custom apps. With CloudWatch, you can monitor your apps, identify issues, and optimise your resources. It has evolved to support more AWS services and features, making it a comprehensive solution for monitoring and troubleshooting cloud-based apps.

<div style="text-align: right">
<a href="https://000008.awsstudygroup.com/">FCJ - Workshop about Amazon CloudWatch</a>
<br>
<a href="https://docs.aws.amazon.com/cloudwatch/">More informations about Amazon CloudWatch</a>
</div>

#### AWS Command Line Interface

The **AWS Command Line Interface (CLI)** is an open-source tool that enables you to interact with AWS services using commands in a terminal. It provides functionality equivalent to the AWS Management Console. It supports various command environments: Linux, Windows, MAC OS. The AWS CLI is a powerful tool for managing cloud infrastructure.

<div style="text-align: right">
<a href="https://000011.awsstudygroup.com/">FCJ - Workshop about AWS CLI</a>
<br>
<a href="https://docs.aws.amazon.com/cli/">More informations about AWS CLI</a>
</div>

#### Amazon Elastic Compute Cloud

**Amazon EC2 (Elastic Compute Cloud)** is a cloud computing service that provides scalable computing capacity in the cloud. It allows you to rent virtual servers (instances) with various configurations. Launched in 2006, EC2 was one of the first cloud computing services offered by Amazon Web Services (AWS). It played a pivotal role in popularizing cloud computing and has since become a cornerstone of the AWS platform.

<div style="text-align: right">
<a href="https://000004.awsstudygroup.com/">FCJ - Workshop about AWS EC2</a>
<br>
<a href="https://docs.aws.amazon.com/ec2/">More informations about AWS EC2</a>
</div>