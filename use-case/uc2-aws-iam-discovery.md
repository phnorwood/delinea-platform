# UC-2: Discovery and Onboarding of IAM User Access Keys

**Objective:** Use the Delinea Platform's discovery capabilities to automatically find AWS IAM Users and their associated Access Keys, then import these secrets into the vault for centralized management.

**Prerequisites:**
- Delinea Platform tenant configured with admin access
- Platform Engine and Distributed Engine installed and active
- AWS account with IAM users and access keys to discover
- AWS IAM credentials with permissions to read IAM users and keys
- Completed UC-1 (folder structure for AWS accounts exists)

**Estimated Time:** 45-60 minutes

---

## Step 1: Create AWS Discovery Service Account

### 1.1: Create IAM User in AWS Console

1. Log in to AWS Console
2. Navigate to **IAM → Users**
3. Click **Create user**
4. Configure:
   - **User name:** `delinea-discovery-svc`
   - **Access type:** ☑ Programmatic access
5. Click **Next: Permissions**
6. Click **Attach existing policies directly**
7. Search for and select: **ReadOnlyAccess** (or create custom policy below)
8. Click **Next: Tags** (optional, skip if not needed)
9. Click **Next: Review**
10. Click **Create user**
11. **IMPORTANT:** Copy the credentials:
    - **Access Key ID:** `AKIA...`
    - **Secret Access Key:** `wJalrXUtn...`
12. Save these securely (you'll need them in Step 2)
13. Click **Close**

### 1.2: Create Custom Discovery Policy (Optional - More Restrictive)

If you prefer minimal permissions instead of ReadOnlyAccess:

1. In AWS Console, navigate to **IAM → Policies**
2. Click **Create policy**
3. Click **JSON** tab
4. Paste the following policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:ListUsers",
        "iam:GetUser",
        "iam:ListAccessKeys",
        "iam:GetAccessKeyLastUsed",
        "iam:ListUserPolicies",
        "iam:ListAttachedUserPolicies",
        "iam:GetUserPolicy",
        "iam:GetPolicy",
        "iam:GetPolicyVersion"
      ],
      "Resource": "*"
    }
  ]
}
```

5. Click **Next: Tags** (optional)
6. Click **Next: Review**
7. Configure:
   - **Name:** `DelineaDiscoveryPolicy`
   - **Description:** `Allows Delinea to discover IAM users and access keys`
8. Click **Create policy**
9. Return to your `delinea-discovery-svc` user
10. Click **Add permissions → Attach policies directly**
11. Search for and select: `DelineaDiscoveryPolicy`
12. Click **Add permissions**

---

## Step 2: Store AWS Discovery Credentials in Platform

1. Log in to Delinea Platform
2. Navigate to **Secret Server → All Secrets**
3. Navigate to folder: `AWS → Admin Accounts`
4. Click **Create Secret**
5. Select template: **AWS Access Key**
6. Configure:
   - **Folder:** `/AWS/Admin Accounts`
   - **Secret Name:** `AWS Discovery Service Account - [Account-ID]`
   - **Access Key ID:** Paste the Access Key ID from Step 1.1
   - **Secret Access Key:** Paste the Secret Access Key from Step 1.1
   - **Notes:** "Service account for AWS IAM user discovery"
7. Click **Create Secret**

---

## Step 3: Configure AWS Discovery Source

### 3.1: Enable Discovery

1. Navigate to **Discovery → Configuration**
2. Click **Edit**
3. Verify **☑ Enable discovery** is checked
4. Click **Save**

### 3.2: Create Discovery Source

1. Navigate to **Discovery → Sources**
2. Click **Create Source**
3. In **Discovery type**, select **Vault**
4. Click **Amazon Web Services (AWS)**
5. Click **Continue**
6. Configure source settings:
   - **Discovery Source Name:** `AWS IAM Users - [Account-Name]`
   - **Description:** `Discovery of IAM users and access keys for [account]`
   - **State:** Enabled
   - **AWS Region:** Select your primary region (e.g., `us-east-1`)
   - **Discovery Secret:** Click **Select existing secret**
     - Browse to `/AWS/Admin Accounts`
     - Select: `AWS Discovery Service Account - [Account-ID]`
     - Click **Select**
   - **Discovery Site:** Select your site
7. Click **Save**

### 3.3: Configure Discovery Scanners

1. The **Add discovery scanners** page appears automatically
2. Review available scanners:
   - **AWS IAM User Scanner** - Discovers IAM users
   - **AWS Access Key Scanner** - Discovers access keys for IAM users
3. Click **Add Scanner** for:
   - ☑ AWS IAM User Scanner
   - ☑ AWS Access Key Scanner
4. Click **Save**

---

## Step 4: Configure Discovery Rules

### 4.1: Create Import Rule for IAM Users

1. Navigate to **Discovery → Configuration**
2. Click **Import Rules** tab
3. Click **Create Rule**
4. Configure:
   - **Rule Name:** `Import AWS IAM Users`
   - **Object Type:** AWS IAM User
   - **Discovery Source:** Select `AWS IAM Users - [Account-Name]`
5. Click **Next**
6. Configure import settings:
   - **Secret Template:** AWS IAM Console Account
   - **Default Folder:** Select `/AWS/Admin Accounts`
   - **Secret Name Pattern:** `AWS IAM - {Username} - [Account-ID]`
7. Click **Next**
8. Configure filters (optional):
   - **Filter by username pattern:** Leave blank to import all
   - **Exclude service accounts:** ☑ (optional)
9. Click **Save**

### 4.2: Create Import Rule for Access Keys

1. Still in **Discovery → Configuration → Import Rules**
2. Click **Create Rule**
3. Configure:
   - **Rule Name:** `Import AWS Access Keys`
   - **Object Type:** AWS Access Key
   - **Discovery Source:** Select `AWS IAM Users - [Account-Name]`
4. Click **Next**
5. Configure import settings:
   - **Secret Template:** AWS Access Key
   - **Default Folder:** Select `/AWS/Admin Accounts`
   - **Secret Name Pattern:** `AWS Access Key - {Username} - [Account-ID]`
6. Click **Next**
7. Configure filters (optional):
   - **Import active keys only:** ☑
   - **Exclude keys older than:** 90 days (optional)
8. Click **Save**

---

## Step 5: Run Discovery Scan

### 5.1: Execute Discovery

1. Navigate to **Discovery → Network View**
2. Click **Sources** tab
3. Find your source: `AWS IAM Users - [Account-Name]`
4. Click the three dots (⋮) menu
5. Click **Run Discovery Scan**
6. Confirm: Click **Run Now**
7. Monitor progress:
   - Status will show "Running"
   - Wait 5-10 minutes for completion
   - Refresh page to see updated status

### 5.2: Review Discovery Results

1. Navigate to **Discovery → Network View**
2. Click **Accounts** tab
3. Filter by source: `AWS IAM Users - [Account-Name]`
4. Review discovered items:
   - Column **Account Name** shows IAM usernames
   - Column **Account Type** shows "AWS IAM User" or "AWS Access Key"
   - Column **Status** shows "Discovered" (not yet imported)
5. Note the count of discovered users and keys

---

## Step 6: Import Discovered Accounts

### 6.1: Select Accounts for Import

1. In **Discovery → Network View → Accounts** tab
2. Filter to show only AWS accounts from your discovery source
3. Review the list of discovered IAM users and access keys
4. Select accounts to import:
   - ☑ Check individual accounts, or
   - ☑ Check the header checkbox to select all
5. Click **Import** button in the toolbar

### 6.2: Configure Import Settings

1. The **Import Accounts** wizard appears
2. Review pre-populated settings:
   - **Folder:** Should show `/AWS/Admin Accounts` (from import rule)
   - **Secret Template:** Should show appropriate template
   - **Site:** Your discovery site
3. Configure password management:
   - Select **Import without changing password**
   - Or select **Generate new password** if you want immediate rotation
4. Click **Next**

### 6.3: Complete Import

1. Review summary:
   - Number of accounts to import
   - Target folder
   - Secret naming pattern
2. Click **Import**
3. Wait for import to complete (30-60 seconds)
4. Click **Close** when finished

---

## Step 7: Verify Imported Secrets

### 7.1: Check Vault Contents

1. Navigate to **Secret Server → All Secrets**
2. Browse to folder: `/AWS/Admin Accounts`
3. Verify new secrets appear:
   - Format: `AWS IAM - [username] - [Account-ID]`
   - Format: `AWS Access Key - [username] - [Account-ID]`
4. Count should match number of accounts imported

### 7.2: Inspect Individual Secret

1. Click on one of the imported secrets
2. Verify **Overview** tab contains:
   - Access Key ID (for access key secrets)
   - Username (for IAM user secrets)
   - Account ID or ARN
3. Click **Discovery** tab
4. Verify:
   - **Discovery Source:** Shows your AWS discovery source
   - **Last Discovered:** Shows recent timestamp
   - **Discovery Status:** Active

---

## Step 8: Configure Continuous Discovery

### 8.1: Set Discovery Schedule

1. Navigate to **Discovery → Sources**
2. Find your source: `AWS IAM Users - [Account-Name]`
3. Click the source name to edit
4. Click **Schedule** tab
5. Click **Edit**
6. Configure schedule:
   - **Enable scheduled discovery:** ☑
   - **Frequency:** Daily
   - **Time:** 2:00 AM (or off-hours for your organization)
7. Click **Save**

### 8.2: Enable Automatic Import (Optional)

1. Still in source configuration
2. Click **Settings** tab
3. Click **Edit**
4. Configure:
   - **☑ Automatically import new accounts**
   - **☑ Update existing secrets from discovery**
5. Click **Save**

---

## Step 9: Verify Discovery Logging

### 9.1: Review Discovery Audit Trail

1. Navigate to **Insights → Reports**
2. Select report: **Discovery Audit Report**
3. Filter by:
   - **Discovery Source:** `AWS IAM Users - [Account-Name]`
   - **Date Range:** Last 7 days
4. Review logged events:
   - Discovery scan initiated
   - Number of accounts found
   - Accounts imported
   - User who performed import
5. Verify timestamps are accurate

### 9.2: Export Discovery Report

1. Click **Export** button
2. Select format: **CSV** or **PDF**
3. Save file for compliance documentation

---

## Step 10: Test Discovered Credentials

### 10.1: Verify IAM User Secret

1. Navigate to one of the imported IAM user secrets
2. Click **Overview** tab
3. Note the username
4. If password was imported, use AWS console to verify:
   - Navigate to AWS Console login
   - Enter IAM username
   - Enter password from secret
   - Verify successful authentication (or verify password needs reset if not imported)

### 10.2: Verify Access Key Secret

1. Navigate to one of the imported access key secrets
2. Click **Overview** tab
3. Copy the **Access Key ID** and **Secret Access Key**
4. Test using AWS CLI (if installed):
   ```bash
   aws configure set aws_access_key_id [Access-Key-ID]
   aws configure set aws_secret_access_key [Secret-Access-Key]
   aws iam get-user
   ```
5. Verify command returns user information
6. Or verify in AWS Console:
   - Navigate to **IAM → Users → [username] → Security credentials**
   - Verify Access Key ID matches what was discovered

---

## Success Criteria Verification

Verify you have successfully completed UC-2:

- ✅ AWS IAM discovery service account created with appropriate permissions
- ✅ Discovery credentials stored securely in Platform vault
- ✅ AWS discovery source configured and enabled
- ✅ Discovery scanners configured for IAM users and access keys
- ✅ Discovery scan executed successfully
- ✅ IAM users discovered and displayed in Network View
- ✅ Access keys discovered and associated with correct IAM users
- ✅ Discovered accounts imported into vault with proper naming
- ✅ Secrets created in correct folder structure
- ✅ Discovery metadata visible in imported secrets
- ✅ Continuous discovery scheduled for automated ongoing scans
- ✅ Discovery audit trail generated and exportable
- ✅ Imported credentials tested and verified functional

---

**Document Version:** 1.0  
**Last Updated:** February 2026