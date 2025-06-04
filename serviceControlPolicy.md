# 📘 What is a Service Control Policy (SCP)?

**Service Control Policies (SCPs)** are a feature of **AWS Organizations** that allow you to manage **permissions guardrails** for AWS accounts. SCPs control what services and actions can be used by IAM users, groups, and roles in those accounts.

> ❗ SCPs do **not grant permissions**. They only set **boundaries on permissions**. Actual permissions must still be granted via IAM policies.

---

# 🧠 Why Use SCPs?

- Enforce compliance and governance across multiple accounts.
- Prevent usage of restricted AWS services (e.g., disallow creation of EC2).
- Apply permission boundaries organization-wide or to specific accounts/OUs.
- Ensure no user can override critical restrictions, even if they have `AdministratorAccess`.

---

# 🏗️ SCP Key Concepts

| Term                    | Description                                                                 |
|-------------------------|-----------------------------------------------------------------------------|
| AWS Organizations       | A service for managing multiple AWS accounts. SCPs are applied within Organizations. |
| Root                    | Top-level container in an AWS Org; SCPs here apply to all accounts.         |
| OU (Organizational Unit)| A group of AWS accounts; SCPs can be applied to OUs for bulk enforcement.   |
| Effective Permissions   | Result of IAM policy **AND** SCP. SCPs can only reduce permissions.         |
| FullAWSAccess           | Default SCP that allows all actions unless explicitly denied.               |

---

# 📋 How SCPs Work

- SCPs are JSON policy documents, similar to IAM policies.
- Applied to:
  - **Root**: affects all accounts
  - **OUs**: affects all accounts in the OU
  - **Accounts**: specific accounts
- An **Allow** in SCP is only effective if **IAM also allows** it.
- A **Deny** in SCP overrides any IAM **Allow**, even if the user is an admin.

---

# 🔐 SCP Evaluation Logic

**Final Permissions = IAM Policies ∩ SCPs**

- SCPs do **not grant access**, they just **filter out** what's allowed.
- If **IAM allows** but **SCP denies** → Access is denied.
- If **IAM denies**, SCP can't override that denial.

---

# 📦 Example SCPs

## ✅ Allow Only S3 and EC2
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:*",
        "ec2:*"
      ],
      "Resource": "*"
    }
  ]
}
```

## 🚫 Deny Use of CloudFormation
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "cloudformation:*",
      "Resource": "*"
    }
  ]
}
```

## 🚫 Deny Access Outside a Specific Region
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": "us-east-1"
        }
      }
    }
  ]
}
```

---

# 🧰 Best Practices

- Start with `FullAWSAccess`, then restrict gradually.
- Use **Deny** policies cautiously – they are absolute.
- Test SCPs in **non-prod accounts or OUs** first.
- Log and monitor using **AWS CloudTrail** to observe effects.
- SCPs apply even to the **Root user** (except for a few actions like billing).

---

# 📎 Common Interview Questions

**Q1. Can SCPs grant permissions?**  
No. SCPs only limit or restrict permissions. IAM policies must grant access.

**Q2. What happens if SCP allows something but IAM denies it?**  
The action is **denied**. IAM policy must explicitly allow it and SCP must not deny it.

**Q3. Can you override an SCP Deny?**  
No. SCP Deny overrides all allows, including from IAM or even the Root user.

**Q4. What’s the default behavior of an account in an Org with no SCP?**  
If the `FullAWSAccess` SCP is attached (default), then all services are available as per IAM policies.

**Q5. How can you restrict usage to one region using SCP?**  
Use a **Deny** with a `StringNotEquals` condition on `aws:RequestedRegion`.

---

# 🧪 Scenario-Based Questions

## ✅ Scenario 1:
"You want to make sure no one can use RDS in a specific account, but EC2 and S3 should still work."

✅ Use an SCP that **denies `rds:*`** actions only.

---

## ✅ Scenario 2:
"How to enforce all developers to use resources only in `us-west-2`?"

✅ Use a Deny condition like:
```json
"Condition": {
  "StringNotEquals": {
    "aws:RequestedRegion": "us-west-2"
  }
}
```

---

## ✅ Scenario 3:
"Even admins shouldn’t be able to delete S3 buckets."

✅ SCP:
```json
{
  "Effect": "Deny",
  "Action": "s3:DeleteBucket",
  "Resource": "*"
}
```

---

# 🔚 Summary

| Feature      | Description                                 |
|--------------|---------------------------------------------|
| Purpose      | Set permission boundaries for accounts      |
| Scope        | Applied to accounts, OUs, or root           |
| Language     | JSON, similar to IAM                        |
| Cannot Do    | Cannot grant permissions                    |
| Use Case     | Governance, compliance, org-wide control    |
