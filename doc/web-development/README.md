# Open WebUI Integration Plan

Date: `2026-10-28`; baseline version reviewed: `0.10.1`.
Status: the overall direction is agreed; implementation has not started, and open details require approval.

This directory records the agreed direction for connecting the independently maintained Open WebUI application to RpCli. No custom chat frontend will be built. Documents elsewhere primarily describe current behavior; this directory describes proposed integration work, not implemented capabilities.

## Reading Order

1. [Architecture decision and scope](architecture-decision.md)
2. [Development, security, and compatibility rules](development-rules.md)
3. [Update policy](update-policy.md)
4. [Delivery stages and acceptance criteria](delivery-plan.md)

For a new execution session, use the separate [agent handoff](../agent-handoff/README.md), [ordered checklist](../agent-handoff/checklist.md), and [progress log](../agent-handoff/progress.md). These track execution and owner gates without duplicating architectural decisions.

## Decision Summary

Use Open WebUI as the conversation interface while retaining RpCli's existing DeepSeek web-token connection, PoW, transport, and agent. Preserve terminal, single-prompt, commit, resume, and serve behavior. Prefer configuration and supported extension points; a small integration adapter is permitted only when a demonstrated incompatibility requires it. Do not rewrite the agent or modify Open WebUI source by default.

Implement and test chat first. Validate conversation correctness before adding file tools in order: read, create, edit, delete. Shell execution is outside this delivery sequence. Do not change what bare `rc` does or add a launcher in the first milestone; use the existing `rc serve` entry point and run Open WebUI separately.

## State Management Decision

Do not add Redux, Zustand, or another application state package for this integration. RpCli currently uses React hooks for terminal UI state and module-level variables for backend sessions and cancellation. Open WebUI owns its own interface state. If session isolation requires work, solve it at the backend integration boundary rather than introducing a second frontend store. See the evidence and ownership rules in the [architecture decision](architecture-decision.md).

## Why Record It This Way?

A separate directory and a small architecture decision document prevent scattered discussions and confusion between future plans and the product reference. This structure complements the [existing maintenance guide](../development/documentation-maintenance.md), rather than replacing it. The `development/` directory guides development of the current product; this directory records this specific initiative.

All documents and paths remain in English. The previous custom-interface proposal has been replaced, not retained as an alternative execution plan. For future initiatives, create separate topic-specific directories rather than duplicate versions of these files.

## Decision Change Control

Every decision must have a status, rationale, compatibility impact, and review criterion. Statuses are "proposed", "accepted", "implemented", and "superseded". Acceptance does not mean tests have passed. This revision replaces an unimplemented design at the owner's request; implemented decisions require a dated migration record rather than silent replacement.

These documents do not authorize commits, pushes, dependency installation, core changes, or public deployment. Each action must stay within the request for the implementation stage. Return to the [documentation index](../README.md).