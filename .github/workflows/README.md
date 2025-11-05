# Kanban Automation Workflow

This directory contains the GitHub Actions workflow for automating Kanban task management and Slack notifications.

## Workflow: kanban-update.yml

The `kanban-update.yml` workflow automatically checks off completed Kanban tasks when a Pull Request is merged and sends a notification to your Slack channel.

### Features

- **Automatic Task Completion**: When a PR is merged, any tasks referenced in the PR body (format: `[TASK-XXX]`) are automatically marked as complete in the KANBAN.md file
- **Slack Notifications**: Sends formatted notifications to your Slack channel with:
  - ✅ Header indicating tasks were completed
  - PR number and title with clickable link
  - PR author information
  - List of completed tasks
  - Merge timestamp

### Setup Instructions

#### 1. Create a Slack Incoming Webhook

1. Go to your Slack workspace's [Incoming Webhooks page](https://api.slack.com/messaging/webhooks)
2. Click "Create New App" → "From scratch"
3. Name your app (e.g., "Kanban Automation") and select your workspace
4. Under "Features" → "Incoming Webhooks", activate webhooks
5. Click "Add New Webhook to Workspace"
6. Select the channel: **#trading-signals** (Channel ID: C09NXCGC8N5)
   - Target: `https://odintradingcicd.slack.com/archives/C09NXCGC8N5`
7. Copy the webhook URL (it will look like: `https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXX`)

#### 2. Add the Webhook to GitHub Secrets

1. Navigate to your repository settings: **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Name: `SLACK_WEBHOOK_URL`
4. Value: Paste the webhook URL from Step 1
5. Click **Add secret**

#### 3. Test the Workflow

1. Create a test PR with tasks in the body:
   ```markdown
   This PR completes the following tasks:
   - [TASK-001] Implement feature X
   - [TASK-002] Fix bug Y
   ```
2. Merge the PR
3. Check your Slack channel for the notification
4. Verify the KANBAN.md file was updated

### Workflow Trigger

The workflow triggers on:
- Event: `pull_request`
- Type: `closed`
- Condition: `github.event.pull_request.merged == true`

### Task Format

Tasks must be referenced in PR bodies using the format:
```
[TASK-XXX]
```

Where `XXX` is any number. Multiple tasks can be referenced in a single PR.

### Slack Message Format

The Slack notification uses Block Kit formatting with three sections:
1. **Header**: "✅ Kanban Tasks Completed"
2. **Main Section**: PR details, author, and task list
3. **Context**: Merge timestamp

### Troubleshooting

**Workflow runs but no Slack message:**
- Verify the `SLACK_WEBHOOK_URL` secret is set correctly
- Check the Actions logs for error messages
- Ensure the webhook URL is active in Slack

**Tasks not being checked off:**
- Verify task format matches `[TASK-XXX]` in PR body
- Check that KANBAN.md exists in repository root
- Review workflow logs for parsing errors

**Slack webhook expired:**
- Regenerate the webhook in Slack
- Update the `SLACK_WEBHOOK_URL` secret in GitHub

### Implementation Details

- Uses `actions/github-script@v7` for scripting flexibility
- Native Node.js HTTPS module for webhook calls
- No external dependencies required
- Conditional execution: only sends notification when tasks are found

### Security Notes

- Never commit webhook URLs to the repository
- Always use GitHub Secrets for sensitive credentials
- Webhook URLs should be treated as passwords
- Regularly rotate webhooks as a security best practice
