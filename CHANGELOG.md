# Changelog

## 1.18.0 — 2026-10-28
- Add Persian inline-button management for social sources: list, add, enable/disable, and delete.
- Ingest new posts from configured public Telegram channels through bot channel-post updates; refresh the webhook subscription when channel-post updates are missing.
- Support X and Truth Social account records through user-configured RSS/Atom bridge URLs, with source attribution preserved through AI curation.
- Queue Telegram posts for the existing AI relevance, Persian translation, and publication pipeline; clean processed posts while retaining unprocessed items.
- Document platform access requirements: Telegram bot must be an administrator in each channel; X and Truth Social require a reachable RSS bridge in this version.

## 1.17.0 — 2026-10-28
- Serialize RSS requests per hostname to avoid bursting Google News RSS with concurrent queries.
- Increase retry handling for transient RSS HTTP errors to four attempts with exponential backoff and structured retry logs.
- Report successful-but-unparseable feed responses as source errors instead of silently treating them as healthy.

## 1.16.0 — 2026-10-28
- Align the default maximum news age with the RSS discovery window (7 days) instead of silently dropping relevant reports after 24 hours.
- Migrate the legacy 6h/24h default to 168h once, preserve explicit 72h/336h/unlimited settings, and bump the filter revision so previously age-rejected stories can be reconsidered.
- Expand priority-news detection for US military and Gulf-region security reporting in English, Persian, Hebrew, and Arabic, so relevant regional stories bypass the age filter.
- Add the underlying pipeline error to the admin health report when AI returns zero items, making model failures diagnosable from Telegram.


## 1.15.0 — 2026-10-28
- Stop advertising retired MiniMax M3.1 Flash `:free`; when OpenRouter's live catalog is available, use it as authoritative instead of permanently merging stale fallback IDs.
- Exclude NVIDIA-hosted models from the free-model catalog and curation attempts after repeated empty-output failures in connection tests; the explicitly excluded Nemotron 3 Ultra remains excluded.
- Filter cached and fallback catalogs against known unusable IDs, shorten fallback cache lifetime, and expand live catalog timeout to reduce stale-model persistence.
- Improve connection tests for reasoning models by increasing test output budget and parsing both string and text-block message content; report more specific diagnostics for empty responses.
- Keep all model selection restricted to free `:free` endpoints; no paid fallback added.

## 1.14.0 — 2026-10-28
- Restore the missing five-minute Cron handler: each scheduled event now awaits the RSS → AI curation → publication pipeline and logs start, result, and unhandled failure.
- Bound scheduled AI curation to three free-model attempts with a 25-second per-model timeout, avoiding long sequential stalls.
- Reject empty or incomplete AI outputs instead of silently accepting relevant stories without Persian headlines/summaries or source IDs, then try another free model.
- Ask for at most eight concise Persian summaries per run and cap output tokens to reduce truncated JSON and partially translated posts.


## 1.13.2 — 2026-10-28
- Fix manual web search appearing to hang: try at most two free models with a 10-second per-model timeout instead of serially trying the entire free catalog for up to 90 seconds per model.
- Shorten free-model catalog discovery timeout to 2.5 seconds so webhook background work is less likely to be terminated before a reply is sent.
- Limit each manual search to three candidate stories to keep Telegram responses within the Worker background execution window; surface a clear failure message when both attempts fail.


## 1.13.1 — 2026-10-28
- Explicitly exclude NVIDIA Nemotron 3 Ultra from both discovered and fallback model lists, as requested.
- Remove the free router that could randomly select the excluded model; use explicit `:free` endpoints only.
- Keep dynamic discovery of all other zero-priced OpenRouter text models and legacy paid-model preference rejection.


## 1.13.0 — 2026-10-28
- Remove any possibility of choosing or falling back to a paid model: model selection only accepts OpenRouter's explicit `:free` variants plus the free `openrouter/free` router.
- Discover the current OpenRouter free text-model catalog dynamically, cache it for 12 hours, and include newly available free models without requiring code edits.
- Remove reliance on the expiring Laguna-only fallback; legacy saved model preferences that are no longer free are ignored.
- Keep paid web search out of scheduled runs; free-model calls may still be subject to OpenRouter free-tier rate limits.


## 1.12.0 — 2026-10-28
- Stop running paid Exa web search automatically every 15 minutes; scheduled news discovery now uses RSS feeds only.
- Keep RSS ingestion scheduled every five minutes and retain broad Iran/Islamic Republic, Reza Pahlavi and team, Israel, US relations, war and security coverage.
- Keep the existing OpenRouter `:free` model catalog for curation and summaries; do not introduce paid-model fallbacks.
- Preserve manual/admin-triggered web search as an optional action rather than a recurring cost.
- Keep single-source preliminary reports and existing duplicate prevention/publication policies intact.


## 1.11.0 — 2026-10-28
- Add automatic focused web search every 15 minutes for Iran, the Islamic Republic, Reza Pahlavi and his team, Israel, US relations, war and regional security; publish/queue results according to the existing publication policy.
- Remove the two-independent-source rejection gate; single-source stories are kept and labeled as preliminary.
- Relax age, allow/block keyword, topic-toggle and AI-confidence filters for priority Iran/Israel/US/Pahlavi stories; expand RSS search windows to seven days and add targeted Persian/English feeds.
- Preserve duplicate prevention and real source links; do not fabricate URLs or facts.


## 1.10.0 — 2026-10-28
- Replace manual topic search's Google News RSS dependency with OpenRouter's `openrouter:web_search` server tool.
- Let the model run multiple web searches across online sources, synthesize up to eight distinct stories, and return source links with each candidate.
- Keep manual search separate from scheduled RSS ingestion; stories remain pending until the admin explicitly approves publication.
- Update Telegram copy to explain web search coverage and limitations.


## 1.9.1 — 2026-10-28
- Explain why no stories reach AI by tracking previously seen items, duplicate feed items, unique candidates, and the no-candidate reason in diagnostics.
- Add bounded retries for transient feed HTTP errors (429/500/502/503/504) and temporary fetch failures.
- Show enabled feed count and zero-AI-candidate explanation in the Telegram run report.


## 1.9.0 — 2026-10-28
- Add a recoverable review queue with retryable publish failures and retryable admin notifications; failed stories remain persisted instead of disappearing.
- Retain rejection metadata and source details for 14 days, expose rejected stories in Telegram, record manual admin rejections, and support selective reprocessing under current editorial rules without auto-publishing.
- Add a no-publication dry run over up to 30 sample stories, reporting pre-filter rejects, AI accepts, manual-review candidates, and rejection reasons without adding drafts or publishing to the channel.
- Expand publishing policy to direct publish, admin approval, save-only, and temporary pause; collection and preparation continue while publication is paused.


## 1.8.0 — 2026-10-28
- Add persistent custom editorial topics with add/edit/delete controls in Telegram; deleting a custom topic removes it from active editorial filters without touching sources or published posts.
- Add manual AI-assisted topic search using Google News RSS in Persian and English, source-grounded summaries, and per-story approve/publish or reject buttons. Manual search never publishes without explicit approval.
- Keep default topics, source management, scheduled pipeline, and existing publication controls intact.


## 1.7.1 — 2026-10-28
- Check the active filter-revision seen keys before applying early filters, avoiding repeated writes and inflated rejection counts for already evaluated items.

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
