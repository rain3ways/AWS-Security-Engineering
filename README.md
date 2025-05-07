# 🔒 AWS Security Engineering: Foundational IAM Access Control

[![AWS IAM](https://img.shields.io/badge/AWS-IAM-FF9900?logo=amazon-aws&style=for-the-badge)](https://aws.amazon.com/iam/)
[![Principle of Least Privilege](https://img.shields.io/badge/Security-Least_Privilege-blueviolet?style=for-the-badge)](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)
[![MFA Enabled](https://img.shields.io/badge/MFA-Enforced-brightgreen?style=for-the-badge)](https://aws.amazon.com/iam/features/mfa/)

**Implementing robust and secure access control foundations in AWS using Identity and Access Management (IAM) best practices, including user/group management, policy enforcement, and role-based access for privileged operations.**

This repository showcases solutions and patterns for establishing secure and scalable access management within AWS environments. The primary project detailed here focuses on setting up a foundational administrative structure using IAM Users, Groups, and Roles, adhering to the principle of least privilege and enabling secure operational practices. This aligns with the DevOps principles of **Flow** (streamlined, secure access provisioning) and **Feedback** (auditable access via CloudTrail).

---

## 🚀 Featured Project: Establishing Secure Administrative Access with IAM

This project addresses the fundamental requirement of securely managing administrative access to an AWS account, moving away from root user usage for daily tasks and implementing a scalable, auditable IAM framework.

### 🎯 Business Challenge
Many organizations, especially when starting with AWS, face the challenge of managing access securely and efficiently. Over-reliance on the root user for administrative tasks poses significant security risks. There's a need for a structured approach to:
* Organize users based on their roles and responsibilities.
* Grant permissions based on the principle of least privilege.
* Provide temporary elevated access for administrative tasks without sharing long-term, highly privileged credentials.
* Ensure accountability and auditability for all actions performed.

### ✨ The Solution & Architectural Decisions
A foundational IAM structure was implemented to address these challenges:

* **IAM Groups for Permission Aggregation:**
    * An `AdminGroup` was created to consolidate administrative permissions.
    * The AWS managed policy `AdministratorAccess` was attached to this group. This centralizes permission management for administrators.
* **Dedicated IAM Users:**
    * An `AdminUser` was created for daily administrative login to the AWS Management Console. This user was added to the `AdminGroup` to inherit administrative privileges, avoiding direct policy attachment to the user.
    * An `OperatorUser` was created with *no direct permissions* initially. This user represents a standard operator who should not have standing administrative privileges.
* **IAM Role for Privileged Access (Role-Based Access Control - RBAC):**
    * An `AdminRole` was created with the `AdministratorAccess` managed policy.
    * The trust policy for `AdminRole` was configured to allow assumption by IAM users within the same AWS account.
* **Role Assumption for `OperatorUser`:**
    * An inline IAM policy (`AllowSwitchAdminPolicy`) was attached directly to `OperatorUser`. This policy explicitly grants *only* the `sts:AssumeRole` permission for the specific `AdminRole` ARN.
    * This setup ensures that `OperatorUser` can only gain administrative privileges temporarily by explicitly switching to `AdminRole`.
* **Security Best Practices Applied:**
    * Avoided root user for daily tasks.
    * Implemented group-based permission management.
    * Utilized IAM Roles for temporary elevated permissions (least privilege for `OperatorUser` by default).
    * Enforced password complexity and mandatory password change on first login for new IAM users.

#### 🏗️ IAM Structure Diagram
* A diagram illustrating the relationships: Root User (secured) -> AdminUser (in AdminGroup) -> Assumes -> AdminRole. And OperatorUser -> Assumes -> AdminRole.
 ![IAM Structure Diagram](/screenshots/iam-structure-diagram.png)
#### 🎬 Video Walkthrough: Secure Login & Role Switching
* A short video demonstrating:
    1.  Logging in as `AdminUser`.
    2.  Logging out and logging in as `OperatorUser` (showing limited/no direct permissions).
    3.  `OperatorUser` successfully switching to `AdminRole`.
    4.  Verifying administrative access while in `AdminRole`.
    5.  Switching back to `OperatorUser`.
    
    [▶️ Watch IAM Setup & Role Switch Demo](https://youtu.be/Wb8gZA3zzmw)
    
### 📈 Demonstrated Security Improvements & Operational Efficiency

| Feature Implemented                      | Security/Operational Benefit                                                                 |
| :--------------------------------------- | :------------------------------------------------------------------------------------------- |
| **IAM Groups for Permissions** | Centralized permission management; easier to add/remove admin users.                         |
| **Dedicated IAM Admin User** | Reduced root user usage; auditable actions tied to a specific user.                          |
| **IAM Role for Admin Tasks (`AdminRole`)** | Temporary elevation of privileges; credentials auto-rotate; adheres to least privilege.       |
| **Dedicated Operator User (`OperatorUser`)** | Enforces least privilege by default; explicit action (Switch Role) required for admin tasks. |
| **Specific `sts:AssumeRole` Policy** | Granular control over which users can assume which roles.                                    |
| **Password Policies Enforced** | Enhanced credential security for console access.                                             |

#### 📸 Screenshots of Key Configurations
* Include screenshots of:
  
    ![AdminGroup Permissions](/screenshots/admingroup-with-administratoraccess.png)
    *AdminGroup with AdministratorAccess policy.*

    ![OperatorUser AssumeRole Policy](/screenshots/operatoruser-permissions.png)
    *Inline policy granting OperatorUser permission to assume AdminRole.*

    ![AdminRole trust relationship policy](/screenshots/adminrole-trust-relationship-policy.png)
    *AdminRole trust relationship policy.*

    ![The Switch Role interface in the AWS console](/screenshots/switch-role-interface.png)
    *The Switch Role interface in the AWS console.*

### 💬 Interview Gold: Q&A Highlights

❓ *"How do you recommend setting up initial administrative access to a new AWS account while adhering to security best practices?"*

✅ **Approach Implemented:** "The first step is to secure the root user with a strong password and MFA, then avoid using it for daily tasks. Next, create an IAM group (e.g., `AdminGroup`) and attach the `AdministratorAccess` managed policy. Then, create individual IAM users (e.g., `AdminUser`) for administrators, add them to this group, and enforce MFA. For privileged operations that aren't needed constantly, create a dedicated IAM Role (e.g., `AdminRole`) with `AdministratorAccess`. Standard operator users (`OperatorUser`) should be created with minimal or no direct permissions, and then granted an inline policy allowing them to *only* assume the `AdminRole` via `sts:AssumeRole`. This ensures administrative privileges are temporary and explicitly invoked."

❓ *"Why is using IAM Roles preferred over directly assigning powerful policies like `AdministratorAccess` to IAM Users?"*

✅ **Rationale:** "IAM Roles provide temporary security credentials that are automatically rotated by AWS, reducing the risk associated with long-term static credentials like access keys for users. When a user assumes a role, they operate with the role's permissions only for that session. This aligns with the principle of least privilege, where users only have elevated permissions when actively performing tasks that require them. It also provides a clearer audit trail (via CloudTrail) of who assumed which role and when, as opposed to a user always having standing high-level permissions."

### 🛠 Tech Stack & Concepts Covered

| Technology/Concept       | Purpose in this Project                                                                    |
| :----------------------- | :----------------------------------------------------------------------------------------- |
| **AWS IAM Users** | Individual identities for console/programmatic access.                                   |
| **AWS IAM Groups** | Collections of users for simplified, centralized permission management.                    |
| **AWS IAM Policies** | JSON documents defining permissions (e.g., `AdministratorAccess`, custom inline policy). |
| **AWS IAM Roles** | Mechanism to grant temporary permissions to trusted entities (users, services).            |
| **Trust Policy** | Defines which principals (e.g., AWS account, IAM users) can assume a role.                 |
| **Permissions Policy** | Defines what actions the assumed role can perform on which resources.                      |
| **`sts:AssumeRole`** | The specific IAM action that allows a principal to assume a role.                          |
| **Principle of Least Privilege** | Granting only the minimum permissions necessary to perform a task.                 |
| **MFA (Mentioned)** | Multi-Factor Authentication for enhanced login security (recommended best practice).     |

### ❓ Technical FAQ

**Q: Why create both an `AdminUser` in `AdminGroup` AND an `AdminRole`? Isn't that redundant?**
A: While both ultimately provide `AdministratorAccess`, they serve slightly different best practice philosophies. The `AdminUser` (in `AdminGroup`) is for individuals whose *primary function is daily administration* and who might need standing admin access for general tasks. The `AdminRole` (assumed by `OperatorUser`) is for users whose primary function *isn't* administration but who *occasionally need to elevate their privileges* for specific tasks. This separation ensures that users like `OperatorUser` don't have standing admin rights, adhering more strictly to least privilege for their default state. In very small setups, one might simplify, but separating is a good habit for larger or more security-conscious environments.

**Q: Can the trust policy for `AdminRole` be more restrictive than allowing any user from the account to assume it?**
A: Absolutely. The trust policy can be configured to allow only specific IAM users or users belonging to specific IAM groups to assume the role. For example, you could restrict `AdminRole` to only be assumable by members of an `OperationsTeam` group, or only by `OperatorUser`. This further tightens security.

**Q: What's the difference between an inline policy and a managed policy?**
A: **Managed policies** (AWS Managed like `AdministratorAccess`, or Customer Managed) are standalone policy objects that can be attached to multiple users, groups, and roles. **Inline policies** are embedded directly into a single user, group, or role and are part_of that identity; if you delete the identity, the inline policy is deleted too. Inline policies are useful for one-to-one relationships or very specific permission sets not intended for reuse. In this lab, the `sts:AssumeRole` permission for `OperatorUser` was made an inline policy because it's specific to that user and that role.

#### 🔬 Deeper Dive / Further Analysis

*In Progress...*

### 📐 Solution Implementation & Verification
1.  **IAM Group & User Setup:** `AdminGroup` created with `AdministratorAccess` policy. `AdminUser` created and added to `AdminGroup`. Console login as `AdminUser` verified.
2.  **IAM Role Setup:** `AdminRole` created with `AdministratorAccess` policy and a trust policy allowing assumption by users in the current AWS account.
3.  **Operator User & Permissions:** `OperatorUser` created with no direct permissions. An inline policy attached to `OperatorUser` granting `sts:AssumeRole` permission specifically for `AdminRole`.
4.  **Role Switching Verification:** Logged in as `OperatorUser`, successfully switched to `AdminRole`, and verified administrative privileges. Confirmed ability to switch back.
   
---

## 📚 Explore Other Security Engineering Projects

This repository will also feature other projects related to AWS security best practices:

| Project Idea                                           | Focus Area                                       | Status    |
| :----------------------------------------------------- | :----------------------------------------------- | :-------- |
| 📂 **[Network Security with VPC & NACLs](./vpc-network-segmentation/)** | VPC Design, Subnetting, NACLs, Security Groups | Planned   |
| 📂 **[Data Encryption at Rest & In Transit](./data-encryption-kms-ssl/)** | KMS, S3 Encryption, SSL/TLS for ELB/CloudFront | Planned   |
| 📂 **[Automated Security Auditing with AWS Config](./aws-config-compliance/)** | Compliance as Code, Conformance Packs           | Planned   |

*(Click links as they become available to explore other implementations.)*

---

* [Return to Main Portfolio](https://github.com/rain3ways/AWS-DevOps-Portfolio)

**🔍 Request Security Audit**  
Need IAM architecture review? [Book Technical Consultation](mailto:rainning.rb@gmail.com?subject=Security%20Audit)

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Profile-blue?logo=linkedin)](https://www.linkedin.com/in/rainb/) | [![Portfolio](https://img.shields.io/badge/Portfolio-Website-green)](https://github.com/rain-xiii/AWS-DevOps-Portfolio) 