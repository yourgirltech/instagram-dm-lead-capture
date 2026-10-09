# Instagram DM Auto-Reply + Lead Capture

An n8n workflow that answers Instagram Direct Messages for a business account and logs every conversation as a lead in Supabase. Built for a business Instagram inbox: common questions get an instant, consistent answer, and nothing that arrives in DMs gets lost.

## How it works

```
Meta webhook (GET)  → verify token check → echo hub.challenge  (or 403)
Meta webhook (POST) → 200 EVENT_RECEIVED immediately (Meta retries slow endpoints)
  → parse sender + text → keyword intent router
      pricing │ location │ hours  → templated reply
      anything else              → LLM fallback reply
  → send the reply through the Instagram Graph API → log the lead to Supabase
```

- **Fast acknowledgement.** Meta gets its 200 before any processing, so slow AI calls never cause duplicate deliveries.
- **Cheap, predictable answers first.** Known intents are matched with keywords and answered from templates. The LLM handles only the long tail.
- **No secrets in the workflow.** The webhook verify token and page access token are read from environment variables (`KORET_IG_VERIFY_TOKEN`, `KORET_IG_PAGE_ACCESS_TOKEN`).

## Setup

1. Import `n8n/ig-dm-auto-reply-lead-capture.json` into n8n.
2. Set the two environment variables on your n8n instance, and allow expressions to read them (`N8N_BLOCK_ENV_ACCESS_IN_NODE=false`).
3. Create a **Supabase** credential and a `leads` table for the log node. Configure the **LLM: Fallback Reply** node with your model credential and prompt.
4. Replace the `REPLACE ME` template replies with the business's real pricing, location and opening hours.
5. In the Meta developer console, subscribe the Instagram account's `messages` webhook to the workflow's production URL, using the same verify token.
