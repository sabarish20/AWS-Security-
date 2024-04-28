# Scenario 
On your first day as a security analyst at Huge Logistics, you activate AWS Security Hub. The next day you are met with a detailed dashboard, revealing a myriad of potential risks within the AWS infrastructure. Use this service to identify risks and help improve their security posture!

#### What is Security Hub?

AWS Security Hub is a cloud security posture management service that performs security best practice checks against your AWS resources. It provides a centralized, aggregated, and prioritized overview of security findings and compliance status in a standard format for a single AWS account and multiple AWS accounts.
AWS Security Hub aggregates security findings from various AWS services such as Amazon GuardDuty, Amazon Inspector, AWS Systems Manager, AWS Health, AWS Config, Amazon Macie, AWS IAM Access Analyzer and AWS Firewall Manager.
It helps in identifying misconfigurations, vulnerabilities, and any deviations from security best practices, providing actionable recommendations for remediation.

#### Pricing: 
AWS Security Hub follows a pay-as-you-go pricing model, where charges are based on the number of security checks conducted and the number of AWS accounts you've enabled Security Hub in. The first 10,000 finding ingestion events every month are free.

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/a95662b5-1780-4275-ad05-b497ec048489)

Please refer this [article](https://medium.com/globant/enhance-aws-cloud-security-with-aws-security-hub-e9b770cb323a) to setup your AWS Security Hub. 

### Reviewing finding using the AWS Console 

The summary panel gives a overview of our compliance against the security standards that we have enabled. 
It shows us the average overall score of the standards and also each individual standards score. 

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/45a9e020-9a2c-4192-817f-fbf7e7bb10aa)

Scrolling down, we can see finding by Region, Most common threat type and other details.

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/46da6d64-52d0-4db4-8e93-f4742dfa7abd)

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/3e8dbfe2-a2cd-4798-bb92-473128fb1f7c)

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/18f3443e-f533-4901-b799-f61c22669138)

Navigating to Controls panel show us number of findings according to severity rating.

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/ac41c9ff-58b7-401a-b58e-cb88f53f26c0)

Clicking on Security standards in the left navigation panel bring up the following page, where we again see our compliance and can view the findings by standard.

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/4629bfcc-65b5-4d03-ba0c-49e89f7d93ec)

Insights panel gives us a collection of common security insights. This includes credentials that may have leaked, publicly available `S3` and compromised `EC2` instances.  

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/9ddc8cf2-a067-4339-969e-06ef9e195a10)

It also seems that one of our EC2 instances is associated with adversary reconnaissance. Does this mean that it's been compromised and the attacker is using it to enumerate our internal cloud environment?

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/eea4c177-d8fc-45f8-a38f-3893ff99a219)

Actually it seems that an attacker or automated bot is attempting to probe or brute force a service. We would need to dig deeper in Amazon Detective to identify the nature of the probing, and find out more information about the potential adversary.

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/74eb69bb-5433-4206-895c-df97a770250a)

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/01aad594-a14a-49c4-9ad0-0f8b4d01c847)

Looking into Findings panel:

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/73b8dc5d-792f-432a-968e-4e199ebd8bef)

We can also sort findings by severity filters or we can also look for specific events. 

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/65da67d8-c2d4-4971-9fb9-101d24fae7a8)

Automations Panel:

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/d0c15a7d-6926-4fb9-9045-d44359ed5d78)

Automations helps in quick modifications and remediations on findings. It supports two types of automations:
- Automation rules - It can be used to automatically update findings such as changing their severity, adding notes to findings and suppressing them. The Security Hub administrator can create an automation rule by defining rule criteria. When a finding matches the defined criteria, Security Hub applies the rule action to it.
- Automated response and remediation - We can create custom EventBridge rules that can take automatic actions based on specific findings and insights.

# Using CLI 

Before starting, configure the keys and profile. 

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/e37106be-c2aa-4446-8c82-488dab5e04ac)

The command below lists all critical findings and includes the finding ID and title as well as the ARN (the globally unique Amazon Resource Name) of the affected resource.
```
aws securityhub get-findings --filters '{"SeverityLabel": [{"Value": "CRITICAL", "Comparison": "EQUALS"}]}' --query 'Findings[].[Id, Title, Resources[0].Id]'
```

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/f3ee8d74-b077-4739-8ea7-5571435a0092)

We can also dig more on specific findings which returns more information on the finding using the below command:

```
aws securityhub get-findings --filters '{"AwsAccountId": [{"Value": "ACCOUNTID", "Comparison": "EQUALS"}], "Id": [{"Value": "RESOURCEARN", "Comparison": "EQUALS"}]}'
```

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/6af89ac5-b6c0-4dc0-b01f-9cc6bbf19c7d)

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/c6e9e165-e14f-4631-9829-3d2907a32ffe)

# Finding the flag:

See if you can investigate the finding ID below and get a flag (MD5 hash) from the resource!

```
arn:aws:securityhub:us-east-1:427648302155:security-control/S3.2/finding/0b6be6de-bc91-4f76-8127-ee2bed6bd473
```

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/8e91b6e0-a36c-4f64-bd80-954d8f1db06b)

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/cc601196-d49b-4cd2-9829-cf086c769561)

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/4476666b-e76b-404e-92d8-aec7d9a1893b)

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/25d963f9-a86a-4d07-9d59-0941d7a33ea1)

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/f9fba86c-0dd0-4557-8fcd-92fa0d8cf020)

Flag: e8e98717c9bc450b625cb967d673f5ab

We have now successfully completed the lab. 
