---
name: spun-mcp
description: Use the Spun MCP server to read and act on a business's WhatsApp conversations, groups and contacts. Covers sign-in, access levels, send approval, the read allowance, what cannot be read, and how to answer "did it send".
---

# Spun MCP

## When to use

Use this skill when you are connected to the Spun MCP server (`https://api.spun.com/api/mcp`) and the person asks about their WhatsApp conversations, groups, contacts, digests or Spun itself, or asks you to send or schedule a message. Read it before your first send, and before you tell anyone a message went out.

## What Spun MCP is

Spun is a WhatsApp inbox for teams. Its MCP server lets the person's own AI client (ChatGPT, Claude, Gemini, Claude Code or another MCP client) read their WhatsApp chats and work with them through Spun. Every capability is a scope on the connection: a connection only sees the tools its scopes allow, and every other tool does not exist for it.

## Endpoint and sign-in

- Server address: `https://api.spun.com/api/mcp`
- claude.ai, ChatGPT and the Gemini app add that address without an API key. claude.ai: Customize, Connectors, Add, Add custom connector. ChatGPT: Plugins, Add, Add custom MCP server (ChatGPT calls it a plugin or a custom MCP server, not a connector). Gemini app: Settings & help, Connected Apps, Add a custom app. The client sends the person to a Spun sign-in page; they use their Spun login. On the consent screen an organization manager picks the organization and the access level. Only organization managers can approve the consent screen.
- Claude Code: add the server, run `/mcp`, choose spun and Authenticate to sign in in the browser. Headless setups pass `Authorization: Bearer wap_...` with a Spun API key instead.
- API keys are created in the Spun web app under Settings, AI Assistant (MCP), Manage API keys, by organization managers, from the same presets. The key value is shown once and cannot be retrieved again.
- An organization manager can revoke any connected app later.

## Access levels and what each refuses

The connection starts from one of three access levels (presets). An organization manager can later narrow a connected app to the Support, Sales or Reporting preset or to chosen permission groups.

- **Read-only analyst**: reads. It can never send; send tools are not on the connection.
- **Assistant (no sends)**: an assistant that can never send.
- **Assistant with sends**: can propose messages. Each message it proposes waits for a person to approve it, unless the organization has opted into the Autonomous posture for this app.

The consent screen never grants autonomous sending. Only an organization manager can turn it on for a connected app afterwards, under Connected AI apps in Spun settings, and re-consenting resets it. Call `get_my_permissions` (never counted) to see the scopes, the send posture, quotas and quiet hours for this connection before you plan work.

## Approval etiquette

- Propose, never assume. A send tool submits a request; it does not mean the message went out.
- The organization's send posture decides what happens next:
  - `draft_only`: the request becomes a draft in the team inbox. A person sends, edits or discards it.
  - `two_step` (the default): the request waits in the AI sends queue in the Spun inbox. A teammate with send permission approves it, and Spun runs fresh checks first (channel state, quotas, whether the customer replied since you wrote it). Unapproved items expire in about 24 hours.
  - `autonomous`: sends run immediately, only when the org posture is autonomous and the connection holds the autonomous-send grant. Quiet hours, quotas and the kill switch still apply.
- `send_message` needs a `conversation_id` or `contact_id` (never a raw phone number), a UUID `idempotency_key` and an `action_reason` that the approver sees. Pass `expected_last_message_id` from `list_messages` so the approver is warned if the conversation moved on.
- `preview_send` is a dry run: it reports whether a submission would be accepted right now and lists warnings such as quiet hours. Nothing is queued.
- Tell the person what you submitted and that it is waiting for approval. Keep the `request_id` the send tool returns.

## How to answer "did it send"

Never answer from memory. Call `get_delivery_status` with the `request_id` (or the `idempotency_key`, or a time range of at most 7 days). It is never counted. Then report the `status` as it is:

- `sent`: it went out. Say so.
- `draft`: saved as a draft; a person still has to send it.
- `pending_approval`: waiting in the AI sends queue for a person to approve.
- `approved`, `held_quiet_hours`, `executing`, `provider_accepted`: on its way, not confirmed sent. Say which one.
- `rejected`, `expired`, `cancelled`, `discarded`: it did not go out. Mention `rejection_reason` if there is one.
- `failed`, `unknown`, `blocked_at_delivery`: say plainly that it was not confirmed, and include `provider_error` if present.

If `human_edited` is true, a person changed the text before it went out.

## Read allowance

- Free MCP: 100 AI calls a month. Each AI read counts, and so does each contact the AI creates or edits.
- Paid plans: 300 connected-AI reads during the 7-day trial, 2,000 a month on Starter, unlimited within fair rate limits on Pro and Max. Only reads that return the organization's data count. Reads never use AI credit.
- Never counted, on any plan: sends and other actions (except on Free MCP, where contact creates and edits count: `create_contact`, `update_contact`), status and permission checks (`get_delivery_status`, `get_my_permissions`), usage summaries (`get_usage_summary`), `get_business_overview`, and questions about Spun itself (`search_knowledge`).
- To check the allowance, call `get_usage_summary`. It returns the AI calls used this period, how many are left, the reset date, and the exact list of tools this connection can call. Ask it instead of guessing, especially before a large job. The person sees the same numbers in Spun under Settings, Connected AI.

## What cannot be read

- **Media on Free MCP**: for photos, voice notes and other media, the AI sees that one arrived, not what is in it. On paid plans, `get_media` returns images inline (size-capped), pre-computed voice-note transcripts and the text of PDF, Word, Excel and PowerPoint files, within per-key daily media limits.
- **Hidden members**: a group member who hides their number is marked hidden (`sender_hidden`) and carries a stable `sender_key` instead of a number. You can group that member's messages, but you never see the number.
- **Scanned PDFs**: a PDF with no text layer comes back unreadable. There is no text recognition (OCR).
- **Video**: metadata only. Its content cannot be read.

## Limits on Free MCP

Some tools listed under Free MCP below refuse part of their work on the Free plan, and the refusal says so:

- `get_media` returns metadata only (type, name and size). The `content`, `transcript` and `text` modes are on Starter.
- Scheduled sends are on paid plans: `schedule_message`, `cancel_scheduled_message`, `cancel_scheduled_messages`, `reschedule_scheduled_message` and `retry_scheduled_message` are refused. `list_scheduled_messages` stays open. Offer `send_message` instead.
- Message history is the last 7 days. Ask for a recent period.
- Up to 200 different contacts a month. Notes the AI adds to contacts are append-only.

## Tool families

<!-- BEGIN GENERATED: tool families. Rewritten by node scripts/export-mcp-skill.mjs from server/services/mcp/toolBundles.ts and home/src/lib/mcpToolPlans.json. Do not edit by hand. -->

174 tools in 37 families. 77 are included in Free MCP; the rest need a paid plan. Some Free tools refuse paid-only modes (see Limits on Free MCP above). A tool can belong to more than one family. What one connection can call also depends on the access level chosen at sign-in; `get_usage_summary` lists the tools your key has.

### orientation

- Free MCP: `get_my_permissions`, `get_delivery_status`, `get_usage_summary`, `list_channels`, `search_knowledge`
- Paid plans: `get_business_overview`

### messaging_read

- Free MCP: `list_conversations`, `get_conversation`, `list_messages`, `list_scheduled_messages`, `get_media`, `list_calls`, `search_messages_and_contacts`
- Paid plans: `get_follow_up_queue`, `list_voice_notes`

### messaging_outbound

- Free MCP: `preview_send`, `send_message`, `schedule_message`, `cancel_scheduled_message`, `cancel_scheduled_messages`, `reschedule_scheduled_message`, `retry_scheduled_message`, `create_media_upload`, `send_media`, `edit_message`, `delete_message`, `forward_message`, `send_location`, `send_contact_card`, `send_poll`, `manage_reaction`, `set_typing`

### chat_management

- Free MCP: `get_chat_language_settings`, `update_chat_status`, `manage_pin`, `manage_star`, `list_labels`, `create_label`, `update_labels`, `assign_conversation`, `list_team_members`

### crm_read

- Free MCP: `search_contacts`, `get_contact`, `get_contact_timeline`

### identity_read

- Free MCP: `get_chat_identity`, `list_identity_candidates`, `find_chat_by_message`, `list_unidentified_chats`, `list_identity_changes`, `list_identity_decisions`

### phone_check

- Paid plans: `check_number_on_whatsapp`

### crm_write

- Free MCP: `create_contact`, `update_contact`

### workspace

- Free MCP: `list_tasks`, `list_projects`
- Paid plans: `create_task`, `update_task`, `complete_task`, `create_project`, `list_business_signals`, `update_signal`, `list_campaigns`

### groups

- Paid plans: `list_groups`, `get_group`, `create_group`, `add_group_participants`, `remove_group_participants`, `manage_group_admins`, `update_group_settings`, `get_group_invite`, `revoke_group_invite`, `leave_group`, `list_communities`, `get_community`

### status_newsletters

- Paid plans: `list_status_updates`, `get_status_viewers`, `get_status_media`, `post_text_status`, `post_media_status`, `list_newsletters`, `search_newsletters`, `get_newsletter_messages`, `follow_newsletter`, `unfollow_newsletter`

### presence_profile

- Paid plans: `get_presence`, `get_my_profile`, `get_business_profile`, `update_profile`

### commerce

- Paid plans: `list_products`, `get_order`

### ai_compute

- Paid plans: `draft_reply`, `translate_text`

### analytics

- Paid plans: `get_message_stats`, `get_staff_activity`, `get_group_activity`

### digests

- Paid plans: `get_chat_summary`, `list_chat_summaries`, `get_period_report`, `get_period_report_evidence`, `get_period_report_run`, `list_period_reports`

### digests_manage

- Paid plans: `define_chat_summary`, `update_chat_summary`, `start_period_report`, `cancel_period_report_run`

### calendar

- Free MCP: `list_calendar_events`, `get_calendar_event`, `check_calendar_conflicts`
- Paid plans: `create_calendar_event`, `update_calendar_event`, `cancel_calendar_event`, `restore_calendar_event`, `add_event_attendees`, `remove_event_attendee`, `set_event_reminders`, `send_event_invites`

### tasks

- Free MCP: `get_task`, `list_task_comments`
- Paid plans: `add_task_comment`, `toggle_checklist_item`, `add_checklist_item`, `archive_task`, `unarchive_task`, `set_task_followers`, `bulk_update_tasks`

### calendar_read

- Free MCP: `list_calendar_events`, `get_calendar_event`, `check_calendar_conflicts`

### tasks_read

- Free MCP: `list_tasks`, `list_projects`, `get_task`, `list_task_comments`

### feedback

- Paid plans: `report_feedback`

### knowledge_read

- Paid plans: `kb_list_articles`, `kb_get_article`, `kb_search`, `kb_list_suggestions`

### knowledge_write

- Paid plans: `kb_upsert_article`, `kb_retag_article`, `kb_delete_article`, `kb_review_suggestion`

### bots_read

- Paid plans: `list_bots`, `get_bot`, `list_auto_reply_rules`, `list_auto_reply_log`, `list_bot_conversations`, `get_bot_transcript`

### bots_write

- Paid plans: `create_bot`, `update_bot`, `set_bot_active`, `create_auto_reply_rule`, `update_auto_reply_rule`

### pipeline_read

- Free MCP: `list_pipelines`, `get_pipeline_board`, `get_contact_stage`, `list_stage_history`, `get_pipeline_metrics`

### pipeline_write

- Paid plans: `move_stage`, `bulk_move_stage`, `undo_stage_move`

### circles_read

- Free MCP: `list_circles`, `get_circle`, `list_contact_circles`

### circles_write

- Free MCP: `create_circle`, `update_circle`, `add_circle_members`, `remove_circle_members`, `delete_circle`

### alarms_read

- Free MCP: `list_alarms`

### alarms_write

- Paid plans: `create_alarm`, `dismiss_alarm`

### keep_in_touch_read

- Free MCP: `list_keep_in_touch`

### keep_in_touch_write

- Paid plans: `set_keep_in_touch`, `clear_keep_in_touch`

### quick_replies_read

- Free MCP: `list_quick_replies`, `get_quick_reply`

### chat_language_write

- Paid plans: `set_chat_language_settings`

### identity_write

- Free MCP: `link_chat_to_contact`, `unlink_chat`, `name_hidden_contact`, `keep_contacts_separate`

<!-- END GENERATED: tool families -->

<!--
SOURCES (W-AEO-SKILL, 2026-11-08). M = home/src/pages/mcp/mcpCopy.ts; tools = server/services/mcp/tools/.
- Endpoint: home/src/pages/Mcp.tsx:60.
- Scopes, tools a key cannot see, presets, narrowing to Support/Sales/Reporting or groups: M:82, M:128, M:411.
- Sign-in for claude.ai, ChatGPT, Gemini, Claude Code, Bearer key for headless: M:327-330. OAuth, org manager picks org and access level, revoke: M:318.
- API keys in Settings, API Keys, managers only: M:406-411.
- Read-only analyst and Assistant (no sends) never send; Assistant with sends waits for approval unless Autonomous: M:670.
- Consent screen never grants autonomous; manager enables it under Connected AI apps; re-consent resets: M:327.
- Postures draft_only / two_step / autonomous: M:382-401. Approval expiry about 24 hours: M:393, M:464.
- send_message inputs and get_delivery_status(request_id): tools/sendTools.ts:1322-1331. preview_send: tools/sendTools.ts:1658-1663.
- get_delivery_status inputs, 7-day range: tools/orientationTools.ts:505-508. Status values: tools/orientationTools.ts:385-401. Row fields (rejection_reason, provider_error, human_edited): tools/orientationTools.ts:412-438.
- get_my_permissions: tools/orientationTools.ts:145-150.
- Free MCP 100 calls, reads and contact creates/edits count, sends/status/usage/Spun questions never count: M:612, M:624.
- Paid reads 300 / 2,000 / unlimited, only data reads count, no AI credit: M:436, M:731. Usage in Settings, Connected AI, usage summary tool: M:731.
- Uncounted tools (get_my_permissions, get_usage_summary, get_business_overview, get_delivery_status, search_knowledge): server/services/mcp/mcpCallAllowance.ts:86-104. Contact writes counted on Free MCP only (create_contact, update_contact): server/services/mcp/mcpCallAllowance.ts:115, :127.
- get_usage_summary fields: tools/wave7Tools.ts:852-857.
- Media on Free: M:626. Paid media: M:124, M:355, M:452, tools/mediaTools.ts:830-838.
- Free MCP limits: get_media modes server/services/mcp/freeMcpCapabilities.ts:36-40, :161-164; scheduled sends and schedule writes :28-35, :145-158; 7-day history :18, :27, M:617; 200 contacts and append-only notes M:614-615.
- Hidden members: M:735, tools/conversationTools.ts:361.
- Scanned PDFs and video: tools/mediaTools.ts:836-837, :808.
- Tool families: generated from server/services/mcp/toolBundles.ts and home/src/lib/mcpToolPlans.json.
NOT CLAIMED: how any assistant ranks, cites or picks tools; delivery or read receipts; any price; any number not in M `free.points`, M `free.rules` or the limits table; which tools a given connection holds (get_usage_summary answers that).
-->
