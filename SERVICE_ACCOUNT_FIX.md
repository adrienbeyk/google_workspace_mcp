# 🔧 Service Account Authentication Fix

## Problem

The MCP server was trying to use **OAuth flow** for service accounts, which caused this error:
```
ValueError: Client secrets must be for a web or installed app.
```

Service accounts (emails ending in `.gserviceaccount.com`) don't use OAuth - they authenticate **directly** with their JSON key file.

---

## Solution

Added service account detection in `auth/google_auth.py` (line 837-871):

```python
# Check if this is a service account
is_service_account = user_google_email.endswith('.gserviceaccount.com')

if is_service_account:
    # Use direct service account authentication
    credentials = service_account.Credentials.from_service_account_file(
        service_account_file,
        scopes=required_scopes
    )
else:
    # Use OAuth flow for regular users
    credentials = get_credentials(...)
```

---

## How It Works

### For Service Accounts:
1. ✅ Detects email ending in `.gserviceaccount.com`
2. ✅ Loads credentials from JSON key file
3. ✅ Authenticates directly (no OAuth)
4. ✅ Returns authenticated service

### For Regular Users:
1. ✅ Uses existing OAuth flow
2. ✅ Generates auth URL
3. ✅ User clicks link to authorize
4. ✅ Returns authenticated service

---

## Configuration

The service account file is loaded from (in order):
1. `GOOGLE_CLIENT_SECRET_PATH` env var ✅ (set in test_setup.sh)
2. `GOOGLE_APPLICATION_CREDENTIALS` env var
3. `CONFIG_CLIENT_SECRETS_PATH` (fallback)

Current setup:
```bash
export GOOGLE_CLIENT_SECRET_PATH="../ChatGPT-Telegram-Bot/auth/viventium-bot-e4b7017af081.json"
export USER_GOOGLE_EMAIL="viventium-bot-mcp@viventium-bot.iam.gserviceaccount.com"
```

---

## Testing

### Restart MCP Server:
```bash
cd google_workspace_mcp
./test_setup.sh
```

### Expected Output:
```
[INFO] Detected service account: viventium-bot-mcp@viventium-bot.iam.gserviceaccount.com
[INFO] Loading service account from: ../ChatGPT-Telegram-Bot/auth/viventium-bot-e4b7017af081.json
[INFO] Service account authenticated successfully
```

### In Telegram:
```
/mcp_list
```

Should show:
```
📋 Available MCP Tools (11 total):

1. read_sheet_values
2. modify_sheet_values
3. create_spreadsheet
...
```

Then test:
```
Read this spreadsheet: 
https://docs.google.com/spreadsheets/d/1_cha666KzOfVZEw4t9eu4P8iBiuRllAmKkxCpCBSsYU
```

Should work directly without OAuth prompts! ✅

---

## Files Modified

- **`auth/google_auth.py`** - Added service account detection (lines 837-871)

---

## Benefits

✅ **No OAuth needed** for service accounts
✅ **Instant authentication** (no browser clicks)
✅ **Works in automation** (bots, scripts, CI/CD)
✅ **Still supports regular users** (OAuth flow intact)

---

**Status:** ✅ Fixed and ready to test!
