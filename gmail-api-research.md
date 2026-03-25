# Gmail API Access Research: Official Anthropic & Google Tools

## Summary of Options

There are **4 main approaches** to safely access Gmail, ordered by ease of use:

| Approach | Provider | Auth Method | Read | Write/Send | Best For |
|----------|----------|-------------|------|------------|----------|
| Claude Google Workspace Connector | Anthropic (built-in) | OAuth via Claude UI | Yes | Drafts only (no send) | Claude Desktop/Web users |
| Community Google Workspace MCP Servers | Community | OAuth 2.0 | Yes | Yes | Claude Code / MCP-compatible tools |
| Third-party MCP Platforms | Composio/Zapier | OAuth 2.0 (managed) | Yes | Yes | No-code / quick setup |
| Gmail API Directly | Google | OAuth 2.0 | Yes | Yes | Custom applications |

**Important clarification:** Google's official MCP servers (March 2026) currently cover **Cloud services only** (BigQuery, GKE, Maps) — **not Gmail/Workspace**. There is no official Google Gmail MCP server yet. For MCP-based Gmail access, community servers are the current option.

---

## Option 1: Claude's Built-in Google Workspace Connector (Recommended for Claude Users)

Anthropic offers **official Google Workspace connectors** built into Claude (claude.ai and Claude Desktop). This is the simplest and most secure approach.

### What It Supports
- **Search and read emails** using natural language
- **Create drafts** with proper formatting and context (user must manually send)
- **Google Calendar** and **Google Drive** access also available
- Available on **Max, Team, Enterprise, and Pro plans**

### What It Cannot Do
- **Cannot send emails** — all sends must be done manually by the user
- **Cannot access attachment content** (metadata only)
- **Cannot see embedded images**

### How to Set Up
1. Go to **claude.ai** → Settings → Integrations (or Claude Desktop → Settings)
2. Click **Connect Google Workspace**
3. Authenticate with your Google account via OAuth
4. Grant the requested permissions

### Security Notes
- Uses Google's standard OAuth 2.0 consent flow
- Anthropic acts as the OAuth client — your credentials are never shared with Claude directly
- Anthropic does **not** train models on connector data
- Connections are authenticated per-user with strict access controls
- You can revoke access at any time from Google Account → Security → Third-party apps
- Data is handled per Anthropic's privacy policy

### Limitations
- Only available in Claude web (claude.ai) and Claude Desktop — **not available in Claude Code CLI**
- Cannot run fully automated/programmatic workflows
- Cannot send emails directly (drafts only)

**Reference:** https://support.claude.com/en/articles/10166901-use-google-workspace-connectors

---

## Option 2: Google's Official MCP Servers (Status Update)

Google announced official MCP server support in early 2026. However, these currently cover **Google Cloud services only**:

### Currently Available (Google Official)
- **Maps / Grounding Lite** — geospatial, places, weather, routing
- **BigQuery** — schema interpretation, query execution
- **GKE** — Kubernetes cluster management

### Planned (No Timeline for Gmail)
- Cloud Run, Cloud Storage, Cloud Resource Manager, AlloyDB, Cloud SQL, Spanner, Looker, Pub/Sub, Dataplex

**Gmail/Workspace is NOT yet covered** by Google's official MCP servers. For MCP-based Gmail access, see the community options below.

---

## Option 3: Community Google Workspace MCP Servers (Best for Claude Code)

Since there is no official Google Gmail MCP server yet, community-built servers are the primary option for Claude Code users.

### Top Options

1. **taylorwilsdon/google_workspace_mcp** (Most Feature-Complete)
   - Covers Gmail, Drive, Docs, Sheets, Calendar, Slides, Chat, Forms, Tasks, Contacts (12 services, 100+ tools)
   - Supports OAuth 2.1, multi-user remote hosting
   - One-click Claude Desktop installation
   - Website: workspacemcp.com

2. **GongRzhe/Gmail-MCP-Server** (Gmail-Specific)
   - Focused Gmail MCP server for Claude Desktop
   - Auto-authentication support
   - Supports sending, reading, and searching emails

3. **j3k0/mcp-google-workspace** (Lightweight)
   - Node.js MCP server for Gmail and Calendar
   - Install via `npx mcp-google-workspace`

4. **MarkusPfundstein/mcp-gsuite**
   - Gmail and Calendar MCP server
   - Flexible search and batch retrieval

### Setup for Claude Code (Example with community server)
1. **Create a Google Cloud Project** at https://console.cloud.google.com
2. **Enable the Gmail API** in your project
3. **Create OAuth 2.0 credentials** (Desktop application type)
4. **Download** the `credentials.json` file
5. **Install and configure** the MCP server in your Claude Code settings:

```json
// In ~/.claude/settings.json or project .mcp.json
{
  "mcpServers": {
    "google-workspace": {
      "command": "npx",
      "args": ["-y", "mcp-google-workspace"],
      "env": {
        "GOOGLE_CLIENT_ID": "<your-client-id>",
        "GOOGLE_CLIENT_SECRET": "<your-client-secret>"
      }
    }
  }
}
```

> **Note:** Verify the exact package name and configuration for whichever server you choose. The MCP ecosystem is evolving rapidly.

### MCP Security Warning
Between January and February 2026, security researchers filed **over 30 CVEs** targeting MCP servers, clients, and infrastructure — primarily due to missing input validation, absent authentication, and blind trust in tool descriptions. **Always audit community MCP server code before granting it access to your email.**

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

## Option 4: Third-Party MCP Platforms (Managed)

For simpler setup without self-hosting:

1. **Composio Gmail MCP** — Managed OAuth2, handles token refresh automatically (composio.dev)
2. **Zapier Gmail MCP** — No-code setup, connects Gmail through Zapier's platform
3. **Merge** — Email MCP servers supporting Gmail, Outlook, and other providers

These platforms handle OAuth and token management for you, but your data flows through their servers.

---

## Option 5: Gmail API Directly (For Custom Applications)

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

### Authentication Methods Comparison

| Method | Use Case | Gmail Compatible? |
|--------|----------|-------------------|
| **OAuth 2.0 (user consent)** | Apps acting on behalf of a user | Yes — the standard and recommended method |
| **Service Accounts** | Server-to-server, no user interaction | Only with Google Workspace + domain-wide delegation |
| **API Keys** | Public data access only | **No** — Gmail requires authentication |

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
- **Access tokens** expire after ~1 hour — treat them as short-lived API keys
- **Refresh tokens** are long-lived and can mint new access tokens — **treat them like passwords**
- Store tokens encrypted at rest (Google Cloud Secret Manager, OS keychains, or hardware security modules)
- Never commit tokens to source control or transmit in plaintext
- Refresh tokens stop working if: user revokes access, token unused for 6 months, or user changes password
- There is a limit of **100 refresh tokens** per Google Account per OAuth client
- Implement token rotation and revocation capabilities

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
| Claude Code / automated workflows | **Option 3:** Community Workspace MCP (e.g., taylorwilsdon) |
| Quick integration, minimal setup | **Option 4:** Composio or Zapier MCP |
| Custom application development | **Option 5:** Gmail API directly |
| Enterprise / organizational use | Service accounts with domain-wide delegation, or wait for Google's official Workspace MCP |

For most users, **start with Option 1** (Claude's built-in connector) if you're using Claude Desktop or claude.ai. If you need programmatic access in Claude Code, **Option 3** (a well-maintained community MCP server) is the current best choice — but **audit the code** before granting it email access. For full custom control, **Option 5** (direct Gmail API) gives you the most flexibility.

---

*Research compiled: 2026-03-25*

### Sources
- Google: Choose Gmail API Scopes — developers.google.com/workspace/gmail/api/auth/scopes
- Google: OAuth 2.0 for Google APIs — developers.google.com/identity/protocols/oauth2
- Google: Gmail API Python Quickstart — developers.google.com/workspace/gmail/api/quickstart/python
- Google Cloud Blog: Official MCP Support for Google Services — cloud.google.com/blog
- Anthropic: Google Workspace Connectors — support.claude.com/en/articles/10166901
- MCP Servers Repository — github.com/modelcontextprotocol/servers
- taylorwilsdon/google_workspace_mcp — github.com/taylorwilsdon/google_workspace_mcp
- GongRzhe/Gmail-MCP-Server — github.com/GongRzhe/Gmail-MCP-Server
