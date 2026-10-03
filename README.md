# Daylight — On-Demand Services Platform

*Technical case study. Architecture and engineering decisions only — no code, no client data.*

## Overview

Daylight is an on-demand services platform with two connected interfaces: a client web app for requesting services and tracking their status, and an internal operations dashboard for managing fulfillment.

The platform ran live at Black Hat 2026, supporting event operations and the demos that contributed to approximately $5 million in contracts signed. That figure reflects the broader commercial outcome of the demos, rather than revenue attributable solely to the platform.

The technical focus was coordinating requests across both interfaces while supporting flexible authentication, live operational updates, and reliable notifications.

**Stack:** React, TypeScript, Tailwind CSS, Supabase (Postgres, Row Level Security, Auth, and Realtime), Vercel, Twilio (phone OTP), and a WhatsApp Business number hosted on Railway for messaging.

## Architecture decisions

### Managed backend with Supabase

Supabase was chosen to balance implementation speed with security requirements in a sensitive environment. Managed Postgres provided the data foundation, Row Level Security provided database-level access controls, and built-in Auth and Realtime reduced the amount of backend infrastructure to build and maintain.

The client web app and operations dashboard served different audiences, so access control was an architectural concern alongside request management and live updates. RLS policies restricted authenticated clients to creating and viewing their own requests, while authorized operations staff could access the operational queue and manage assignments and status changes. Enforcing these boundaries in Postgres meant access restrictions did not depend solely on frontend logic.

### Messaging on a WhatsApp Business number

The messaging layer ran on a WhatsApp Business number hosted on Railway, inside an app built around it — not through a third-party messaging API. That kept control over the sending flow, including delivery tracking, in one place.

### React and TypeScript across both interfaces

Both interfaces used React, TypeScript, and Tailwind CSS. This kept the frontend stack consistent while supporting two distinct workflows: clients requesting and monitoring services, and staff coordinating assignments and completion.

Supabase Realtime powered live updates in the operations dashboard, allowing staff to follow changes without refreshing the page.

### Authentication designed for an event

Daylight supported Google OAuth, email authentication, and phone OTP through Twilio. At an event, requiring every user to authenticate through the same method creates unnecessary friction. Providing three paths let users choose a method available to them.

### Deployment with Vercel

Vercel was chosen for preview deployments and zero-configuration hosting. Preview deployments made changes reviewable before they reached the live application, while managed hosting reduced deployment overhead.

## Hard problems

### Delivery-aware notifications

WhatsApp was the primary notification channel, but template restrictions and rate limits caused messages to fail silently. Sending a notification was not enough to establish that it had reached its recipient.

Delivery was tracked through the Railway logs: an automatic error-code handler watched for failed message states and triggered the SMS fallback without manual intervention. No critical notification depended on a single channel.

The distinction between a send attempt and successful delivery mattered operationally. A request could progress correctly inside the platform while the person relying on its notification remained unaware of the change.

### Phone-number normalization

Phone numbers arrived in inconsistent formats. The input flow had an explicit country selector and normalized numbers to E.164 at the boundary before they reached the messaging layer.

But registrations also came through service channels where people never specified a country — and those numbers still had to be registered. An automatic country handler took those inputs and converted them into real dialable number codes, so the messy path resolved to the same E.164 format as the clean one instead of failing or guessing silently.

## Operations at Black Hat

The operations dashboard provided a live request queue organized into pending, assigned, and done states. Realtime updates kept the queue current without page refreshes.

Staff could assign requests across the team and filter the queue by service type and priority. The dashboard also supported marking requests complete from a phone on the event floor, connecting fulfillment activity directly to the operational view.

The client web app included a status page so clients could follow their requests. An end-of-day summary provided a consolidated view of the day's activity.

Together, these capabilities supported the request lifecycle: client submission, operational assignment, fulfillment, and completion. Delivery-aware notifications complemented the live interfaces by providing an automatic alternative when the primary messaging channel failed.

## Limitations

Daylight was built for an event deadline. Its live use demonstrated that it could support the event workflow, but did not establish readiness for sustained operation at larger scale.

A longer-lived version would need audit logging for operational changes, further notification rate-limit hardening, and load testing. Those additions would improve traceability, make messaging behavior more predictable under constrained delivery conditions, and establish performance limits beyond the event setting.
