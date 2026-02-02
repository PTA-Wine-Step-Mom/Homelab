# 2026-02-03: Configuring Proxmox Single Sign-On with Microsoft Entra ID

**Date:** February 3, 2026  
**Author:** Jacob Strange  
**Objective:** Configure Proxmox VE to authenticate users via Microsoft Entra ID (Azure AD) using OpenID Connect  
**Reference:** [Microsoft Entra ID SSO for Proxmox](https://docs.lucanoahcaprez.ch/books/azure-active-directory/page/microsoft-entra-id-sso-for-proxmox)

## Overview

This build log documents the configuration of Single Sign-On (SSO) between Proxmox Virtual Environment and Microsoft Entra ID. This integration enables users to authenticate using their organizational credentials instead of local Proxmox accounts.

### Benefits
- Centralized user authentication
- Automatic user provisioning
- Group-based access control
- Improved security through organizational identity policies

### Limitations
- SSO only works for Proxmox web interface, not console access
- Limited to OpenID Connect (not full OAuth 2.0)
- Console connections to cluster nodes still require Linux authentication

## Prerequisites

- [ ] Administrative access to Microsoft Entra ID
- [ ] Proxmox VE cluster with web interface access
- [ ] Root or administrative privileges on Proxmox
- [ ] Valid SSL certificate for Proxmox (recommended)
- [ ] Network connectivity from users to both Proxmox and Entra ID

## Implementation Steps

### Step 1: Create App Registration in Microsoft Entra ID

#### 1.1 Access Entra Admin Center
1. Navigate to [Microsoft Entra Admin Center](https://entra.microsoft.com/)
2. Sign in with administrative credentials
3. Go to **Identity** > **Applications** > **App registrations**

#### 1.2 Create New Registration
1. Click **New registration**
2. Configure the following settings:
   - **Name:** `ProxmoxVE-SSO`
   - **Supported account types:** "Accounts in this organizational directory only"
   - **Redirect URI:** 
     - Type: `Web`
     - URI: `https://<PROXMOX-FQDN>:8006/`
   - **Example:** `https://proxmox.homelab.local:8006/`

3. Click **Register**

#### 1.3 Record Application Details
📝 **Save the following values:**
- **Application (client) ID:** `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
- **Directory (tenant) ID:** `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

#### 1.4 Configure API Permissions
1. Navigate to **API permissions**
2. Verify the following delegated permissions are present:
   - `email`
   - `offline_access`
   - `openid`
   - `profile`
3. If missing, add them via **Add a permission** > **Microsoft Graph** > **Delegated permissions**
4. Click **Grant admin consent for [Tenant Name]**

#### 1.5 Create Client Secret
1. Go to **Certificates & secrets**
2. Click **New client secret**
3. Configure:
   - **Description:** `Proxmox SSO Secret`
   - **Expires:** Choose appropriate duration (12 months recommended)
4. Click **Add**
5. **⚠️ CRITICAL:** Copy the secret value immediately - it won't be shown again

📝 **Save the client secret value:** `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

### Step 2: Configure Optional Group Claims (Recommended)

#### 2.1 Enable Group Information in Tokens
1. In the App Registration, navigate to **Token configuration**
2. Click **Add groups claim**
3. Select **Security groups**
4. Ensure **ID** token is checked
5. Click **Add**

This enables group-based access control in Proxmox.

### Step 3: Configure OpenID Connect Realm in Proxmox

#### 3.1 Access Proxmox Web Interface
1. Connect to Proxmox web interface: `https://<PROXMOX-FQDN>:8006`
2. Log in with root credentials
3. Navigate to **Datacenter** > **Realms**

#### 3.2 Add OpenID Connect Server
1. Click **Add** > **OpenID Connect Server**
2. Configure the realm settings:

| Field | Value | Notes |
|-------|-------|-------|
| **Realm** | `entra` | Unique identifier (lowercase, no special chars) |
| **Issuer URL** | `https://login.microsoftonline.com/<TENANT-ID>/v2.0` | Replace `<TENANT-ID>` with actual tenant ID |
| **Client ID** | `<APPLICATION-ID>` | From Step 1.3 |
| **Client Key** | `<CLIENT-SECRET>` | From Step 1.5 |
| **Scopes** | `email openid profile` | Default scopes (leave as-is) |
| **Username Claim** | `email` | Recommended for consistency |
| **Autocreate Users** | ✅ **Checked** | Automatically creates user accounts |
| **Default** | ❌ **Unchecked** | Keep local auth as default initially |
| **Comment** | `Microsoft Entra ID SSO` | User-friendly description |

3. Click **Add** to save the configuration

### Step 4: Configure Permissions and Access Control

#### 4.1 Create Proxmox Groups for Access Management
1. Navigate to **Datacenter** > **Permissions** > **Groups**
2. Click **Create**
3. Create groups as needed:
   - **Group ID:** `ProxmoxAdmins`
   - **Comment:** `Administrators via Entra ID`

#### 4.2 Assign Permissions to Groups
1. Navigate to **Datacenter** > **Permissions**
2. Click **Add** > **Group Permission**
3. Configure permission:
   - **Path:** `/` (root level)
   - **Group:** `ProxmoxAdmins`
   - **Role:** `Administrator` (or appropriate role)
4. Click **Add**

#### 4.3 Map Entra ID Groups (Optional)
If using group claims, users can be automatically assigned to Proxmox groups based on their Entra ID group membership. This requires additional configuration in the realm settings.

### Step 5: Testing and Validation

#### 5.1 Test SSO Login
1. Open new incognito/private browser window
2. Navigate to Proxmox web interface
3. Select realm: **Microsoft Entra ID SSO** from dropdown
4. Click **Login**
5. Should redirect to Microsoft login page
6. Enter organizational credentials
7. Should redirect back to Proxmox dashboard

#### 5.2 Verify User Creation
1. After successful login, navigate to **Datacenter** > **Permissions** > **Users**
2. Verify new user was created with format: `username@entra`

#### 5.3 Test Permission Assignment
1. Verify the SSO user has appropriate permissions
2. Test access to various Proxmox functions based on assigned roles

## Troubleshooting

### Common Issues and Solutions

#### Issue: "Invalid redirect URI"
**Solution:** Verify the redirect URI in Entra ID exactly matches Proxmox URL including port 8006

#### Issue: "User not found" after successful authentication
**Solution:** Ensure "Autocreate Users" is enabled in the realm configuration

#### Issue: Users lack permissions after login
**Solution:** 
- Verify user is assigned to appropriate Proxmox groups
- Check group permissions are correctly configured
- Consider using Entra ID group claims for automatic assignment

#### Issue: SSL/TLS certificate warnings
**Solution:** 
- Install valid SSL certificate on Proxmox
- Update Entra ID redirect URI if hostname changes

### Verification Commands

```bash
# Check Proxmox authentication logs
tail -f /var/log/daemon.log | grep pveproxy

# Test OIDC endpoint connectivity
curl -I "https://login.microsoftonline.com/<TENANT-ID>/v2.0/.well-known/openid_configuration"
```

## Post-Implementation Tasks

### Security Hardening
- [ ] Review and limit API permissions in Entra ID app registration
- [ ] Implement conditional access policies for Proxmox access
- [ ] Regular audit of SSO user accounts and permissions
- [ ] Monitor authentication logs for suspicious activity

### Documentation Updates
- [ ] Update user onboarding documentation
- [ ] Create troubleshooting guide for end users
- [ ] Document group mapping procedures

### Ongoing Maintenance
- [ ] Schedule periodic review of client secret expiration
- [ ] Plan renewal process for certificates and secrets
- [ ] Monitor Entra ID and Proxmox version compatibility

## Configuration Summary

| Component | Configuration | Status |
|-----------|---------------|---------|
| Entra ID App Registration | ✅ Created | Complete |
| API Permissions | ✅ Configured | Complete |
| Client Secret | ✅ Generated | Complete |
| Group Claims | ✅ Enabled | Complete |
| Proxmox OIDC Realm | ✅ Configured | Complete |
| User Auto-creation | ✅ Enabled | Complete |
| Permission Groups | ✅ Created | Complete |
| SSO Testing | ⏳ Pending | In Progress |

## Related Documentation

- [Proxmox User Management](../Runbooks/Configure-Services.md)
- [Network Architecture](../Architecture/Networking.md)
- [Security Policies](../Decisions/)

## Next Steps

1. **Immediate:**
   - Complete SSO testing with multiple user accounts
   - Validate group membership synchronization
   - Document user experience and login flow

2. **Short-term:**
   - Implement conditional access policies
   - Configure monitoring and alerting for authentication failures
   - Create user training materials

3. **Long-term:**
   - Evaluate integration with other homelab services
   - Consider implementing privileged access management
   - Assess feasibility of extending SSO to console access

---

**Build Log Status:** 🟡 In Progress  
**Next Review Date:** February 10, 2026  
**Success Criteria:** Users can successfully authenticate to Proxmox using Entra ID credentials with appropriate permissions
