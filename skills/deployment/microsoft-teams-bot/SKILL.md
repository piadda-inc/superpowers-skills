---
name: Microsoft Teams Bot Deployment
description: Deploy conversational bots to Microsoft Teams using Azure Bot Service and avoid multi-tenant deprecation, auth failures, and state management issues
when_to_use: when deploying a conversational bot to Microsoft Teams with Azure Bot Service, Cloud Run, or handling Bot Framework authentication errors
version: 1.0.0
languages: all
dependencies: Azure Portal, GCP Cloud Run (optional), Bot Framework SDK
---

# Microsoft Teams Bot Deployment

## Overview

Microsoft Teams bots require precise Azure Bot Service configuration. As of July 31, 2025, multi-tenant bot registration is deprecated, requiring single-tenant configuration with explicit tenant IDs.

**Core principle:** Teams bots need three secrets (App ID, Password, Tenant ID) and single-tenant authentication to work correctly.

## When to Use

Use this skill when:
- Deploying a new Teams bot with Bot Framework
- Getting "Unauthorized" or "401" errors from Teams
- Migrating from multi-tenant to single-tenant bot configuration
- Integrating Teams bot with cloud platforms (GCP, AWS, Azure)
- Bot worked in development but fails in production

**Don't use for:**
- Teams webhooks only (no Bot Framework adapter)
- Teams app packages without conversational bot functionality

## Critical Gotchas

### 1. Multi-Tenant Bot Deprecation (CRITICAL)

**As of July 31, 2025, multi-tenant bots are deprecated.**

If you see in Azure App Registration:
- `"signInAudience": "AzureADMultipleOrgs"` → DEPRECATED
- Supported account types: "Accounts in any organizational directory" → DEPRECATED

**Required:**
- `"signInAudience": "AzureADMyOrg"` → Single-tenant only
- Supported account types: "Accounts in this organizational directory only"

**Why this matters:** Multi-tenant authentication no longer works. Bot will return Unauthorized errors even with correct App ID and Password.

### 2. Three Required Secrets

Teams bots need **three** secrets, not two:

1. **TEAMS_BOT_APP_ID** - Application (client) ID from Azure App Registration
2. **TEAMS_BOT_APP_PASSWORD** - Client secret value (not secret ID)
3. **TEAMS_BOT_TENANT_ID** - Azure tenant ID (NEW requirement for single-tenant)

**Common mistake:** Assuming only App ID and Password are needed. Without Tenant ID, single-tenant bots fail authentication.

### 3. Bot Framework Adapter Configuration

The `BotFrameworkAdapterSettings` MUST include `channel_auth_tenant`:

```python
# ❌ WRONG: Multi-tenant (deprecated)
settings = BotFrameworkAdapterSettings(
    app_id=app_id,
    app_password=app_password,
    channel_auth_tenant="common"  # Multi-tenant - NO LONGER WORKS
)

# ✅ CORRECT: Single-tenant
settings = BotFrameworkAdapterSettings(
    app_id=app_id,
    app_password=app_password,
    channel_auth_tenant=app_tenant_id  # Actual tenant ID required
)
```

**Validation:** Always check that `TEAMS_BOT_TENANT_ID` is set and not empty.

### 4. Conversation State Management

For stateful bots (multi-turn conversations), validate state at **load time**, not just save time:

```python
def get_state(self, conversation_id: str):
    """Retrieve conversation state"""
    doc = firestore_ref.get()

    if doc.exists:
        state = doc.to_dict().get("state")

        # ✅ Validate at load time to prevent corrupted state
        if state and "messages" in state:
            if len(state["messages"]) > 20:
                # State corrupted/oversized - delete and start fresh
                self.delete_state(conversation_id)
                return None

        return state

    return None
```

**Why:** Conversation state can grow unbounded across turns. Corrupted state from failed operations can cause payload size errors (11+ MB) when sending Adaptive Cards.

## Quick Reference Checklist

**Azure Portal Setup:**
- [ ] App Registration exists in Azure Portal
- [ ] Changed to single-tenant: "Accounts in this organizational directory only"
- [ ] Generated client secret (save the VALUE, not the ID)
- [ ] Copied Tenant ID from Azure Portal → Microsoft Entra ID → Overview
- [ ] Bot Channels Registration configured with App ID

**Secret Management (GCP example):**
```bash
# Create three secrets
echo -n "YOUR_APP_ID" | gcloud secrets create teams-bot-app-id --data-file=-
echo -n "YOUR_CLIENT_SECRET" | gcloud secrets create teams-bot-app-password --data-file=-
echo -n "YOUR_TENANT_ID" | gcloud secrets create teams-bot-tenant-id --data-file=-

# Grant access to service account
gcloud secrets add-iam-policy-binding teams-bot-app-id \
  --member="serviceAccount:SERVICE_ACCOUNT" \
  --role="roles/secretmanager.secretAccessor"
# Repeat for app-password and tenant-id
```

**Code Configuration:**
- [ ] Bot Framework adapter uses single-tenant config
- [ ] `channel_auth_tenant=app_tenant_id` (not "common")
- [ ] All three environment variables loaded
- [ ] Validation throws error if tenant ID missing
- [ ] Conversation state validated at load time (if stateful)

**Deployment Verification:**
```bash
# Check logs for auth errors
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=YOUR_BOT" --limit 50

# Common error patterns:
# ❌ "Unauthorized" → Check single-tenant config
# ❌ "Permission denied on secret" → Check IAM bindings
# ❌ "Payload size exceeds limit" → Check state validation
```

## Implementation Steps

### Step 1: Configure Azure App Registration

1. Go to [Azure Portal](https://portal.azure.com) → Microsoft Entra ID
2. Copy **Tenant ID** from Overview (you'll need this later)
3. Go to App registrations → Find your bot app
4. Click **Authentication** → Change to:
   - Supported account types: "Accounts in this organizational directory only (Single tenant)"
5. Click **Certificates & secrets** → New client secret
   - Save the **Value** (not Secret ID) - shown once only
6. Click **Overview** → Copy **Application (client) ID**

### Step 2: Store Secrets

Store three secrets in your secret manager (GCP/AWS/Azure):
- `teams-bot-app-id` = Application (client) ID
- `teams-bot-app-password` = Client secret VALUE
- `teams-bot-tenant-id` = Tenant ID

### Step 3: Configure Bot Framework Adapter

```python
import os
from botbuilder.core import BotFrameworkAdapter, BotFrameworkAdapterSettings

def create_bot_adapter():
    # Load from environment (populated from secret manager)
    app_id = os.getenv("TEAMS_BOT_APP_ID")
    app_password = os.getenv("TEAMS_BOT_APP_PASSWORD")
    app_tenant_id = os.getenv("TEAMS_BOT_TENANT_ID")

    # Validate all three are present
    if not app_id or not app_password:
        raise ValueError("TEAMS_BOT_APP_ID and TEAMS_BOT_APP_PASSWORD required")

    if not app_tenant_id:
        raise ValueError("TEAMS_BOT_TENANT_ID required for single-tenant auth")

    # Single-tenant configuration
    settings = BotFrameworkAdapterSettings(
        app_id=app_id,
        app_password=app_password,
        channel_auth_tenant=app_tenant_id  # Critical: actual tenant ID
    )

    return BotFrameworkAdapter(settings)
```

### Step 4: Add State Validation (If Stateful)

If your bot maintains conversation state across turns:

```python
from typing import Dict, Any, Optional

class ConversationStateManager:
    def get_state(self, conversation_id: str) -> Optional[Dict[str, Any]]:
        """Retrieve conversation state with validation"""
        doc = self.collection.document(conversation_id).get()

        if doc.exists:
            state = doc.to_dict().get("state")

            # Prevent corrupted/oversized state
            if state and "messages" in state:
                if len(state["messages"]) > 20:
                    print(f"⚠️ State has {len(state['messages'])} messages - clearing")
                    self.delete_state(conversation_id)
                    return None

            return state

        return None
```

**Why:** Prevents payload size errors when sending Adaptive Cards with corrupted conversation state.

### Step 5: Deploy and Verify

```bash
# Deploy to cloud
./deploy.sh  # or your deployment command

# Check logs for errors
gcloud logging read "resource.type=cloud_run_revision" --limit 50

# Test in Teams
# Send message to bot in Teams
# Check for "Unauthorized" errors in logs
```

## Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Using multi-tenant config | `Unauthorized` / `401` errors | Change App Registration to single-tenant |
| Missing tenant ID | `Unauthorized` / `401` errors | Add `TEAMS_BOT_TENANT_ID` to secrets |
| `channel_auth_tenant="common"` | `Unauthorized` in production | Use actual tenant ID, not "common" |
| Only two secrets configured | `ValueError` on startup | Add `teams-bot-tenant-id` secret |
| No state validation at load | `Payload size exceeds limit` | Add validation in `get_state()` |
| Client secret ID instead of value | `Unauthorized` | Use secret VALUE from Azure Portal |
| Wrong service account permissions | `Permission denied on secret` | Grant `secretAccessor` role |

## Multi-Tenant Migration Path

If you have an existing multi-tenant bot (before July 31, 2025):

1. **Find tenant ID:** Azure Portal → Microsoft Entra ID → Overview
2. **Change App Registration:** Authentication → Single tenant
3. **Add tenant ID secret:** Store in secret manager
4. **Update adapter config:** Add `channel_auth_tenant=tenant_id`
5. **Redeploy:** With all three secrets configured
6. **Test:** Verify no Unauthorized errors

## Platform-Specific Notes

### GCP Cloud Run

```bash
# Deploy with secrets
gcloud run deploy BOT_NAME \
  --image gcr.io/PROJECT/BOT_NAME \
  --update-secrets TEAMS_BOT_APP_ID=teams-bot-app-id:latest,TEAMS_BOT_APP_PASSWORD=teams-bot-app-password:latest,TEAMS_BOT_TENANT_ID=teams-bot-tenant-id:latest
```

### AWS Lambda

Use AWS Secrets Manager and load in Lambda initialization:

```python
import boto3
import json

def get_secret(secret_name):
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])

# Load secrets
bot_secrets = get_secret('teams-bot-credentials')
app_id = bot_secrets['app_id']
app_password = bot_secrets['app_password']
app_tenant_id = bot_secrets['tenant_id']
```

### Azure App Service

Use Azure App Configuration or Key Vault:

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

credential = DefaultAzureCredential()
client = SecretClient(vault_url="https://YOUR_VAULT.vault.azure.net/", credential=credential)

app_id = client.get_secret("teams-bot-app-id").value
app_password = client.get_secret("teams-bot-app-password").value
app_tenant_id = client.get_secret("teams-bot-tenant-id").value
```

## References

- [Microsoft: Multi-tenant bot deprecation](https://github.com/microsoft/botframework-sdk/issues/6693)
- [Azure: Provision and publish a bot](https://learn.microsoft.com/en-us/azure/bot-service/provision-and-publish-a-bot)
- [Azure: Find your tenant ID](https://learn.microsoft.com/en-us/azure/azure-portal/get-subscription-tenant-id)
- [Bot Framework: Single-tenant apps](https://learn.microsoft.com/en-us/azure/bot-service/bot-builder-authentication)

## Real-World Impact

From production deployment (October 2025):
- Multi-tenant config → Unauthorized errors 100% of requests
- Single-tenant with tenant ID → Successful authentication
- State validation prevented 11MB payload errors
- Three-secret configuration required for all environments
