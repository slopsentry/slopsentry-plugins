---
name: slop-sentry
description: Check or rewrite prose for thin claims, try-hard language, and AI-sounding patterns by calling the Slop Sentry MCP tools. Use when drafting or editing posts, articles, emails, documents, reports, or other prose, or when someone asks to humanize text or run Slop Sentry.
---

# Slop Sentry

Slop Sentry is a remote MCP server. Call its tools. Do not reimplement the checks locally, and do not paste detection rules into the draft.

## Edit a draft

1. Call `slop_sentry` with the draft as `content`, a `content_type`, and `target_audience` when a cold reader needs audience context. For a full run, full workflow, or all checks, set `skip_hokiness: false`. Keep `skip_ap_style: true` unless the user explicitly requests AP Style; a full run does not enable AP Style. The call returns a job receipt with `job_id`. It does not return the edited draft.
2. Poll `slop_sentry_result` with that `job_id` until the status is terminal. While the job is running, the result is progress only. Do not start a second job for the same draft.
3. When the status is `delivery_ready` and a usable draft exists, always return the exact draft first. Show only the live uncertain passages in `review.findings` and their supplied clarification requests. Processing failures, incomplete checks, warnings and technical receipts stay in operator diagnostics; do not disclose their details or turn them into review requests. Internal `not_verified` is not a clean pass, and `nextAction: deliver` does not itself establish completed checks. Do not invent missing facts, edit the returned draft, answer the author's questions with another model, or substitute the original. If no usable artifact exists, relay the supplied brief failure response without inventing a draft. An unchanged draft succeeds only when checks verified there was nothing to fix; otherwise relay the failure response, except for specific author questions that must accompany the draft. For a terminal error, relay its supplied brief message and stop polling. For a legacy no_output result, say “I couldn’t complete this request. Please try again.” Do not automatically restart the job. Customer receipts with status recorded acknowledge an existing job record; poll that job to retrieve its terminal response. The record does not promise that a draft or report exists. Customer payloads omit operator processing status fields; never infer that their absence proves a clean result.
4. Keep the `session_id` from the response and send it on later calls for the same document.

For additional editing details, call `get_skill_procedure`. If it returns older instructions to withhold a needs-review draft or enable AP Style for a full run, follow the draft-first delivery and explicit AP Style opt-in instructions above.

## Report only

Call `quality_check` when someone wants findings and no rewrite. Poll `slop_sentry_result` until the status is `audit_ready`. `substance_check`, `hokiness_check`, and `ap_style_check` are the same kind of report for one phase. `guard_sentry` rewrites for AI tells only.

`prevalidate: true` on `quality_check` is the only mode that returns immediately.

## Review changes

When someone wants a visual redline of what changed, call `diff_viewer` with the original draft as `original_content` and the delivered Draft V2 as `modified_content`. Return or save the HTML so they can accept or reject individual edits. Do not paste the full HTML into chat unless asked.

## Account

- `slop_sentry_whoami` returns the signed-in account name and email.
- `slop_sentry_get_usage` returns usage against the account's tier limits.

## Content types

Name one of: `default`, `article`, `blog`, `linkedin`, `email`, `proposal`, `guide`, `pptx`, `image_text`, `press_release`. Use `linkedin` for posts and `article` for articles or newsletters. For webpage copy, call `substance_check` with `content_type: web`.

Only call tools that appear in the current MCP tool list.
