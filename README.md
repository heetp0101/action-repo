# action-repo

This is the GitHub repository that sends webhook events to the webhook receiver. It's a simple repository used to demonstrate GitHub Actions and Webhooks integration.

## Overview

This repository (`action-repo`) serves as the **source repository** for GitHub webhook events. Whenever specific events happen in this repo (push, pull request, etc.), GitHub automatically sends webhooks to the registered endpoint.

## How It Works

1. **Webhook Configuration** - This repo has a GitHub webhook configured to send events to the webhook receiver
2. **Events Triggered** - When push, PR, or merge happens, GitHub sends webhook data
3. **Data Captured** - The webhook receiver processes and stores this data in MongoDB
4. **Display** - The UI polls MongoDB and displays these events

## Setup Instructions

### 1. Initial Setup

```
# Clone this repository
git clone <action-repo-url>
cd action-repo

# Create a sample file
echo "# GitHub Actions Demo" > README.md

# Make initial commit
git add .
git commit -m "Initial commit"
git push origin main
```

### 2. Configure GitHub Webhook

This is the **most important step**. You need to configure a webhook in this repository settings:

#### Steps:

1. Go to your repository on GitHub
2. Navigate to **Settings** (top-right menu)
3. Click **Webhooks** in the left sidebar
4. Click **Add webhook** button
5. Fill in the following:

   - **Payload URL:** `http://your-webhook-server/webhook`
     - For local testing: Use ngrok to expose local server
     - For production: Use your deployed webhook-repo URL
   
   - **Content type:** Select `application/json`
   
   - **Which events would you like to trigger this webhook?**
     - Select "Let me select individual events"
     - ✅ Check **Pushes** (for PUSH events)
     - ✅ Check **Pull requests** (for PULL_REQUEST and MERGE events)
     - Note: MERGE events are detected when a PR is closed with merged=true
     - "Merge groups" option is NOT needed
   
   - **Active:** Make sure ✅ is checked
   
   - Click **Add webhook**

#### Example Payload URL Formats:

**Local Development (with ngrok):**
```
http://abc123.ngrok.io/webhook
```

**Heroku Deployment:**
```
https://my-webhook-app.herokuapp.com/webhook
```

**AWS/GCP Deployment:**
```
https://webhook.mycompany.com/webhook
```

### 3. Testing the Webhook

#### Test Delivery (from GitHub UI):

1. Go to repo **Settings** → **Webhooks**
2. Click on the webhook you created
3. Scroll down to "Recent Deliveries"
4. Click on any delivery to see request/response
5. Click **Redeliver** to test again

#### Verify Data Was Stored:

```
# Check MongoDB directly
mongo
> use github_actions
> db.actions.find()

# Or use the API
curl http://localhost:5000/api/actions
```

## Making Changes to Trigger Events

### Trigger a PUSH Event

```
# Make a change to any file
echo "Some change" >> README.md

# Commit and push
git add .
git commit -m "Test push event"
git push origin main
```

**Expected Result:**
- GitHub sends webhook
- Webhook receiver stores action
- UI shows: "{author} pushed to "main" on {timestamp}"

### Trigger a PULL_REQUEST Event

```
# Create a new branch
git checkout -b feature-branch

# Make a change
echo "Feature code" > feature.txt

# Push the branch
git add .
git commit -m "Add feature"
git push origin feature-branch
```

**Then on GitHub:**
1. Go to your repository
2. Click **Pull requests** tab
3. Click **New pull request**
4. Select base: `main` and compare: `feature-branch`
5. Create pull request

**Expected Result:**
- GitHub sends webhook for pull_request event
- Webhook receiver stores action
- UI shows: "{author} submitted a pull request from "feature-branch" to "main" on {timestamp}"

### Trigger a MERGE Event

1. After creating a pull request (from above)
2. Click **Merge pull request** button
3. Confirm merge

**Expected Result:**
- GitHub sends webhook for merge commit
- Webhook receiver detects merge action
- UI shows: "{author} merged branch "feature-branch" to "main" on {timestamp}"

## File Structure

```
action-repo/
├── README.md          # This file with instructions
├── .github/
│   └── workflows/     # (Optional) GitHub Actions CI/CD
└── (your project files)
```

## Webhook Integration Details

### What Gets Sent for Each Event Type

#### PUSH Event
```
{
  "pusher": {
    "name": "john-doe"
  },
  "after": "abc123def456...",  // commit hash
  "ref": "refs/heads/main"      // branch reference
}
```

#### PULL_REQUEST Event
```
{
  "action": "opened",
  "pull_request": {
    "number": 5,
    "user": {
      "login": "john-doe"
    },
    "head": {
      "ref": "feature-branch"
    },
    "base": {
      "ref": "main"
    }
  }
}
```

#### PUSH (Merge Commit)
Detected as a special PUSH event when merging PRs.

### Testing Locally with ngrok

```bash
# Install ngrok: https://ngrok.com/download

# Start ngrok tunnel to port 5000
ngrok http 5000

# Copy the https URL (e.g., https://abc123.ngrok.io)

# Add to webhook URL in GitHub settings:
# https://abc123.ngrok.io/webhook

# Now local webhooks will work!
```
