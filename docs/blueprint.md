# StreakMate — Bot specification

**Archetype:** custom

**Voice:** encouraging and concise — write every user-facing message, button label, error, and empty state in this voice.

StreakMate is a personal Telegram bot that helps users build and maintain habits through simple reminders and one-button completion tracking. It maintains private habit history, calculates current and best streaks, shows weekly reports, and respects time zones. The bot provides encouraging feedback while minimizing friction with one-click interactions.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- habit builders
- health and wellness enthusiasts
- students and lifelong learners
- people seeking behavior change

## Success criteria

- Users successfully track at least 3 habits for 7 consecutive days
- 80% of users complete weekly habit reports
- Users report reduced friction in habit tracking

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu and begin onboarding
- **+ New habit** (button, actor: user, callback: habit:new) — Create a new habit with schedule and reminder settings
  - inputs: habit title, schedule type, reminder time
  - outputs: confirmation of new habit
- **/new** (command, actor: user, command: /new) — Alternative entry for creating a new habit
- **Mark done** (button, actor: user, callback: habit:mark_done) — Record a habit as completed for the current day
  - inputs: habit ID
  - outputs: confirmation of completion and updated streaks
- **Snooze 30m** (button, actor: user, callback: habit:snooze) — Postpone the current reminder by 30 minutes
  - inputs: habit ID
  - outputs: updated reminder time
- **Mark skipped today** (button, actor: user, callback: habit:mark_skipped) — Record a habit as skipped for the current day
  - inputs: habit ID
  - outputs: confirmation of skipped day
- **Pause habit** (button, actor: user, callback: habit:pause) — Pause a habit to stop reminders temporarily
  - inputs: habit ID
  - outputs: confirmation of paused status
- **View history** (button, actor: user, callback: habit:view_history) — Open a compact week view of a habit's status
  - inputs: habit ID
  - outputs: weekly status summary

## Flows

### onboarding
_Trigger:_ /start

1. Detect or request timezone
2. Explain core actions with example
3. Show main menu

_Data touched:_ User

### create_habit
_Trigger:_ habit:new or /new

1. Name the habit
2. Choose schedule type (daily/weekdays/times_per_week)
3. Configure schedule details
4. Set reminder time
5. Add optional description
6. Confirm creation

_Data touched:_ Habit

### reminder_flow
_Trigger:_ scheduled reminder

1. Send reminder message with buttons
2. Handle button presses (mark done/snooze/mark skipped/pause)

_Data touched:_ DayRecord, Habit

### manual_edit
_Trigger:_ habit:view_history

1. Show week view with tappable days
2. Allow status changes for specific days
3. Update stats immediately

_Data touched:_ DayRecord, SeriesStats

### weekly_summary
_Trigger:_ weekly digest time

1. Generate summary for each habit
2. Show milestones and progress
3. Send digest message

_Data touched:_ SeriesStats, DayRecord

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user account information
  - fields: Telegram ID, display name, locale, timezone
- **Habit** _(retention: persistent)_ — User-defined habit with schedule and tracking parameters
  - fields: id, user_id, title, description, schedule_type, schedule_details, reminder_time_local, paused_flag, creation_date
- **DayRecord** _(retention: persistent)_ — Daily status record for a habit
  - fields: habit_id, dateLocal, status, timestamp, source, unique_marking_id
- **SeriesStats** _(retention: persistent)_ — Calculated statistics for habit streaks and completion
  - fields: current_streak, best_streak, percent_complete_over_window
- **NotificationRecord** _(retention: persistent)_ — Tracking of reminder attempts and user actions
  - fields: habit_id, attempt_timestamp, delivered_flag, action_token

## Integrations

- **Telegram** (required) — Bot API messaging and user interaction
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure free plan limits (default 5 habits)
- Enable/disable weekly summaries
- Set milestone celebration thresholds
- Configure premium features (unlimited habits, advanced analytics)

## Notifications

- Daily habit reminders with action buttons
- Weekly summary digest
- Milestone celebration notifications
- Status change confirmations

## Permissions & privacy

- All user data is private and only accessible by the user's Telegram ID
- No third-party sharing by default
- Data retention policies for habit history
- User controls for milestone notifications

## Edge cases

- Time zone changes during habit tracking
- Multiple attempts to mark the same day
- Reminder time conflicts with user's local midnight
- Paused habits resuming after long periods
- Manual edits to past days affecting streaks

## Required tests

- End-to-end habit creation and tracking flow
- Time zone handling across multiple regions
- Duplicate day marking prevention
- Weekly summary generation with milestone detection
- Premium feature toggling and limits enforcement

## Assumptions

- Telegram-provided timezone will be used when available
- Default free limit of 5 habits is acceptable
- Users will prefer one-click interactions over typing
- Milestone celebrations will be non-intrusive and optional
- Manual edits will be used for corrections, not gaming the system
