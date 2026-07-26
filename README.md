# Resumable YouTube Live chat AI on Render

A cron-triggered Render web service that reconnects to one reusable YouTube
ingest stream, answers at most three previously unseen live-chat messages per
run through the Exa demo endpoint, and renders the current question and answer
into a generated 720p video feed.

Each authenticated `POST /tick` starts at most one background stream job.
The default job lasts 270 seconds and deliberately does **not** complete the
YouTube broadcast. A five-minute cron therefore reconnects to the same watch
page with a short buffering gap.

## State

YouTube resource IDs and OAuth credentials are Render environment variables.
A bounded list of processed chat IDs is kept in `/tmp` and survives ordinary
cron calls but not a Render restart or redeploy. Following a restart, existing
chat history may be considered again.

## Endpoints

- `GET /health` — process status, with no secrets
- `POST /tick` — starts a job when authorized with
  `Authorization: Bearer $CRON_SHARED_SECRET`

## Environment

See `render.yaml`. `YOUTUBE_STREAM_ID` must identify a reusable stream and
`YOUTUBE_BROADCAST_ID` must identify a non-completed broadcast bound to it.

