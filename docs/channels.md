# Channels

Status: Research

## V1 Surfaces

- Inbound email from the admin user
- SMS via ClickSend
- Scheduled jobs / cron jobs
- Web UI is documented separately in [web-ui.md](web-ui.md)

## Channel Model

- First install must support inbound communication, not just outbound notifications
- Email, SMS, scheduled jobs, and web UI should all converge on one shared core
- The preferred structural reference is the lightweight adapter and registry model seen in `nanoclaw`
- Scheduled work should stay with explicit tasks and cron or scheduled jobs rather than a separate heartbeat concept
- Canonical direct-thread channel names in v1 should be `web`, `sms`, and `admin-email`
- Web, `sms`, and `admin-email` should support attachments in v1

## Routing

- Admin email, SMS, and web chat should map into the user's single direct thread in v1 unless explicitly routed to a task-linked thread
- Tasks are represented as task-linked threads rather than as a separate top-level product surface
- Scheduled jobs and recurring workflows are treated as first-class automation surfaces on the same core
- Replies to direct interactions should default to the same surface that initiated the interaction
- Scheduled tasks should use an explicit notification list rather than inferring one from the originating surface
- In v1, scheduled-task notification lists may be empty `[]` or may contain `direct-thread` and/or `sms`
- In v1, scheduled-task notifications should always target the task owner rather than arbitrary recipients
- The current direct-thread reply target should be tracked in a tiny sidecar metadata file rather than repeated in every stored message
- Inbound `sms` media should be downloaded promptly into local thread attachment storage rather than left as provider URLs
- Inbound `admin-email` attachments should be extracted from the Maildir message into local thread attachment storage
- Attachment-capable channels should convert local files into substrate-compatible user-message content at ingress rather than relying on ad-hoc transcript attachment fields
- Unknown SMS or email senders should receive no response in v1
- Unknown SMS or email senders should be rejected before any model call or token spend
- Unauthorized inbound SMS or email attempts should still be logged somewhere deterministic, with the exact log location deferred
- Unknown authenticated web users should be routed into the onboarding flow instead of getting direct app access
- Outbound email in v1 should stay limited to admin notifications rather than general member replies
