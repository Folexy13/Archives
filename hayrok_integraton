# Jira and Slack Integration Documentation

## Table of Contents
1. [Overview](#overview)
2. [Jira Integration](#jira-integration)
   - [OAuth Connection Flow](#jira-oauth-connection-flow)
   - [API Endpoints](#jira-api-endpoints)
   - [Payload Structures](#jira-payload-structures)
   - [Webhook Events](#jira-webhook-events)
3. [Slack Integration](#slack-integration)
   - [OAuth Connection Flow](#slack-oauth-connection-flow)
   - [API Endpoints](#slack-api-endpoints)
   - [Payload Structures](#slack-payload-structures)
   - [Webhook Events](#slack-webhook-events)
4. [Configuration Requirements](#configuration-requirements)
5. [Security Considerations](#security-considerations)

---

## Overview

This document provides comprehensive guidance for integrating with Jira and Slack services through the Hayrok integration service. Both integrations use OAuth 2.0 for authentication and support webhook-based event processing.

**Base URL**: `http://localhost:7000` (configurable via `PUBLIC_URL` environment variable)

**Required Headers**:
- `x-tenant-id`: Tenant identifier
- `x-user-id`: User identifier

---

## Jira Integration

### Jira OAuth Connection Flow

#### 1. Initialize OAuth Flow

**Endpoint**: `POST /integrations/jira/connect/init`

**Request Headers**:
```json
{
  "x-tenant-id": "tenant-123",
  "x-user-id": "user-456"
}
```

**Request Body**:
```json
{
  "scopes": [
    "read:jira-work",
    "write:jira-work",
    "read:jira-user",
    "manage:jira-webhook"
  ],
  "redirectUrl": "https://your-app.com/integrations/callback"
}
```

**Response**:
```json
{
  "authUrl": "https://auth.atlassian.com/authorize?audience=api.atlassian.com&client_id=...",
  "state": "encrypted-state-token",
  "message": "Redirect user to authUrl to complete Jira authorization"
}
```

#### 2. OAuth Callback

After user authorization, Jira redirects to: `GET /integrations/jira/callback?code={code}&state={state}`

**Response**:
```json
{
  "success": true,
  "integrationId": "uuid-v4",
  "cloudId": "cloud-id-123",
  "siteName": "My Jira Site",
  "siteUrl": "https://mysite.atlassian.net",
  "jiraAccountId": "account-id",
  "displayName": "John Doe",
  "webhookUrl": "http://localhost:7000/webhooks/jira/tenant-123",
  "webhookSecret": "secret-key",
  "availableResources": [
    {
      "id": "cloud-id-123",
      "name": "My Jira Site",
      "url": "https://mysite.atlassian.net",
      "scopes": ["read:jira-work", "write:jira-work"]
    }
  ]
}
```

#### 3. Check Connection Status

**Endpoint**: `GET /integrations/jira/status`

**Response**:
```json
{
  "connected": true,
  "integrationId": "uuid-v4",
  "cloudId": "cloud-id-123",
  "siteName": "My Jira Site",
  "siteUrl": "https://mysite.atlassian.net",
  "displayName": "John Doe",
  "email": "john.doe@example.com",
  "avatarUrl": "https://avatar-url.com/avatar.png",
  "scopes": ["read:jira-work", "write:jira-work"],
  "connectedAt": "2024-01-01T00:00:00.000Z",
  "webhookUrl": "http://localhost:7000/webhooks/jira/tenant-123",
  "tokenExpiresAt": "2024-01-02T00:00:00.000Z"
}
```

### Jira API Endpoints

#### Issue Management

##### Create Issue
**Endpoint**: `POST /integrations/jira/issues`

**Request Body**:
```json
{
  "projectKey": "PROJ",
  "summary": "Bug in login functionality",
  "description": "Users cannot login with valid credentials",
  "issueType": "Bug",
  "priority": "High",
  "assigneeAccountId": "account-id-123",
  "labels": ["bug", "login", "critical"],
  "components": ["Authentication"],
  "parentKey": "PROJ-100",
  "customFields": {
    "customfield_10001": "value"
  }
}
```

**Response**:
```json
{
  "success": true,
  "issue": {
    "id": "10001",
    "key": "PROJ-123",
    "self": "https://api.atlassian.com/ex/jira/cloud-id/rest/api/3/issue/10001",
    "url": "https://mysite.atlassian.net/browse/PROJ-123"
  }
}
```

##### Update Issue
**Endpoint**: `PUT /integrations/jira/issues/{issueKey}`

**Request Body**:
```json
{
  "summary": "Updated summary",
  "description": "Updated description",
  "priority": "Medium",
  "assigneeAccountId": "new-assignee-id",
  "statusId": "10001",
  "labels": ["updated", "in-progress"],
  "components": ["Backend"],
  "customFields": {
    "customfield_10002": "new-value"
  }
}
```

##### Get Issue
**Endpoint**: `GET /integrations/jira/issues/{issueKey}`

**Response**:
```json
{
  "id": "10001",
  "key": "PROJ-123",
  "self": "https://api.atlassian.com/...",
  "url": "https://mysite.atlassian.net/browse/PROJ-123",
  "fields": {
    "summary": "Issue summary",
    "description": {
      "type": "doc",
      "version": 1,
      "content": [...]
    },
    "status": {
      "id": "1",
      "name": "Open",
      "statusCategory": {
        "name": "To Do"
      }
    },
    "priority": {
      "id": "3",
      "name": "High"
    },
    "assignee": {
      "accountId": "account-id",
      "displayName": "John Doe",
      "emailAddress": "john@example.com"
    }
  },
  "transitions": [...],
  "changelog": {...}
}
```

##### Delete Issue
**Endpoint**: `DELETE /integrations/jira/issues/{issueKey}`

##### Transition Issue
**Endpoint**: `POST /integrations/jira/issues/{issueKey}/transitions`

**Request Body**:
```json
{
  "transitionId": "31",
  "comment": "Moving to In Progress",
  "resolution": "Fixed"
}
```

##### Add Comment
**Endpoint**: `POST /integrations/jira/issues/{issueKey}/comments`

**Request Body**:
```json
{
  "body": "This is a comment on the issue",
  "visibility": {
    "type": "role",
    "value": "Developers"
  }
}
```

##### Search Issues (JQL)
**Endpoint**: `POST /integrations/jira/search`

**Request Body**:
```json
{
  "jql": "project = PROJ AND status = Open",
  "startAt": 0,
  "maxResults": 50,
  "fields": ["summary", "status", "assignee"],
  "expand": ["changelog", "transitions"]
}
```

**Response**:
```json
{
  "startAt": 0,
  "maxResults": 50,
  "total": 123,
  "issues": [
    {
      "id": "10001",
      "key": "PROJ-123",
      "fields": {...}
    }
  ]
}
```

### Jira Payload Structures

#### Webhook Event Payload

Jira sends webhooks to: `POST /webhooks/jira/{tenantId}`

##### Issue Created/Updated/Deleted Event
```json
{
  "webhookEvent": "jira:issue_created",
  "timestamp": "2024-01-01T00:00:00.000Z",
  "issue": {
    "id": "10001",
    "key": "PROJ-123",
    "self": "https://api.atlassian.com/...",
    "fields": {
      "summary": "Issue summary",
      "description": "Issue description",
      "issuetype": {
        "id": "10001",
        "name": "Bug",
        "iconUrl": "https://..."
      },
      "status": {
        "id": "1",
        "name": "Open",
        "statusCategory": {
          "id": "2",
          "key": "new",
          "name": "To Do"
        }
      },
      "priority": {
        "id": "3",
        "name": "High",
        "iconUrl": "https://..."
      },
      "project": {
        "id": "10000",
        "key": "PROJ",
        "name": "My Project"
      },
      "assignee": {
        "accountId": "account-id",
        "displayName": "John Doe",
        "emailAddress": "john@example.com",
        "avatarUrls": {
          "48x48": "https://..."
        }
      },
      "reporter": {
        "accountId": "reporter-id",
        "displayName": "Jane Smith"
      },
      "labels": ["bug", "critical"],
      "components": [
        {
          "id": "10001",
          "name": "Backend"
        }
      ],
      "created": "2024-01-01T00:00:00.000Z",
      "updated": "2024-01-01T01:00:00.000Z"
    }
  },
  "user": {
    "accountId": "user-id",
    "displayName": "User Name",
    "emailAddress": "user@example.com"
  },
  "changelog": {
    "id": "10001",
    "items": [
      {
        "field": "status",
        "fieldtype": "jira",
        "fieldId": "status",
        "from": "1",
        "fromString": "Open",
        "to": "3",
        "toString": "In Progress"
      }
    ]
  }
}
```

##### Comment Event
```json
{
  "webhookEvent": "comment_created",
  "timestamp": "2024-01-01T00:00:00.000Z",
  "issue": {
    "id": "10001",
    "key": "PROJ-123",
    "fields": {
      "summary": "Issue summary"
    }
  },
  "comment": {
    "id": "10000",
    "self": "https://api.atlassian.com/...",
    "body": "Comment text",
    "author": {
      "accountId": "author-id",
      "displayName": "John Doe",
      "emailAddress": "john@example.com"
    },
    "created": "2024-01-01T00:00:00.000Z",
    "updated": "2024-01-01T00:00:00.000Z",
    "visibility": {
      "type": "role",
      "value": "Developers"
    }
  }
}
```

### Jira Webhook Events

Supported webhook events:
- `jira:issue_created` - Issue created
- `jira:issue_updated` - Issue updated
- `jira:issue_deleted` - Issue deleted
- `comment_created` - Comment added
- `comment_updated` - Comment updated
- `worklog_updated` - Work log updated
- `issue_property_set` - Issue property set
- `issue_property_deleted` - Issue property deleted

---

## Slack Integration

### Slack OAuth Connection Flow

#### 1. Initialize OAuth Flow

**Endpoint**: `POST /integrations/slack/connect/init`

**Request Headers**:
```json
{
  "x-tenant-id": "tenant-123",
  "x-user-id": "user-456"
}
```

**Response**:
```json
{
  "authUrl": "https://slack.com/oauth/v2/authorize?client_id=...&scope=...",
  "state": "encrypted_state_string"
}
```

**Default Scopes**:
- `channels:read` - View basic channel info
- `channels:write.invites` - Invite members to channels
- `chat:write` - Post messages
- `users:read` - View users in workspace
- `channels:history` - View messages in public channels
- `groups:history` - View messages in private channels
- `im:read` - View direct messages list
- `im:history` - View direct message history
- `mpim:read` - View group DMs list
- `mpim:history` - View group DM history

#### 2. OAuth Callback

After user authorization, Slack redirects to: `GET /integrations/slack/callback?code={code}&state={state}`

**Response** (redirect to frontend):
```
http://localhost:3000/integrations/slack?success=true&team=My%20Workspace
```

#### 3. Check Connection Status

**Endpoint**: `GET /integrations/slack/status`

**Response**:
```json
{
  "connected": true,
  "integrationId": "uuid-v4",
  "teamId": "T01234567",
  "teamName": "My Workspace",
  "botUserId": "U01234567",
  "slackUserId": "U98765432",
  "scopes": ["channels:read", "chat:write", "users:read"],
  "connectedAt": "2024-01-01T00:00:00.000Z",
  "webhookUrl": "http://localhost:7000/webhooks/slack/tenant-123",
  "incomingWebhook": {
    "url": "https://hooks.slack.com/services/...",
    "channel": "#general",
    "channelId": "C01234567"
  }
}
```

### Slack API Endpoints

#### Messaging

##### Send Message
**Endpoint**: `POST /integrations/slack/messages/send`

**Request Body**:
```json
{
  "channel": "C01234567",
  "text": "Hello, World!",
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Bold text* and _italic text_"
      }
    }
  ],
  "attachments": [
    {
      "color": "good",
      "title": "Attachment Title",
      "text": "Attachment text"
    }
  ],
  "thread_ts": "1234567890.123456",
  "reply_broadcast": false,
  "unfurl_links": true,
  "unfurl_media": true
}
```

**Response**:
```json
{
  "success": true,
  "message": {
    "type": "message",
    "text": "Hello, World!",
    "user": "U01234567",
    "ts": "1234567890.123456"
  },
  "channel": "C01234567",
  "ts": "1234567890.123456"
}
```

##### Update Message
**Endpoint**: `PUT /integrations/slack/messages/update`

**Request Body**:
```json
{
  "channel": "C01234567",
  "ts": "1234567890.123456",
  "text": "Updated message",
  "blocks": [...],
  "attachments": [...]
}
```

##### Delete Message
**Endpoint**: `DELETE /integrations/slack/messages/delete`

**Request Body**:
```json
{
  "channel": "C01234567",
  "ts": "1234567890.123456"
}
```

##### Add Reaction
**Endpoint**: `POST /integrations/slack/reactions/add`

**Request Body**:
```json
{
  "channel": "C01234567",
  "timestamp": "1234567890.123456",
  "name": "thumbsup"
}
```

#### Channel Management

##### Get Channels
**Endpoint**: `GET /integrations/slack/channels`

**Query Parameters**:
- `exclude_archived`: boolean (default: true)

**Response**:
```json
{
  "success": true,
  "channels": [
    {
      "id": "C01234567",
      "name": "general",
      "isChannel": true,
      "isPrivate": false,
      "isArchived": false,
      "isMember": true,
      "numMembers": 50,
      "topic": "General discussion",
      "purpose": "Company-wide announcements",
      "created": 1234567890
    }
  ]
}
```

##### Get Channel History
**Endpoint**: `GET /integrations/slack/channels/{channelId}/history`

**Query Parameters**:
- `limit`: number (default: 100)
- `oldest`: timestamp
- `latest`: timestamp

**Response**:
```json
{
  "success": true,
  "messages": [
    {
      "type": "message",
      "text": "Message text",
      "user": "U01234567",
      "ts": "1234567890.123456",
      "thread_ts": "1234567890.123456"
    }
  ],
  "hasMore": false,
  "pinCount": 2
}
```

#### User Management

##### Get Users
**Endpoint**: `GET /integrations/slack/users`

**Response**:
```json
{
  "success": true,
  "users": [
    {
      "id": "U01234567",
      "name": "john.doe",
      "realName": "John Doe",
      "displayName": "John",
      "email": "john@example.com",
      "isBot": false,
      "isAdmin": false,
      "isOwner": false,
      "deleted": false,
      "avatar": "https://avatars.slack-edge.com/...",
      "timezone": "America/New_York"
    }
  ]
}
```

#### File Upload

##### Upload File
**Endpoint**: `POST /integrations/slack/files/upload`

**Request Body**:
```json
{
  "channels": ["C01234567"],
  "content": "File content as string",
  "filename": "document.txt",
  "filetype": "text",
  "initial_comment": "Here's the document",
  "thread_ts": "1234567890.123456",
  "title": "Important Document"
}
```

**Response**:
```json
{
  "success": true,
  "file": {
    "id": "F01234567",
    "name": "document.txt",
    "title": "Important Document",
    "mimetype": "text/plain",
    "size": 1234,
    "url": "https://files.slack.com/...",
    "permalink": "https://myworkspace.slack.com/files/..."
  }
}
```

### Slack Payload Structures

#### Webhook Event Payload

Slack sends webhooks to: `POST /webhooks/slack/{tenantId}`

##### URL Verification
```json
{
  "type": "url_verification",
  "challenge": "3eZbrw1aBm2rZgRNFdxV2595E9CY3gmdALWMmHkvFXO7tYXAYM8P"
}
```

##### Message Event
```json
{
  "type": "event_callback",
  "team_id": "T01234567",
  "event_id": "Ev01234567",
  "event_time": 1234567890,
  "event": {
    "type": "message",
    "subtype": "message_changed",
    "text": "Hello, World!",
    "user": "U01234567",
    "channel": "C01234567",
    "channel_type": "channel",
    "ts": "1234567890.123456",
    "thread_ts": "1234567890.123456",
    "blocks": [...],
    "attachments": [...],
    "files": [...],
    "edited": {
      "user": "U01234567",
      "ts": "1234567890.123456"
    },
    "bot_id": "B01234567"
  }
}
```

##### App Mention Event
```json
{
  "type": "event_callback",
  "team_id": "T01234567",
  "event": {
    "type": "app_mention",
    "text": "<@U01234567> hello bot!",
    "user": "U98765432",
    "channel": "C01234567",
    "ts": "1234567890.123456",
    "thread_ts": "1234567890.123456",
    "blocks": [...]
  }
}
```

##### Reaction Event
```json
{
  "type": "event_callback",
  "team_id": "T01234567",
  "event": {
    "type": "reaction_added",
    "reaction": "thumbsup",
    "user": "U01234567",
    "item": {
      "type": "message",
      "channel": "C01234567",
      "ts": "1234567890.123456"
    },
    "item_user": "U98765432",
    "event_ts": "1234567890.123456"
  }
}
```

##### Channel Event
```json
{
  "type": "event_callback",
  "team_id": "T01234567",
  "event": {
    "type": "channel_created",
    "channel": {
      "id": "C01234567",
      "name": "new-channel",
      "created": 1234567890,
      "creator": "U01234567",
      "is_channel": true,
      "is_private": false
    }
  }
}
```

##### Member Event
```json
{
  "type": "event_callback",
  "team_id": "T01234567",
  "event": {
    "type": "member_joined_channel",
    "user": "U01234567",
    "channel": "C01234567",
    "channel_type": "C",
    "team": "T01234567",
    "inviter": "U98765432"
  }
}
```

### Slack Webhook Events

Supported webhook events:
- `message` - Message posted
- `message.channels` - Message in public channel
- `message.groups` - Message in private channel
- `message.im` - Direct message
- `message.mpim` - Group direct message
- `app_mention` - Bot mentioned
- `channel_created` - Channel created
- `channel_deleted` - Channel deleted
- `channel_rename` - Channel renamed
- `member_joined_channel` - Member joined channel
- `member_left_channel` - Member left channel
- `reaction_added` - Reaction added to message
- `reaction_removed` - Reaction removed from message
- `team_join` - New member joined workspace
- `user_change` - User profile changed

---

## Configuration Requirements

### Environment Variables

```bash
# Jira Configuration
JIRA_OAUTH_CLIENT_ID=your-jira-client-id
JIRA_OAUTH_CLIENT_SECRET=your-jira-client-secret
JIRA_OAUTH_CALLBACK_URL=http://localhost:7000/integrations/jira/callback

# Slack Configuration
SLACK_OAUTH_CLIENT_ID=your-slack-client-id
SLACK_OAUTH_CLIENT_SECRET=your-slack-client-secret
SLACK_OAUTH_CALLBACK_URL=http://localhost:7000/integrations/slack/callback
SLACK_SIGNING_SECRET=your-slack-signing-secret

# General Configuration
PUBLIC_URL=http://localhost:7000
FRONTEND_URL=http://localhost:3000
ENCRYPTION_KEY=your-32-character-encryption-key

# MongoDB
MONGODB_URI=mongodb://localhost:27017/integration-service

# Kafka
KAFKA_BROKERS=localhost:9092
KAFKA_CLIENT_ID=integration-service
KAFKA_GROUP_ID=integration-service-group

# AWS S3 (for webhook payload storage)
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
S3_BUCKET_NAME=webhook-payloads
```

### Jira App Configuration

1. Create an OAuth 2.0 app at https://developer.atlassian.com/console/myapps/
2. Configure OAuth 2.0 settings:
   - Callback URL: `http://localhost:7000/integrations/jira/callback`
   - Permissions: Jira API (read/write)
3. Note the Client ID and Client Secret

### Slack App Configuration

1. Create a Slack app at https://api.slack.com/apps
2. Configure OAuth & Permissions:
   - Redirect URLs: `http://localhost:7000/integrations/slack/callback`
   - Bot Token Scopes: Add required scopes
   - User Token Scopes: `identity.basic`, `identity.email`, `identity.avatar`
3. Configure Event Subscriptions:
   - Request URL: `http://localhost:7000/webhooks/slack/{tenantId}`
   - Subscribe to bot events
4. Note the Client ID, Client Secret, and Signing Secret

---

## Security Considerations

### Token Storage
- All tokens are encrypted using AES-256-CBC before storage
- Encryption keys are tenant-specific
- Tokens are stored in MongoDB with encryption

### Webhook Verification
- **Jira**: Uses webhook URL secrecy and HTTPS
- **Slack**: Uses HMAC-SHA256 signature verification with signing secret

### OAuth State Protection
- State parameter encrypted with AES-256-CBC
- Contains userId, tenantId, and timestamp
- State expires after 10 minutes

### Access Control
- All endpoints require `x-tenant-id` and `x-user-id` headers
- Integrations are tenant and user-specific
- Tokens are isolated per tenant

### Token Refresh
- Jira: Automatic token refresh using refresh token
- Slack: Bot tokens don't expire (user tokens may need refresh)

### Audit Logging
- All actions are logged with:
  - Tenant ID
  - User ID
  - Action type
  - Resource details
  - Timestamp
  - Metadata

### Data Storage
- Webhook payloads stored in S3 for audit trail
- Payload hash calculated for integrity verification
- Sensitive data encrypted at rest

---

## Error Handling

### Common Error Responses

#### 400 Bad Request
```json
{
  "statusCode": 400,
  "message": "Missing required parameters",
  "error": "Bad Request"
}
```

#### 401 Unauthorized
```json
{
  "statusCode": 401,
  "message": "Integration not found or inactive",
  "error": "Unauthorized"
}
```

#### 404 Not Found
```json
{
  "statusCode": 404,
  "message": "Resource not found",
  "error": "Not Found"
}
```

#### 500 Internal Server Error
```json
{
  "statusCode": 500,
  "message": "Internal server error",
  "error": "Internal Server Error"
}
```

### Integration-Specific Errors

#### Jira Errors
- Token expired: Automatic refresh attempted
- Invalid project/issue: Returns specific error message
- Permission denied: Returns 403 with details

#### Slack Errors
- Invalid channel: Returns channel not found
- Rate limited: Returns retry-after header
- Invalid signature: Webhook rejected with 401

---

## Best Practices

### Rate Limiting
- Implement exponential backoff for API calls
- Respect rate limit headers from APIs
- Cache frequently accessed data

### Webhook Processing
- Process webhooks asynchronously via Kafka
- Store raw payloads for debugging
- Implement idempotency checks

### Error Recovery
- Implement retry logic with backoff
- Log all errors for monitoring
- Provide clear error messages to users

### Monitoring
- Track integration health metrics
- Monitor webhook processing times
- Alert on authentication failures

### Data Synchronization
- Use webhooks for real-time updates
- Implement periodic sync for consistency
- Handle partial failures gracefully
