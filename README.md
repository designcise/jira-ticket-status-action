# Jira Ticket Status Check Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Jira%20Ticket%20Status%20Check-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=github)](https://github.com/marketplace/actions/jira-ticket-status-check)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Block pull requests automatically when associated Jira tickets have blocking workflow statuses like "Blocked", "On Hold", or "Requires fixing".

Perfect for teams practicing trunk-based development or using feature branches where multiple commits reference different Jira tickets.

---

## ✨ Features

- 🔍 **Scans all commits** - Extracts ticket references from every commit message in the PR
- 🎯 **Smart deduplication** - Only checks each unique ticket once
- 🚫 **Configurable blocking** - Define which Jira statuses should block merging
- 🔓 **Bypass mechanism** - Emergency hotfixes can skip validation with keywords
- 📊 **Beautiful summaries** - See ticket status overview directly in GitHub Actions
- ⚡ **Fast and reliable** - Uses Jira REST API v3 with efficient queries
- 🔒 **Secure** - Uses GitHub secrets for credentials
- 🎨 **Flexible** - Works with any Jira ticket prefix (PROJ, JIRA, DEV, etc.)
- 🏷️ **Position agnostic** - Finds tickets anywhere in commit messages

---

## 🚀 Quick Start

### 1. Add to your workflow

Create `.github/workflows/jira-check.yml`:

```yaml
name: Jira Ticket Status Check
on:
  pull_request:
    types: [opened, edited, synchronize]

jobs:
  check-jira:
    runs-on: ubuntu-latest
    steps:
      - uses: designcise/jira-ticket-status-action@v1
        with:
          jira-base-url: ${{ secrets.JIRA_BASE_URL }}
          jira-email: ${{ secrets.JIRA_EMAIL }}
          jira-api-token: ${{ secrets.JIRA_API_TOKEN }}
          ticket-prefix: 'PROJ'  # Your Jira project prefix
```

### 2. Configure secrets

Go to **Settings → Secrets and variables → Actions** and add:

| Secret | Example Value | How to Get |
|--------|---------------|------------|
| `JIRA_BASE_URL` | `https://company.atlassian.net` | Your Jira instance URL |
| `JIRA_EMAIL` | `bot@company.com` | Email of Jira user |
| `JIRA_API_TOKEN` | `ATATT3xFfGF0...` | [Generate here](https://id.atlassian.com/manage-profile/security/api-tokens) |

### 3. Enable branch protection

Go to **Settings → Branches → Branch protection rules** and:
1. ✅ Enable "Require status checks to pass before merging"
2. ✅ Select `check-jira` (or your job name)

Done! 🎉

---

## 📖 Usage Examples

### Basic Usage

```yaml
- uses: designcise/jira-ticket-status-action@v1
  with:
    jira-base-url: ${{ secrets.JIRA_BASE_URL }}
    jira-email: ${{ secrets.JIRA_EMAIL }}
    jira-api-token: ${{ secrets.JIRA_API_TOKEN }}
    ticket-prefix: 'TEAM'
```

**Default blocked statuses:** `Requires fixing`, `Blocked`, `On Hold`, `Waiting for Dependency`

**Default bypass keyword:** `noticket`

---

### Custom Blocked Statuses

Define exactly which statuses should block PRs:

```yaml
- uses: designcise/jira-ticket-status-action@v1
  with:
    jira-base-url: ${{ secrets.JIRA_BASE_URL }}
    jira-email: ${{ secrets.JIRA_EMAIL }}
    jira-api-token: ${{ secrets.JIRA_API_TOKEN }}
    ticket-prefix: 'PROJ'
    blocked-statuses: '["Blocked", "Paused", "Rejected", "Needs Clarification"]'
```

---

### Multiple Bypass Keywords

Allow different keywords for bypassing the check:

```yaml
- uses: designcise/jira-ticket-status-action@v1
  with:
    jira-base-url: ${{ secrets.JIRA_BASE_URL }}
    jira-email: ${{ secrets.JIRA_EMAIL }}
    jira-api-token: ${{ secrets.JIRA_API_TOKEN }}
    ticket-prefix: 'DEV'
    bypass-keywords: '["noticket", "hotfix", "emergency", "skip-jira"]'
```

**PR titles that bypass:**
- `[noticket] Update dependencies`
- `[HOTFIX] Critical security patch`
- `emergency: Fix memory leak`
- `skip-jira Refactor tests`
- `[no-ticket] Quick fix` (handles variations)

---

### Multiple Projects

Check tickets from different projects:

```yaml
jobs:
  check-project-a:
    runs-on: ubuntu-latest
    steps:
      - uses: designcise/jira-ticket-status-action@v1
        with:
          jira-base-url: ${{ secrets.JIRA_BASE_URL }}
          jira-email: ${{ secrets.JIRA_EMAIL }}
          jira-api-token: ${{ secrets.JIRA_API_TOKEN }}
          ticket-prefix: 'PROJA'

  check-project-b:
    runs-on: ubuntu-latest
    steps:
      - uses: designcise/jira-ticket-status-action@v1
        with:
          jira-base-url: ${{ secrets.JIRA_BASE_URL }}
          jira-email: ${{ secrets.JIRA_EMAIL }}
          jira-api-token: ${{ secrets.JIRA_API_TOKEN }}
          ticket-prefix: 'PROJB'
```

---

### Using Outputs

Make decisions based on check results:

```yaml
- name: Check Jira tickets
  id: jira
  uses: designcise/jira-ticket-status-action@v1
  with:
    jira-base-url: ${{ secrets.JIRA_BASE_URL }}
    jira-email: ${{ secrets.JIRA_EMAIL }}
    jira-api-token: ${{ secrets.JIRA_API_TOKEN }}
    ticket-prefix: 'PROJ'

- name: Notify team if blocked
  if: steps.jira.outputs.has-blocked == 'true'
  run: |
    echo "Blocked tickets: ${{ steps.jira.outputs.blocked-tickets }}"
    # Send Slack notification, create comment, etc.
```

---

### Only Check for Specific Branches

Run the check only when merging to production branches:

```yaml
name: Jira Ticket Status Check
on:
  pull_request:
    types: [opened, edited, synchronize]
    branches:
      - main
      - production
      - staging

jobs:
  check-jira:
    runs-on: ubuntu-latest
    steps:
      - uses: designcise/jira-ticket-status-action@v1
        with:
          jira-base-url: ${{ secrets.JIRA_BASE_URL }}
          jira-email: ${{ secrets.JIRA_EMAIL }}
          jira-api-token: ${{ secrets.JIRA_API_TOKEN }}
          ticket-prefix: 'PROJ'
```

---

## 📋 Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `jira-base-url` | Jira instance URL (e.g., `https://company.atlassian.net`)<br>⚠️ No trailing slash | **Yes** | - |
| `jira-email` | Email address for Jira API authentication | **Yes** | - |
| `jira-api-token` | API token for authentication ([How to create](https://id.atlassian.com/manage-profile/security/api-tokens)) | **Yes** | - |
| `ticket-prefix` | Jira ticket prefix (e.g., `PROJ`, `JIRA`, `DEV`) | **Yes** | - |
| `blocked-statuses` | JSON array of Jira statuses that should block PRs | No | `["Requires fixing", "Blocked", "On Hold", "Waiting for Dependency"]` |
| `bypass-keywords` | JSON array of keywords in PR title to bypass check | No | `["noticket"]` |

---

## 📤 Outputs

| Output | Type | Description |
|--------|------|-------------|
| `blocked-tickets` | JSON Array | List of tickets with blocking statuses (e.g., `[{"key":"PROJ-123","status":"Blocked"}]`) |
| `all-tickets` | JSON Array | All unique tickets found in commits |
| `has-blocked` | Boolean | `true` if any ticket has a blocking status, `false` otherwise |

---

## 🎯 How It Works

### Workflow

```
1. Check for bypass keywords in PR title
   ↓
   If found → Skip all checks ✅
   If not found → Continue

2. Extract tickets from commits
   ↓
   Scan all commit messages for {PREFIX}-{number}
   Examples: 
   - "PROJ-100: Add feature" → Extract "PROJ-100"
   - "chore(PROJ-100): test" → Extract "PROJ-100"
   - "Fix PROJ-100 and PROJ-101" → Extract both
   Remove duplicates using Set

3. Check Jira status
   ↓
   For each unique ticket:
   - Query Jira API: /rest/api/3/issue/{key}?fields=status
   - Get current workflow status

4. Validate & Block
   ↓
   If any ticket status is in blocked-statuses list:
   - ❌ Fail the check
   - 🚫 Block PR merge
   - 📊 Show summary table
   
   Otherwise:
   - ✅ Pass the check
   - 🎉 Allow merge
```

---

## 💡 Commit Message Formats

The action finds tickets **anywhere** in the commit message. All these formats work:

### Standard Format
```
PROJ-123: Add user authentication
```

### Conventional Commits
```
feat(PROJ-123): add login page
chore(PROJ-123): update dependencies
fix(PROJ-123): resolve memory leak
```

### Brackets
```
[PROJ-123] Implement feature
```

### Multiple Tickets
```
Fix PROJ-123 and PROJ-456
```

### Ticket at End
```
Add feature for PROJ-123
```

### Mixed Case (Still Works)
```
proj-123: lowercase still works
PROJ-123: UPPERCASE WORKS TOO
```

---

## 💡 Example Scenarios

### Scenario 1: Single Feature Branch

**Configuration:**
```yaml
ticket-prefix: 'DEV'
```

**Commits:**
```
abc1234 DEV-100: Implement user authentication
def5678 chore(DEV-100): Add tests for auth
ghi9012 Fix linting issues in DEV-100
```

**Result:**
- Finds 1 unique ticket: `DEV-100`
- Checks status: `In Progress` ✅
- **Decision: PASS** - PR can be merged

---

### Scenario 2: Merge from develop to staging

**Configuration:**
```yaml
ticket-prefix: 'PROJ'
```

**Commits:**
```
123abcd PROJ-100: Feature A
456efgh feat(PROJ-101): Feature B
789ijkl PROJ-102: Feature C
012mnop Fix typo in PROJ-100
```

**Result:**
- Finds 3 unique tickets: `PROJ-100`, `PROJ-101`, `PROJ-102`
- Checks statuses:
  - `PROJ-100`: `Done` ✅
  - `PROJ-101`: `Blocked` ❌
  - `PROJ-102`: `In Review` ✅
- **Decision: FAIL** - Cannot merge due to PROJ-101

**Action Summary:**
```
Jira Ticket Status Check
┌──────────┬────────────┬──────────────┐
│ Ticket   │ Status     │ Result       │
├──────────┼────────────┼──────────────┤
│ PROJ-100 │ Done       │ ✅ OK        │
│ PROJ-101 │ Blocked    │ ❌ BLOCKED   │
│ PROJ-102 │ In Review  │ ✅ OK        │
└──────────┴────────────┴──────────────┘

❌ The following tickets have blocking statuses:
  - PROJ-101: Blocked
```

---

### Scenario 3: Emergency Hotfix

**PR Title:** `[noticket] Emergency production fix for memory leak`

**Result:**
- Detects bypass keyword: `noticket`
- **Decision: PASS** - Skips all checks
- GitHub Actions notice: `Bypassing Jira check due to keyword: "noticket"`

---

### Scenario 4: Multiple Bypass Keywords

**Configuration:**
```yaml
bypass-keywords: '["noticket", "hotfix", "emergency"]'
```

**PR Titles that bypass:**
- `[noticket] Update dependencies` → Matches `noticket`
- `[HOTFIX] Critical patch` → Matches `hotfix`
- `emergency: Fix production` → Matches `emergency`
- `[no-ticket] Quick fix` → Matches `noticket` (handles variations)

---

## 🔐 Security Best Practices

### Use a Service Account

Create a dedicated Jira user for CI/CD:

```
Email: github-ci@company.com
Name: GitHub CI Bot
Permissions: Browse Projects (read-only)
```

**Benefits:**
- ✅ Audit trail of API usage
- ✅ Minimal permissions (least privilege)
- ✅ Easy to rotate credentials
- ✅ No personal API tokens in CI

---

### Protect Your Secrets

1. **Never commit secrets** to your repository
2. **Use environment secrets** for organization-wide sharing
3. **Rotate API tokens** regularly (recommended: every 90 days)
4. **Review access logs** in Jira periodically

---

## 🛠️ Setup Guide

### Step 1: Create Jira API Token

1. Log into Jira with the account you want to use for CI
2. Navigate to: https://id.atlassian.com/manage-profile/security/api-tokens
3. Click **"Create API token"**
4. Label: `GitHub Actions CI`
5. Click **"Create"**
6. **⚠️ Copy the token immediately** (you won't see it again)

### Step 2: Add GitHub Secrets

1. Go to your repository on GitHub
2. Navigate to **Settings → Secrets and variables → Actions**
3. Click **"New repository secret"**
4. Add each secret:

**JIRA_BASE_URL**
```
https://your-company.atlassian.net
```
⚠️ Important: No trailing slash, no `/browse/` or other paths

**JIRA_EMAIL**
```
github-ci@your-company.com
```
Must match the email of the account that created the API token

**JIRA_API_TOKEN**
```
ATATT3xFfGF0abcdefghijklmnop1234567890
```
Paste the full token copied in Step 1

### Step 3: Find Your Ticket Prefix

Your ticket prefix is the part before the dash in your Jira tickets:

- `PROJ-123` → prefix is `PROJ`
- `DEV-456` → prefix is `DEV`
- `JIRA-789` → prefix is `JIRA`

You can find this in your Jira project settings or by looking at any ticket URL:
```
https://company.atlassian.net/browse/PROJ-123
                                         ^^^^
                                      This is your prefix
```

### Step 4: Configure Branch Protection

1. Go to **Settings → Branches**
2. Click **"Add branch protection rule"**
3. Branch name pattern: `main` (or your default branch)
4. Enable **"Require status checks to pass before merging"**
5. Search for your job name (e.g., `check-jira`)
6. Click **"Create"**

### Step 5: Test It

1. Create a test PR with a Jira ticket in commit message:
   ```bash
   git commit -m "PROJ-123: Test PR for Jira integration"
   git push origin my-feature-branch
   ```
2. Open a PR on GitHub
3. Check the **Actions** tab for workflow run
4. Verify the ticket status is checked

---

## 🐛 Troubleshooting

### Error: "Failed to fetch PROJ-XXX: HTTP 404"

**Cause:** The Jira user doesn't have permission to view the ticket

**Solution:**
1. Log into Jira with the account from `JIRA_EMAIL`
2. Try to access the ticket manually
3. If you can't see it, grant **Browse Projects** permission
4. Or use a different account with broader access

---

### Error: "No PROJ-{number} found in commits"

**Cause:** Commits don't contain ticket references with your specified prefix

**Solution:**
1. Verify commit messages include `{PREFIX}-{number}` (e.g., `PROJ-123`)
2. Check that `ticket-prefix` matches your Jira project (case-sensitive)
3. Or add bypass keyword to PR title (e.g., `[noticket]`)

**Valid formats:**
- ✅ `PROJ-123: Add feature`
- ✅ `chore(PROJ-123): test`
- ✅ `Fix PROJ-123`
- ❌ `PROJ 123` (no dash)
- ❌ `proj-123` (wrong case for prefix)

---

### Error: "bypass-keywords is not valid JSON array"

**Cause:** Invalid JSON format in `bypass-keywords` input

**Solution:**
```yaml
# ❌ Wrong
bypass-keywords: noticket, hotfix

# ✅ Correct
bypass-keywords: '["noticket", "hotfix"]'
```

---

### Warning: "Never Accessed" in Atlassian

**Cause:** API token hasn't been used successfully

**Solution:**
1. Verify `JIRA_EMAIL` exactly matches the account that created the token
2. Check for typos in the token (regenerate if unsure)
3. Ensure `JIRA_BASE_URL` is correct (no trailing slash)

---

### Action runs on every commit

**Expected behavior** when using `synchronize` trigger - this ensures new commits are checked.

**To reduce frequency:**
```yaml
on:
  pull_request:
    types: [opened, edited]  # Removed synchronize
```

⚠️ **Note:** New commits won't retrigger the check. Edit PR title to rerun.

---

## 🤔 FAQ

### Q: Can I use this with Jira Server (self-hosted)?

**A:** Yes! Just use your Jira Server URL as `jira-base-url`:
```yaml
jira-base-url: https://jira.company.com
```

Ensure the REST API v3 is enabled.

---

### Q: Does the ticket need to be at the start of the commit message?

**A:** No! The action finds tickets **anywhere** in the commit message:
- ✅ `PROJ-123: Add feature`
- ✅ `chore(PROJ-123): test`
- ✅ `Add feature for PROJ-123`
- ✅ `Fix PROJ-100 and PROJ-101`

---

### Q: Can I check multiple ticket prefixes in one PR?

**A:** Not in a single action call, but you can run the action multiple times:

```yaml
steps:
  - uses: designcise/jira-ticket-status-action@v1
    with:
      ticket-prefix: 'PROJ'
      # ... other inputs

  - uses: designcise/jira-ticket-status-action@v1
    with:
      ticket-prefix: 'DEV'
      # ... other inputs
```

---

### Q: What if a ticket doesn't exist?

**A:** The action logs a warning and continues checking other tickets. Only existing, accessible tickets are validated.

---

### Q: How do I allow multiple bypass keywords?

**A:** Use a JSON array:
```yaml
bypass-keywords: '["noticket", "hotfix", "emergency", "skip-jira"]'
```

All keywords are case-insensitive and handle variations like `no-ticket`, `noticket`, `no ticket`.

---

### Q: Does this work with Jira Data Center?

**A:** Yes, as long as the REST API v3 is available at `/rest/api/3/issue/{key}`

---

### Q: Can I whitelist statuses instead of blacklisting?

**A:** Not directly, but you can achieve this by listing all unwanted statuses in `blocked-statuses`:

```yaml
# Allow only: "In Progress", "In Review", "Done"
# Block everything else:
blocked-statuses: '["New", "Backlog", "Blocked", "On Hold", "Paused", "Rejected"]'
```

---

### Q: How do I find my ticket prefix?

**A:** Look at any Jira ticket. The prefix is the part before the dash:
- URL: `https://company.atlassian.net/browse/PROJ-123` → `PROJ`
- Ticket: `DEV-456` → `DEV`
- Ticket: `JIRA-789` → `JIRA`

---

### Q: Are bypass keywords case-sensitive?

**A:** No! All keywords are matched case-insensitively:
- `noticket` matches `NOTICKET`, `NoTicket`, `[NOTICKET]`, etc.
- `hotfix` matches `HOTFIX`, `HotFix`, `[hotfix]`, etc.

---

## 📊 Supported Jira Versions

| Jira Type | Supported | API Version |
|-----------|-----------|-------------|
| Jira Cloud | ✅ Yes | REST API v3 |
| Jira Server 8.0+ | ✅ Yes | REST API v3 |
| Jira Data Center | ✅ Yes | REST API v3 |
| Jira Server <8.0 | ⚠️ Maybe | REST API v2 (not tested) |

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

### Development Setup

```bash
git clone https://github.com/designcise/jira-ticket-status-action.git
cd jira-ticket-status-action

# Test locally using act
act pull_request -s JIRA_BASE_URL=... -s JIRA_EMAIL=... -s JIRA_API_TOKEN=...
```

### Reporting Issues

Found a bug? [Open an issue](https://github.com/designcise/jira-ticket-status-action/issues) with:
- Workflow YAML
- Error message
- Jira version (Cloud/Server/Data Center)
- Ticket prefix being used
- Example commit messages

---

## 📜 License

MIT License - see [LICENSE](LICENSE) file for details

---

## 🙏 Acknowledgments

Powered by:
- [actions/github-script](https://github.com/actions/github-script)
- [Jira REST API v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)

---

## 📞 Support

- 📖 [Documentation](https://github.com/designcise/jira-ticket-status-action#readme)
- 🐛 [Issue Tracker](https://github.com/designcise/jira-ticket-status-action/issues)
- 💬 [Discussions](https://github.com/designcise/jira-ticket-status-action/discussions)

---

**⭐ If this action helps your team, please give it a star!**
