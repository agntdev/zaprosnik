# Request Manager Bot — Bot specification

**Archetype:** custom

**Voice:** concise and polite — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that allows users to create and track requests, with notifications sent to a designated owner chat. The owner can manage request statuses and communicate with users directly through the bot.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- end users on Telegram
- owner and their team

## Success criteria

- Users can create and track requests with clear status updates
- Owner receives notifications and can update request statuses
- All interactions are handled within Telegram

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **Create request** (button, actor: user, callback: create_request:start) — Initiate the request creation flow
  - inputs: title, description
  - outputs: request_id, status
- **My requests** (button, actor: user, callback: my_requests:list) — View a list of the user's active requests
  - inputs: user_id
  - outputs: request list with statuses
- **Help** (button, actor: user, callback: help:show) — Display usage instructions and contact information
  - outputs: help text

## Flows

### Create Request Flow
_Trigger:_ create_request:start

1. Ask for title
2. Ask for description (optional)
3. Confirm submission
4. Provide request ID and status

_Data touched:_ Request

### My Requests Flow
_Trigger:_ my_requests:list

1. List active requests
2. Show details for selected request
3. Offer cancel option if allowed

_Data touched:_ Request, User

### Owner Notification Flow
_Trigger:_ new_request:notify

1. Send notification to owner chat
2. Include buttons for status changes and messaging user

_Data touched:_ Request, Notification

### Status Update Flow
_Trigger:_ change_status:action

1. Owner selects new status
2. Notify user of status change
3. Update request status in database

_Data touched:_ Request, Notification

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user account information
  - fields: telegram_id, display_name
- **Request** _(retention: persistent)_ — A generic request/task/order submitted by a user
  - fields: id, user_id, title, description, status, created_at, updated_at
- **Notification** _(retention: persistent)_ — Messages sent to owner/team about request changes
  - fields: request_id, message, timestamp

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Designate owner chat for notifications
- View and update request statuses
- Send direct messages to users through the bot

## Notifications

- New request notifications to owner chat
- Status change notifications to users
- Help instructions and contact info

## Permissions & privacy

- Access to Telegram user IDs and display names
- Storage of request data with user consent implied by usage
- Secure handling of owner chat notifications

## Edge cases

- User tries to cancel a request that's already in progress
- Owner tries to update a request without proper permissions
- Multiple users submit requests simultaneously

## Required tests

- End-to-end test of request creation and status update flow
- Test notification delivery to owner chat
- Test user request cancellation and status visibility

## Assumptions

- Owner chat is a single designated Telegram chat
- Russian is the primary language with English fallback
- Status set includes New, In Progress, Needs Info, Completed, Rejected
