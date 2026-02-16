# Delinea Discovery & RPC Permissions Reference

This document provides a comprehensive reference of permissions required for **Delinea Platform**, including for discovery, management, and CIEM/ITDR.

---

## Active Directory

### Discovery

**What Discovery Scans:**
- Organizational Units (OUs) and Windows computers on the domain
- Domain Accounts (AD user accounts)
- Local Accounts (Local Windows accounts on each machine)
- IIS Application Pools (running as AD accounts)
- Windows Services (running as AD accounts)
- Scheduled Tasks (running as AD accounts)

**Account Requirements:**

**Domain-Level Permissions:**
- Default AD configuration typically allows scanning computers and users
- Account must be able to query Active Directory via LDAP

**Machine-Level Permissions:**
- **Access this computer from the network** permission
- Membership in local **Administrators** group on target machines
- **Network access: Restrict clients allowed to make remote calls to SAM** (for Windows Server 2016+, Windows 10 1607+)
  - Discovery account must have **Remote Access** permission allowed

**Service Account Discovery (Additional Requirements):**
- Must be a **domain account**
- Must be in the **Administrators** group on target machines

**Configuration Method:**
Use Group Policy to add discovery account to local Administrators group:
- Navigate to: Computer Configuration > Preferences > Control Panel Settings
- Configure: Local Users and Groups > Local Group > Administrators (Built-in)

**Best Practice - Limit Login Privileges:**
- Add discovery account to **Deny log on locally** policy
- Add discovery account to **Deny log on through Remote Desktop Services** policy

**Technologies Used:**
- LDAP (for querying AD and OUs)
- WMI (Windows Management Instrumentation) - for services
- WMA (Microsoft Web Administration) - for IIS application pools
- Windows Task Scheduler interfaces - for scheduled tasks

**Known Limitations:**
- **Windows Server 2016/2019:** Scheduled tasks not discovered unless Secret Server instance or distributed engine is on the same domain as target server
  - Scheduled task discovery only retrieves SID, which must be translated to username on same domain

---

### RPC (Remote Password Changing)

**Overview:**
The privileged Secret Server RPC service account requires granular Active Directory permissions configured using ADSI Edit and Active Directory Users and Computers.

**ADSI Permissions (Password Settings Container):**

**Location:** CN=System > CN=Password Settings Container

**Required Permission:**
- **Read** permission on Password Settings Container

**Delegate Control Permissions:**

**Applied To:** Organizational Unit (OU) or top-level domain where accounts reside

**Configuration:** Custom task delegation for User objects only

**Required Permissions:**
- Change Password
- Read lockoutTime
- Read pwdLastSet
- Reset Password
- Write lockoutTime
- Write pwdLastSet
- Write UserAccountControl
- Read UserAccountControl

**Protected Group Accounts (Additional Configuration):**

For accounts in protected groups (e.g., Domain Admins), use `dsacls` commands on domain controller:

**Account Unlock Permissions:**
```
dsacls "dc=<domain>,dc=<tld>" /G "<domain>\<account>:RP;msDS-User-Account-Control-Computed;user" /I:S
dsacls "dc=<domain>,dc=<tld>" /G "<domain>\<account>:RPWP;lockoutTime;user" /I:S
dsacls "CN=AdminSDHolder,CN=System,DC=<domain>,DC=<tld>" /G "<domain>\<account>:RPWP;lockoutTime"
```

**Password Reset Permissions:**
```
dsacls "dc=<domain>,dc=<tld>" /G "<domain>\<account>:CA;Reset Password;user" /I:S
dsacls "CN=AdminSDHolder,CN=System,DC=<domain>,DC=<tld>" /G "<domain>\<account>:CA;Reset Password"
```

**SDProp Process:**
After configuring AdminSDHolder permissions, the Security Descriptor Propagator (SDProp) must propagate changes to protected admin groups. This can be initiated manually using `ldp.exe` with the `RunProtectAdminGroupsTask` attribute.

**Tools Required:**
- **ADSI Edit** - Active Directory Service Interfaces editor (found on Domain Controllers)
- **Active Directory Users and Computers** - Standard AD management console
- **dsacls** - Command-line tool for setting AD permissions (for protected groups)
- **ldp.exe** - LDAP Data Interchange Format tool (for triggering SDProp)

---

## Amazon Web Services (AWS)

### Discovery
- `iam:ListUsers`
- `iam:GetLoginProfile`
- `iam:ListAccessKeys`
- `EC2:DescribeInstances`
- `EC2:DescribeAvailabilityZones`

### RPC - IAM Access Keys (Self-Rotation)
For accounts rotating their own keys:
- `iam:DeleteAccessKey`
- `iam:UpdateAccessKey`
- `iam:CreateAccessKey`
- `iam:ListAccessKeys`

### RPC - IAM Access Keys (Privileged)
For privileged accounts rotating other users' keys:
- `iam:DeleteAccessKey`
- `iam:UpdateAccessKey`
- `iam:CreateAccessKey`
- `iam:ListAccessKeys`

### RPC - IAM Console Password
- `iam:UpdateLoginProfile`
- `iam:ChangePassword`


### CIEM Discovery
**CloudTrail Configuration (Required):**
- CloudTrail must collect data from all regions
- CloudTrail must collect global service events (IAM and STS API calls)
- CloudTrail logs must be delivered to an S3 bucket

**S3 Bucket Permissions:**
- `s3:GetObject` on CloudTrail logs bucket
- Applied to the integration role (default: `AuthomizeCrossAccountTrustRole`)

**IAM Permissions:**
- AWS ReadOnlyAccess managed policy (automatically assigned via CloudFormation)

### AWS Identity Center (Additional Permissions)
When integrating AWS Identity Center:
- `AWSSSOReadOnly`
- `AWSSSODirectoryReadOnly`

---

## Google Cloud Platform (GCP)

### Discovery
- **Project Viewer** role
  - Lists projects, zones, service accounts, and instances

### RPC/Heartbeat
- **Service Account Key Admin** role
  - Creates, deletes, and rotates service account keys

### ITDR Integration
**Configuration:** Domain-wide Delegation in Google Workspace Admin Console

**OAuth 2.0 Scopes Required:**

These are permission scope identifiers (not URLs to visit) that must be configured in Google Workspace Admin Console under Security > Access and Data Control > API Controls > Domain-wide Delegation:

- `https://www.googleapis.com/auth/cloudplatformorganizations.readonly`
- `https://www.googleapis.com/auth/admin.directory.customer.readonly`
- `https://www.googleapis.com/auth/admin.directory.domain.readonly`
- `https://www.googleapis.com/auth/admin.directory.group.member.readonly`
- `https://www.googleapis.com/auth/admin.directory.group.readonly`
- `https://www.googleapis.com/auth/admin.directory.notifications`
- `https://www.googleapis.com/auth/admin.directory.orgunit.readonly`
- `https://www.googleapis.com/auth/admin.directory.rolemanagement.readonly`
- `https://www.googleapis.com/auth/admin.directory.user.readonly`
- `https://www.googleapis.com/auth/admin.directory.userschema.readonly`
- `https://www.googleapis.com/auth/admin.reports.audit.readonly`
- `https://www.googleapis.com/auth/cloud-platform.read-only`

---

## MSFT Entra ID

### Discovery
**Application Permissions Required** (must be Application Permissions, NOT Delegated Permissions):
- `EntitlementManagement.Read.All`
- `RoleManagement.Read.Directory`
- `User.Read.All`

### RPC
**Service Principal Role Assignments:**
- **User Administrator** role (for non-administrator accounts)
- **Privileged Authentication Administrator** role (for administrator accounts)

### ITDR/CIEM Integration

#### Read-Only Mode
**Microsoft Graph API Permissions** (Application Permissions with admin consent required)

**Coverage:**
- Users
- Groups
- Roles and admins
- Applications, Application Access, and Azure Managed Identities
- App Registrations
- Sign-in and Audit Log activities
- SharePoint Sites
- Office 365 Audit Log activities

#### Read/Write Mode
**Includes all Read-Only permissions PLUS:**
- Additional write-enabling OAuth scopes

**Capabilities:**
- Disable/enable users
- Revoke user sessions
- Add/remove users from groups

#### Azure Cloud Integration (Optional)
**Covers:**
- Azure subscriptions
- Resource groups
- Resources
- Roles and privileges
- Management log activities
- LLMs and AI Agents

**Required Role:**
- **Reader** role on subscriptions for CIEM functionality

---

## Unix

### Discovery
**Account Requirements:**
- SSH access to target systems
- Read access to `/etc/passwd`
- `sudo passwd <username>` capability (required for account takeover)

### RPC
- `passwd` command is used
- Default local settings apply during password changes

---

## ESXi

### Discovery
- **Shell Access** permission
- **Query VRM Policy** permission

---

## Windows (Local Accounts)

### Discovery
**Required Permissions:**
- **Access this computer from the network** permission
- Membership in local **Administrators** group

**Additional Requirement (Windows Server 2016+, Windows 10 1607+):**
- **Network access: Restrict clients allowed to make remote calls to SAM** policy
  - Discovery account must have **Remote Access** permission allowed

**Applicable to:**
- Windows Server 2016, 2019
- Windows 10 version 1607 and later
- Earlier versions with specific KB updates installed

---

## Windows (Services, Scheduled Tasks, App Pools, COM+)

### Discovery
**Account Requirements:**
- Domain account
- Membership in **Administrators** group on target machines
- **Access this computer from the network** permission

**Best Practice (Optional but Recommended):**
Limit discovery account login privileges:
- Add to **Deny log on locally** policy
- Add to **Deny log on through Remote Desktop Services** policy

---

## Notes

### Permission Types
- **Application Permissions:** Granted at the application/tenant level, do not require user interaction
- **Delegated Permissions:** Require a signed-in user, permissions are constrained by user's privileges
- **OAuth Scopes:** Define the level of access granted to an application for specific APIs

### CloudFormation Integration
AWS integrations using CloudFormation automatically provision the required IAM roles and policies. Manual integrations require creating these roles and policies manually.

### Domain-wide Delegation
Google Cloud Platform ITDR integration requires Domain-wide Delegation to be configured in Google Workspace Admin Console to grant the service account access to Google Workspace APIs on behalf of users.

---
