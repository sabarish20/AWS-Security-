Starting as the IAM user Chris, the attacker discovers that they can assume a role that has full Lambda access and pass role permissions. The attacker can then perform privilege escalation to obtain full admin access.

Enumerating the User:

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/44342e51-a1dd-4514-8ada-1d0834549877)



Inline Policies:
- This lists the Inline policies that is attached to the user. 
- Inline policies are policies that are directly attached to an IAM user, group, or role.
- It is a strict one-to-one relationship between the policy and identity.
- They are deleted when you delete the identity.

Enumerate Inline policies:

```
aws iam list-user-policies --user-name USERNAME
```
![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/5f846c90-17c8-4e24-85d1-53816eb8918c)

Managed Policies:
- There are two types of Managed Policies:
	1) AWS Managed Policy
       - It is created and administered by AWS.
       - It is a standalone policy means that the policy has its own `ARN` that includes the policy name.
       - For example, `arn:aws:iam::aws:policy/IAMReadOnlyAccess` is an AWS managed policy.    
	2) Customer Managed Policies
       - These are policies that are customly designed by the user.   
  
Enumerating Managed policies:

```
aws iam list-attached-user-policies --user-name USERNAME
```
![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/8dff4592-de2a-433c-9c15-3697f0fba982)

Listing the policy versions:

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/9862ba9b-0af5-4746-86b1-e85625ab8362)

Getting the policy:

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/784160c4-6479-41a2-a7ca-3c765048bda3)

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/5d10371e-ad2f-4eab-98e0-77ddd24bf026)

List the available roles

```
aws iam list-roles
```
![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/222bd1fd-04fd-441b-8ac6-511785b2a05d)

Lets analyze the permissions associated with the cg-lambdamanager role to see what can be gained by assuming it. Run the following to enumerate the cg-lambdamangers permissions.

```
aws iam list-attached-role-policies --role-name ROLE-NAME 
```

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/9b6d78cf-7f19-4366-96d3-0b0a4e9adce0)

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/0df82ae7-606f-48ed-8a3a-70d33201ea13)

![image](https://github.com/sabarish20/Security-in-AWS/assets/85452305/80055da5-cd23-4fd1-ba98-2e3923d6cdb9)



