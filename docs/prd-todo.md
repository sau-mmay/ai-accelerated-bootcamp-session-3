# Product Requirements Document (PRD) - TODO App Upgrade (MVP + Post-MVP)

## 1. Overview

This PRD defines the scope for upgrading the current basic TODO app (title + completed) into a more practical but still teachable version. The primary goals are to improve task planning and urgency management with minimal complexity.

Scope decisions come from two sources:
- Initial requirement discovery from the meeting transcript.
- Final scope lock from the Slack follow-up, which confirms MVP and defers selected enhancements.

The product direction is to deliver a lean MVP with due dates, priorities, and core date-based filters, while keeping data local and avoiding backend changes.

---

## 2. MVP Scope

- Add optional due date field on each task:
  - Field name: dueDate
  - Format: ISO date string YYYY-MM-DD
  - If invalid, treat as absent (ignore invalid value)
- Add task priority field:
  - Allowed values: P1, P2, P3
  - Default value: P3
- Keep existing task title requirement:
  - title is required
- Add filters:
  - All
  - Today
  - Overdue
- Maintain local-only storage:
  - No backend changes
  - No external storage integration
- Ensure MVP remains simple and teachable:
  - Implement only required behaviors above
  - Defer visual and advanced ordering enhancements to Post-MVP

---

## 3. Post-MVP Scope

- Visual overdue highlighting (for example, red styling for overdue tasks).
- Sorting rules:
  - Overdue tasks first
  - Then by priority (P1 to P3)
  - Then by due date ascending
  - Tasks without due dates last
- Optional UI enhancements discussed in discovery (for example, richer visual priority badges) if still desired after MVP delivery.

---

## 4. Out of Scope

- Notifications
- Recurring tasks
- Multi-user support
- Keyboard navigation enhancements
- External storage integrations

