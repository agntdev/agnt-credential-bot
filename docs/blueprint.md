# Student Credential Distributor — Bot specification

**Archetype:** workflow

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that allows school admins to securely distribute student login credentials via private messages. Admins upload a CSV containing student records, and the bot sends individualized password messages to each student's Telegram account while tracking delivery status and reporting results.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- school administrators

## Success criteria

- Admin receives a summary report with delivery counts and failure details after distribution

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Initialize admin session and validate admin identity
- **Upload CSV** (button, actor: admin, callback: upload:start) — Begin CSV upload and validation process
  - inputs: CSV file
  - outputs: Validation preview
- **Confirm Send** (button, actor: admin, callback: distribution:confirm) — Execute password distribution to all validated student records
  - inputs: Confirmed CSV data
  - outputs: Delivery status updates
- **Get Report** (button, actor: admin) — Request downloadable CSV of delivery results
  - inputs: Report format preference
  - outputs: CSV file

## Flows

### Admin Onboarding
_Trigger:_ /start

1. Validate admin identity
2. Display main menu

_Data touched:_ Admin user

### CSV Distribution
_Trigger:_ upload:start

1. Receive CSV file
2. Validate column headers
3. Preview first 5 records
4. Request confirmation

_Data touched:_ Student records

### Password Delivery
_Trigger:_ distribution:confirm

1. Send individual messages
2. Track delivery status
3. Generate failure list
4. Compile summary report

_Data touched:_ Student records, Delivery status

### Report Generation
_Trigger:_ report:generate

1. Format success/failure data
2. Generate downloadable CSV
3. Send report to admin

_Data touched:_ Student records

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Student Record** _(retention: persistent with 30-day default retention)_ — Securely stored student credential distribution data
  - fields: ID/FanNumber, Full name, Telegram identifier, Encrypted password, Delivery status, Timestamps

## Integrations

- **Telegram** (required) — Private messaging and admin notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- CSV upload validation
- Password distribution initiation
- Delivery status reporting
- Data retention management

## Notifications

- Delivery success/failure summary to admin
- Individual student message delivery status updates

## Permissions & privacy

- Only admin can initiate distribution
- Passwords stored encrypted at rest
- Student messages sent only to verified Telegram identifiers

## Edge cases

- Invalid CSV format handling
- Telegram rate limiting during mass distribution
- Failed message retries with error reporting

## Required tests

- End-to-end CSV upload → message delivery → report generation workflow

## Assumptions

- First /start user is automatically designated admin
- CSV contains exact column set with case-insensitive matching
- Default message template used unless overridden
