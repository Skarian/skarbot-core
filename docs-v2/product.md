# Product

This document defines Skarbot from the user’s point of view: who it is for, which surfaces it exposes, and how membership works. Persistent file layout lives in [Architecture](architecture.md). Tooling, approvals, and autonomy rules live in [Runtime](runtime.md).

## Identity

Skarbot is a personal claw-style assistant, not a platform product. One deployment serves one trusted household. The deployment has a single admin who owns the VM, the model auth, and the operating policy.

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

The web app is the main interface. It exists to expose the same conversations and task threads that already exist in the runtime.

The web app includes:

- a home page that explains what Skarbot is
- a direct-thread and task-thread viewer
- file upload and attachment display
- a simple onboarding flow for authenticated people who are not yet members
- a waiting screen for pending or denied users
- basic logs and status visibility

The web app does not introduce a second product model. It does not own a separate chat system, task system, or approval system. Task files remain file-backed; the web app is primarily a viewer and conversation surface.

## Membership and identity

Web identity comes from the authenticated email supplied by `exe.dev`. Skarbot uses that email to find a user record.

Membership is app-managed:

- authenticated users in `active/` go straight into the app
- authenticated users in `pending/` or `denied/` see a waiting screen
- unknown authenticated users enter onboarding

Onboarding collects:

- name
- email
- phone number
- country code
- timezone

Friendly phone input is normalized before storage. Email is normalized and used as the web identity key. Phone is normalized to E.164 and used as the SMS identity key.

The initial admin user is created during deployment setup, not through the public onboarding flow.

## Channel behavior

Web, SMS, and admin email all land in the user’s direct thread unless the interaction is explicitly routed into a task thread.

Channel rules:

- replies default to the same surface that last received the user
- attachments are saved locally at ingress and attached to the thread as normal user content
- unknown SMS or email senders are rejected before any model call
- unauthorized inbound attempts are logged
- outbound member email is not a general product surface
- outbound email is reserved for admin notifications and approvals

SMS is a communication surface and a notification surface. It is not an approval surface.

## Threads in the product

A user sees two kinds of threads:

- **Direct thread** — the long-lived personal conversation
- **Task thread** — scoped work that should not clutter the direct thread

The thread viewer makes that distinction explicit. Scheduled work runs in task threads. Capability-building work runs in task threads. Compaction boundaries are visible in the thread viewer so the conversation history remains legible.

## What Skarbot is optimized for

Skarbot is optimized for:

- one self-hosted deployment
- one admin
- a small set of approved users
- file-backed state
- thin product surfaces over a strong shared core

It is not trying to be a general collaboration suite, a broad SaaS agent platform, or a product with many partially-overlapping interfaces.

## See also

- [Architecture](architecture.md)
- [Runtime](runtime.md)
- [Deployment](deployment.md)
