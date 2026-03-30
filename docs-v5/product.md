# Product

This document defines Skarbot from the user’s point of view: what it is, who it serves, which surfaces it exposes, and how membership works. Persistent file layout lives in [Architecture](architecture.md). Runtime behavior and tool contracts live in [Runtime](runtime.md).

## Identity

Skarbot is a personal claw-like household assistant, not a platform product. One deployment serves one trusted household. The deployment has one admin who owns the VM, the model auth, and the operating policy.

Each approved person gets their own long-lived direct thread. Family members share the deployment, but they do not share one conversation.

## Supported surfaces

Skarbot exposes four surfaces:

| Surface | Role |
| --- | --- |
| Web app | Main interactive interface |
| SMS | Lightweight mobile interaction and notifications |
| Admin email | Operational inbox for the admin and the admin approval surface |
| Scheduled jobs | Time-based automation bound to task threads |

All four surfaces resolve to the same underlying thread and task model. The UI layer is thin by design.

## Web app

The web app is the main interface. It exists to expose the same direct threads, task threads, attachments, and approvals that already exist in the runtime.

The web app includes:

- a home page that explains what Skarbot is
- a direct-thread and task-thread viewer
- file upload and attachment display
- a simple onboarding flow for authenticated people who are not yet members
- a waiting screen for users whose membership is still pending or denied
- basic logs and status visibility

The web app does not introduce a second chat system, task system, or approval system. Task files remain file-backed. The app is primarily a viewer and conversation surface.

### Viewer behavior

The thread viewer makes two distinctions explicit:

- direct threads versus task threads
- ordinary message history versus compaction boundaries

Compaction boundaries are first-class thread events in the viewer. They do not also appear as separate approvals, notifications, or status cards.

Chat and task-thread timestamps are rendered in the viewer’s local time.

## Membership and identity

Web identity comes from the authenticated email supplied by `exe.dev`. Skarbot uses that email as the primary web membership lookup key.

Membership is app-managed:

- authenticated users in `active/` go straight into the app
- authenticated users in `pending/` or `denied/` see the waiting screen
- unknown authenticated users enter onboarding

Onboarding collects:

- name
- email
- phone number
- country code
- timezone

If Skarbot can infer a timezone during onboarding, it proposes that value, but the user must still confirm or correct it before submission.

Email is normalized and used as the web identity key. Phone input is normalized to E.164 and used as the SMS identity key.

If onboarding submits a phone number that collides with a different existing user record, onboarding fails immediately and does not create a new record.

There is no canonical self-service profile editor. Approved users do not self-edit `name`, `email`, `phone`, or `timezone` through product surfaces. Those changes happen through admin action or direct file edit.

The initial admin user is created during deployment setup, not through the public onboarding flow.

## Channel behavior

Web, SMS, and admin email all land in the user’s direct thread unless the interaction is explicitly routed into a task thread.

Channel rules:

- replies default to the same surface that last received the user
- attachment-capable channels save files locally at ingress and convert them into standard substrate-compatible user content
- unknown SMS or email senders are rejected before any model call
- unauthorized inbound SMS and email attempts are logged to a deterministic runtime log
- unknown authenticated web users are routed into onboarding instead of the main app
- outbound member email is not a general product surface
- outbound email is reserved for admin notifications and approvals

SMS is a communication surface and a notification surface. It is not an approval surface.

## Threads in the product

A user sees two kinds of threads:

- **Direct thread** — the long-lived personal conversation
- **Task thread** — scoped work that should not clutter the direct thread

Scheduled work runs in task threads. Capability-building work runs in task threads. Direct-thread routing remains separate from logs and operational status.

## What Skarbot is optimized for

Skarbot is optimized for one self-hosted deployment, one admin, a small set of approved users, file-backed state, and thin product surfaces over a strong shared runtime.

It is not trying to be a general collaboration suite, a broad SaaS agent platform, or a product with many overlapping interfaces.

## See also

- [Architecture](architecture.md)
- [Runtime](runtime.md)
- [Deployment](deployment.md)
