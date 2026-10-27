# Changelog

## 1.2.3 — 2026-10-28
- Expose safe Telegram configuration-presence flags in `/health` without exposing secret values.
- Log Telegram `getUpdates` and `deleteWebhook` API failures so polling conflicts and token/configuration errors are visible in Cloudflare logs.
- Validate Telegram `sendMessage` responses and surface failures instead of silently swallowing them.
