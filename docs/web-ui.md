# Web UI

Status: Research

## Role

- The web UI should stay thin and reuse the same conversations, tasks, and routing state as other surfaces
- The initial web UI should ride on `exe.dev`'s built-in HTTPS proxy
- The main app proxy may be public in v1, with Skarbot enforcing login and membership inside the app

## V1 Scope

- A home page that explains what Skarbot is
- A conversation and thread viewer that acts as the main chat-style UI
- File upload and attachment display inside direct threads
- Basic logs and status visibility
- An onboarding form for authenticated web users who are not yet members
- A waiting screen for users whose membership is still pending or denied
- The thread viewer should clearly distinguish between direct threads and task-linked threads
- The thread viewer should render compaction boundaries as first-class thread events

## Access and Visibility

- Protected routes should redirect unauthenticated users through `exe.dev` login
- The app should use the proxy-provided authenticated email as the primary web membership lookup key in v1
- Approved members should be allowed into the main app based on Skarbot membership records rather than `exe.dev` sharing alone
- Authenticated users already present in `active` should go straight into the app instead of seeing onboarding again
- Authenticated users already present in `pending` or `denied` should see the waiting screen instead of creating a duplicate record
- Onboarding should fail fast when the submitted phone collides with a different existing user record, and that failure should not create a new record
- Membership approval itself does not need a dedicated UI in v1; the admin flow can stay filesystem-driven
- Admin approvals do not require a dedicated web queue in v1 because the primary admin approval surface is the `exe.dev` email path
- Direct-thread visibility should be scoped by authenticated `exe.dev` user identity
- Task-linked threads should remain visible as task threads rather than being mixed indistinguishably with direct threads
- Compaction events should be visible in the web UI thread viewer, but they should not also appear in separate status, notification, or approval surfaces
- The UI should not be the primary editing surface for task files
- Chat and task-thread timestamps should be rendered in the viewer's local time in v1
- Web uploads should be stored as local thread attachment artifacts and converted at ingress into substrate-compatible user-message content following the `pi-web-ui` attachment pattern
