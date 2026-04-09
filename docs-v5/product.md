# Product

This document defines Skarbot from the user’s point of view: what it is, who it serves, which channels it supports, and how membership works. Persistent file layout lives in [Architecture](architecture.md). Runtime behavior and tool contracts live in [Runtime](runtime.md)

## Identity

Skarbot is a personal claw-like 24/7 AI assistant for a household. One deployment serves a household. The deployment has one admin who owns the VM

Each approved person gets their own long-lived user thread with Skarbot. Family members share the deployment, but they do not share one conversation

## Interacting with Skarbot

Skarbot supports four ways for users to interact with it:

| Method          | Description                                              |
| --------------- | -------------------------------------------------------- |
| Web app         | Main interactive interface                               |
| SMS             | Lightweight mobile interaction and notifications         |
| Admin email     | Operational inbox for admin communications and approvals |
| Scheduled tasks | Time-based automation bound to user-defined tasks        |

Whether someone uses the web app, SMS, admin email, or schedules tasks, Skarbot keeps the interaction tied to that user’s own conversations and configured behavior

## Web app

The web app is the primary interface. It allows interacting with each member's long-lived user thread and individual task threads

The web app includes:

- a home page and docs that explains what Skarbot is and how to use it
- a user thread and task thread chat interface with ability to add attachments
- a simple onboarding flow for authenticated people who are not yet approved users
- a waiting screen for users whose approval is still pending or denied
- basic logs and health status visibility

### Chat interface

The chat interface makes two distinctions explicit:

- user threads versus task threads
- ordinary message history versus compaction boundaries and tool calls

Chat and task-thread timestamps are rendered in the viewer’s local time

## Membership and identity

Web identity comes from the authenticated email supplied by `exe.dev`. Skarbot uses that email as the primary web membership lookup key

Membership is app-managed:

- authenticated users with records in `~/skarbot/state/users/active/` go straight into the app
- authenticated users with records in `~/skarbot/state/users/pending/` or `~/skarbot/state/users/denied/` see the waiting screen
- authenticated users with no existing user record enter onboarding

Onboarding collects:

- first and last name
- email
- phone number
- country code
- timezone

If Skarbot can infer a timezone during onboarding using location data, it proposes that value, but the user must still confirm or correct it before submission

The user id is derived from the submitted first and last name. Email and phone number are normalized and tied to a specific user

If onboarding submits a first and last name, phone number, or email address that collides with an existing user record, onboarding fails immediately and does not create a new record

Once onboarding is complete, the admin receives an email asking to approve the user. After approval, users cannot change their own membership record fields; those changes must be made by the administrator on the filesystem

The initial admin user is created during deployment setup, not through the public onboarding flow

## Channel behavior

Web, SMS, and admin email all land in the user thread unless the interaction is explicitly routed into a task thread

Channel rules:

- replies default to the same channel the user last responded from
- unknown SMS or email senders are rejected before any model invocation
- unauthorized inbound SMS and email attempts are logged to a deterministic runtime log
- unknown authenticated web users are routed into onboarding instead of the main app
- outbound member email is not a general product channel
- outbound email is reserved for admin notifications and approvals

User approvals may occur over web and SMS. Admin approvals occur over email

## Threads in the product

A user sees two kinds of threads:

- **User thread** — the long-lived personal conversation
- **Task thread** — scoped work that should not clutter the user thread

Scheduled work runs in task threads. Capability-building work runs in task threads

## What Skarbot is optimized for

Skarbot is a personal AI assistant for households that is designed to be self-hosted by a single administrator and serve a small set of approved users. The runtime relies on the filesystem for state and seeks to be as simple as possible

It aims to have as few integrations as possible out of the box while being extensible enough to support any user's needs over time

## See also

- [Architecture](architecture.md)
- [Runtime](runtime.md)
- [Deployment](deployment.md)
