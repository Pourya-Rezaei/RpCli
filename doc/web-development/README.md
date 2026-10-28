# Web Interface Development Plan

Date: `2026-10-28`; baseline version reviewed: `0.10.1`.
Status: the overall direction is agreed; implementation has not started, and open details require approval.

This directory records decisions and development rules for an assistant-ui-based web interface. Documents in other directories primarily describe current behavior; these documents specify proposed future behavior. A feature appearing in this plan does not mean it exists in the application.

## Reading Order

1. [Architecture decision and scope](architecture-decision.md)
2. [Development, security, and compatibility rules](development-rules.md)
3. [Update policy](update-policy.md)
4. [Delivery stages and acceptance criteria](delivery-plan.md)

## Decision Summary

Add a local web interface to the project, retaining the current DeepSeek connection and existing agent as the foundation. Do not remove or change the terminal interface, single prompt, commit, resume, or serve without authorization. The new interface should communicate with the core through a narrow, independent integration layer.

Opening the web interface with `rc` was initially proposed in discussion; preserving the CLI unchanged was subsequently emphasized. Changing the command's default behavior is therefore not currently considered authorized. The choice of a separate command or an opt-in web entry option must be finalized before implementing the launcher. No command proposed in these documents is an existing feature.

## Why Record It This Way?

A separate directory and a small architecture decision document prevent scattered discussions and confusion between future plans and the product reference. This structure complements the [existing maintenance guide](../development/documentation-maintenance.md), rather than replacing it. The `development/` directory guides development of the current product; this directory records this specific initiative.

The directory was originally named in Persian at the user's request; the files inside retained English names for references and tooling. The translated documents now reside in `web-development/`. For future plans, create separate topic-specific directories rather than duplicate versions of these files.

## Decision Change Control

Every decision must have a status, rationale, compatibility impact, and review criterion. Statuses are "proposed", "accepted", "implemented", and "superseded". Acceptance of the architecture does not mean tests have passed. Do not silently rewrite an implemented decision; record significant changes with a date, rationale, and link to the replacement decision.

These documents do not authorize commits, pushes, dependency installation, core changes, or public deployment. Each action must stay within the request for the implementation stage. Return to the [documentation index](../README.md).