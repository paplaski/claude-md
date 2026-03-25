# Gmail API Access Research: Official Anthropic & Google Tools

## Summary of Options

There are **4 main approaches** to safely access Gmail, ordered by ease of use:

| Approach | Provider | Auth Method | Read | Write/Send | Best For |
|----------|----------|-------------|------|------------|----------|
| Claude Google Workspace Connector | Anthropic (built-in) | OAuth via Claude UI | Yes | Yes (draft) | Claude Desktop/Web users |
| Google's Official Gmail MCP Server | Google | OAuth 2.0 | Yes | Yes | Claude Code / MCP-compatible tools |
| Community Gmail MCP Servers | Third-party | OAuth 2.0 | Yes | Yes | Claude Code with custom setup |
| Gmail API Directly | Google | OAuth 2.0 | Yes | Yes | Custom applications |

---

## Option 1: Claude's Built-in Google Workspace Connector (Recommended for Claude Users)

Anthropic offers **official Google Workspace connectors** built into Claude (claude.ai and Claude Desktop). This is the simplest and most secure approach.

### What It Supports
- **Search and read emails** using natural language
- **Draft emails** with proper formatting and context
- **Google Calendar** and **Google Drive** access also available

### How to Set Up
1. Go to **claude.ai** → Settings → Integrations (or Claude Desktop → Settings)
2. Click **Connect Google Workspace**
3. Authenticate with your Google account via OAuth
4. Grant the requested permissions

### Security Notes
- Uses Google's standard OAuth 2.0 consent flow
- Anthropic acts as the OAuth client — your credentials are never shared with Claude directly
- You can revoke access at any time from Google Account → Security → Third-party apps
- Data is handled per Anthropic's privacy policy

### Limitations
- Only available in Claude web (claude.ai) and Claude Desktop — **not available in Claude Code CLI**
- Cannot run fully automated/programmatic workflows

**Reference:** https://support.claude.com/en/articles/10166901-use-google-workspace-connectors

---

## Option 2: Google's Official MCP Server for Gmail

Google has released **official MCP (Model Context Protocol) servers** for their Workspace products including Gmail.

### Google Workspace MCP Server
- **Repository:** `github.com/nichochar/google-mcp` (Google-endorsed) and Google's own toolbox (`github.com/googleapis/genai-toolbox`)
- **Supported services:** Gmail, Google Calendar, Google Drive, Google Docs
- **Auth:** OAuth 2.0 with Google Cloud project credentials

### Setup for Claude Code
1. **Create a Google Cloud Project** at https://console.cloud.google.com
2. **Enable the Gmail API** in your project
3. **Create OAuth 2.0 credentials** (Desktop application type)
4. **Download** the `credentials.json` file
5. **Install and configure** the MCP server:

```json
// In your Claude Code MCP settings (~/.claude/settings.json or project .mcp.json)
{
  "mcpServers": {
    "gmail": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/google-workspace-mcp"],
      "env": {
        "GOOGLE_CLIENT_ID": "<your-client-id>",
        "GOOGLE_CLIENT_SECRET": "<your-client-secret>",
        "GOOGLE_REDIRECT_URI": "http://localhost:3000/oauth/callback"
      }
    }
  }
}
```

> **Note:** Verify the exact package name at the time of setup. The MCP ecosystem is evolving rapidly. Check https://github.com/modelcontextprotocol/servers for the latest official server list.

### Gmail API Scopes (Least Privilege)

| Scope | Access Level |
|-------|-------------|
| `gmail.readonly` | Read-only access to all mail |
| `gmail.labels` | Manage labels only |
| `gmail.send` | Send emails only (no read) |
| `gmail.compose` | Create, read, update drafts and send |
| `gmail.modify` | Read, send, delete, manage (not permanent delete) |
| `gmail.metadata` | Read metadata (headers, labels) only — no body |
| `mail.google.com` | Full access (avoid unless necessary) |

**Best practice:** Start with `gmail.readonly` and `gmail.send` if you need to read and send. Avoid `mail.google.com` (full access) unless absolutely required.

**Reference:** https://developers.google.com/workspace/gmail/api/auth/scopes

---

## Option 3: Community MCP Servers for Gmail

Several well-maintained community MCP servers exist:

### Notable Options

1. **GongRzhe/Gmail-MCP-Server** (GitHub)
   - Open-source, popular
   - Auto-authentication support
   - Supports read, send, search, label management

2. **Composio Gmail MCP**
   - Managed OAuth2 — handles token refresh automatically
   - Easier setup than self-hosted
   - https://composio.dev/toolkits/gmail/framework/claude-code

3. **Zapier Gmail MCP**
   - No-code setup
   - Connects Gmail actions through Zapier's platform
   - Good for simple automations

### Security Considerations for Community Servers
- **Review the source code** before using any community MCP server
- Ensure the server doesn't store or transmit your OAuth tokens to third parties
- Prefer servers with active maintenance and community review
- Check that tokens are stored locally, not on remote servers

---

## Option 4: Gmail API Directly (For Custom Applications)

If you're building your own application or script to access Gmail.

### Setup Steps

1. **Google Cloud Console Setup**
   ```
   1. Go to https://console.cloud.google.com
   2. Create a new project (or select existing)
   3. Navigate to APIs & Services → Library
   4. Search for "Gmail API" → Enable
   5. Go to APIs & Services → Credentials
   6. Create Credentials → OAuth 2.0 Client ID
   7. Configure consent screen (External or Internal)
   8. Download credentials.json
   ```

2. **Python Example (Official Google Client Library)**
   ```python
   from google.oauth2.credentials import Credentials
   from google_auth_oauthlib.flow import InstalledAppFlow
   from googleapiclient.discovery import build
   import base64
   from email.mime.text import MIMEText

   SCOPES = ['https://www.googleapis.com/auth/gmail.readonly',
             'https://www.googleapis.com/auth/gmail.send']

   def get_gmail_service():
       creds = None
       # token.json stores the user's access and refresh tokens
       if os.path.exists('token.json'):
           creds = Credentials.from_authorized_user_file('token.json', SCOPES)
       if not creds or not creds.valid:
           if creds and creds.expired and creds.refresh_token:
               creds.refresh(Request())
           else:
               flow = InstalledAppFlow.from_client_secrets_file(
                   'credentials.json', SCOPES)
               creds = flow.run_local_server(port=0)
           with open('token.json', 'w') as token:
               token.write(creds.to_json())
       return build('gmail', 'v1', credentials=creds)

   # Read emails
   def list_messages(service, query='is:unread'):
       results = service.users().messages().list(
           userId='me', q=query, maxResults=10).execute()
       return results.get('messages', [])

   # Send email
   def send_email(service, to, subject, body):
       message = MIMEText(body)
       message['to'] = to
       message['subject'] = subject
       raw = base64.urlsafe_b64encode(
           message.as_bytes()).decode()
       service.users().messages().send(
           userId='me', body={'raw': raw}).execute()
   ```

3. **Install Dependencies**
   ```bash
   pip install google-api-python-client google-auth-oauthlib google-auth-httplib2
   ```

### Node.js Alternative
```bash
npm install googleapis @google-cloud/local-auth
```

### Official Client Libraries
- **Python:** `google-api-python-client`
- **Node.js:** `googleapis` (npm)
- **Java:** `google-api-services-gmail`
- **Go:** `google.golang.org/api/gmail/v1`
- **.NET:** `Google.Apis.Gmail.v1`

**Reference:** https://developers.google.com/workspace/gmail/api/quickstart/python

---

## Security Best Practices

### Authentication
- **Always use OAuth 2.0** — never use API keys for Gmail (API keys don't work for user data)
- **Never use service accounts** for personal Gmail — service accounts are for Google Workspace domain-wide delegation only
- Store `credentials.json` and `token.json` securely — **never commit them to git**
- Add to `.gitignore`:
  ```
  credentials.json
  token.json
  *.json.bak
  ```

### Principle of Least Privilege
- Request only the scopes you actually need
- `gmail.readonly` + `gmail.send` covers most use cases
- Avoid `mail.google.com` (full access) unless you need to delete/permanently modify emails
- Review and audit granted scopes periodically

### Token Management
- OAuth tokens include a **refresh token** — treat it like a password
- Store tokens in a secure location (encrypted storage, OS keychain, or environment variables)
- Implement token rotation and revocation capabilities
- Set token expiry handling in your code

### Application Verification
- For apps accessing >100 users, Google requires **OAuth app verification**
- For personal use, you can use the app in "testing" mode (limited to test users you add)
- Testing mode tokens expire every 7 days

### Network Security
- Always use HTTPS (Google APIs enforce this)
- Use the latest versions of Google client libraries (they handle TLS properly)
- If running an MCP server locally, bind to `localhost` only — never expose to the network

### Revocation
- You can revoke app access at any time: https://myaccount.google.com/permissions
- Programmatic revocation: `POST https://oauth2.googleapis.com/revoke?token={token}`

---

## Recommendation

| Use Case | Recommended Approach |
|----------|---------------------|
| Casual email access with Claude | **Option 1:** Claude Workspace Connector |
| Claude Code / automated workflows | **Option 2:** Google's Official MCP Server |
| Quick integration, minimal setup | **Option 3:** Composio or Zapier MCP |
| Custom application development | **Option 4:** Gmail API directly |

For most users, **start with Option 1** (Claude's built-in connector) if you're using Claude Desktop or claude.ai. If you need programmatic access in Claude Code, **Option 2** (Google's official MCP server) is the most secure and maintained choice.

---

*Research compiled: 2026-03-25*
*Sources: Google Developers documentation, Anthropic support docs, MCP servers repository, community MCP registries*
