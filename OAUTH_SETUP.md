# 🔐 OAuth Setup for Regular Users

## ✅ Current Status

**Service Account:** ✅ Working!
**OAuth for Users:** ✅ Already implemented, just needs OAuth client setup!

---

## How It Works

The server **automatically detects** authentication type:

```python
Email: "viventium-bot-mcp@...gserviceaccount.com" → Service Account (auto)
Email: "user@gmail.com"                           → OAuth (user clicks link)
```

---

## Setup OAuth for Regular Users

### 1. Create OAuth Client ID

1. Go to [Google Cloud Console](https://console.cloud.google.com/apis/credentials)
2. Select your project (or create one)
3. Click **"Create Credentials"** → **"OAuth client ID"**
4. Choose **"Web application"**
5. Set **Authorized redirect URIs:**
   ```
   http://localhost:8000/oauth2callback
   http://127.0.0.1:8000/oauth2callback
   ```
6. Click **"Create"**
7. **Download the JSON file** (client_secret_xxx.json)

### 2. Configure Environment

Add to your `.env` or `test_setup.sh`:

```bash
# OAuth Client Credentials
export GOOGLE_OAUTH_CLIENT_ID="your-client-id.apps.googleusercontent.com"
export GOOGLE_OAUTH_CLIENT_SECRET="your-client-secret"

# OR use the JSON file path
export GOOGLE_CLIENT_SECRET_PATH="/path/to/client_secret_xxx.json"
```

### 3. Enable APIs

In [Google Cloud Console](https://console.cloud.google.com/apis/library):
- ✅ Google Sheets API
- ✅ Google Calendar API (if needed)
- ✅ Gmail API (if needed)
- ✅ Google Drive API (if needed)

---

## Testing OAuth

### Start Server
```bash
cd google_workspace_mcp
./test_setup.sh
```

### Test with Regular User Email

In your bot/Telegram, use a regular Gmail address:

```
Read spreadsheet with user@gmail.com:
https://docs.google.com/spreadsheets/d/xxx
```

**Expected Flow:**

1. Server detects it's NOT a service account
2. Generates OAuth URL
3. Bot returns message with auth link
4. User clicks link → Google asks for permissions
5. User approves → redirected to callback
6. Credentials stored → next time auto-authenticated!

**Example Response:**
```
🔐 Authentication Required for Google Sheets

Please click this link to authorize:
https://accounts.google.com/o/oauth2/auth?...

After authorizing, try your request again!
```

---

## Current Config Summary

### Service Account (Automated)
```bash
USER_GOOGLE_EMAIL=viventium-bot-mcp@viventium-bot.iam.gserviceaccount.com
GOOGLE_CLIENT_SECRET_PATH=../ChatGPT-Telegram-Bot/auth/viventium-bot-e4b7017af081.json
```
✅ Works for sheets shared with service account
✅ No user interaction needed

### OAuth (User-based)
```bash
GOOGLE_OAUTH_CLIENT_ID=your-id.apps.googleusercontent.com
GOOGLE_OAUTH_CLIENT_SECRET=your-secret
```
✅ Works for user's private sheets
✅ User clicks auth link once
✅ Credentials cached for future use

### API Key (Public Only)
```bash
GOOGLE_API_KEY=your-api-key
```
✅ Works for public/readonly sheets
✅ No auth needed
✅ Fastest option

---

## Multi-User Support

The server supports **multiple users simultaneously**:

1. **Service account** → Always uses service account auth
2. **user1@gmail.com** → Stores OAuth credentials for user1
3. **user2@gmail.com** → Stores OAuth credentials for user2

Each user's credentials are stored separately in:
```
~/.google_workspace_mcp/credentials/
```

---

## Quick Test Commands

### Service Account (Should work now!)
```
Read this sheet: https://docs.google.com/spreadsheets/d/1_cha666KzOfVZEw4t9eu4P8iBiuRllAmKkxCpCBSsYU
```

### OAuth User (Needs OAuth client setup)
```
/mcp_call start_google_auth user@gmail.com sheets
```

Should return auth URL if OAuth client is configured!

---

## Troubleshooting

### "Client secrets must be for a web or installed app"
- ❌ Using service account JSON for OAuth
- ✅ Create **OAuth client ID** (separate from service account)

### "OAuth callback unavailable"
- ❌ Server not running on correct port
- ✅ Check server is on http://localhost:8000

### "Invalid redirect URI"
- ❌ Redirect URI not added to Google Console
- ✅ Add http://localhost:8000/oauth2callback

---

**Status:** OAuth is already coded and ready! Just need OAuth client credentials! 🚀
