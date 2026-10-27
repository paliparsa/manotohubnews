# Changelog

## 1.7.0 — 2026-10-28
- Add Telegram editorial controls for topic categories, allow-list and block-list keywords, filter sensitivity, and maximum news age.
- Make filter changes versioned so recent feed items can be evaluated again under updated rules without automatically publishing historical items.
- Add persistent run diagnostics, rejection-reason counters, feed error summaries, and explicit logging/counting for failed admin draft notifications.
- Fix the pipeline's blanket seen-marking behavior by recording per-item rejection reasons and using filter-revision-scoped seen keys.
- Keep approval-before-publish and automatic publication behavior intact.

## 1.6.1 — 2026-10-28
- Fix the `/cancel` command regex that caused Wrangler's esbuild parser to fail, restoring deployability of the source-management update.

## 1.6.0 — 2026-10-28
- Add admin source management for listing configured RSS/Atom feeds, enabling/disabling sources, and deleting sources with confirmation.
- Allow admins to add public RSS/Atom feed URLs with optional language tags; validate feed URLs before saving.
- Persist source configuration in Durable Object storage and retain the existing default source list on first initialization.


## 1.5.0 — 2026-10-28
- Add persistent admin controls to turn channel publishing on/off and choose automatic publishing or approval before publishing.
- Queue prepared stories for admin review when approval mode is selected or publishing is switched off; support approve/publish and reject actions from Telegram inline buttons.
- Keep news collection and preparation running independently from channel publishing; expose publishing mode in the admin status screen.


## 1.4.2 — 2026-10-28
- Automatically register the Telegram webhook when missing or pointing to the wrong Worker URL, with a safe force-refresh option on `/health?setupWebhook=1`.
- Report webhook registration and delivery status from `/health` without exposing bot credentials; preserve queued Telegram updates during registration.

## 1.4.1 — 2026-10-28
- Correct the Cloudflare Cron Trigger to run every five minutes, matching the news pipeline schedule and removing idle one-minute invocations.

## 1.4.0 — 2026-10-28
- Replace one-minute Telegram getUpdates polling with an authenticated Telegram webhook for near-instant admin replies.
- Keep the five-minute Cron exclusively for the news publishing pipeline.
- Acknowledge webhook updates immediately and process them with the Worker execution context; log processing failures without exposing secrets.
- Require the `TELEGRAM_WEBHOOK_SECRET` Worker secret and validate Telegram's `X-Telegram-Bot-Api-Secret-Token` header.

## 1.3.0 — 2026-10-28
- Replace command-first admin help with a compact inline-button menu for status, model selection, OpenRouter key setup, connection testing, and guarded key deletion.
- Support Telegram callback queries through the existing Cron polling flow.
- Allow the admin to send the API key as a follow-up message after choosing the key button; validate it, best-effort delete the message, and store it encrypted.

## 1.2.3 — 2026-10-28
- Expose safe Telegram configuration-presence flags in `/health` without exposing secret values.
- Log Telegram `getUpdates` and `deleteWebhook` API failures so polling conflicts and token/configuration errors are visible in Cloudflare logs.
- Validate Telegram `sendMessage` responses and surface failures instead of silently swallowing them.
