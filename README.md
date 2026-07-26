# Resumable YouTube Live chat AI on Render

A cron-triggered Render web service that reconnects to one reusable YouTube
ingest stream, answers at most three previously unseen live-chat messages per
run through the Exa demo endpoint, and renders the current question and answer
into a generated 720p video feed.

Each authenticated `POST /tick` reads up to three unseen messages, generates
their answers, reconnects long enough to show each response card, and then
stops ingest without completing the YouTube broadcast. With no new messages it
briefly resends the latest card. Buffering between five-minute cron calls is
expected.

If a tick arrives while an older run is still working, the newer invocation
signals the old run to stop and takes the single encoder lock as soon as the old
RTMP connection closes. This gives the newest cron invocation precedence
without opening two simultaneous connections to the same YouTube stream key.

## State

YouTube resource IDs and OAuth credentials are Render environment variables.
The service keeps the latest processed `publishedAt` timestamp plus a bounded
set of message IDs in `/tmp`. The timestamp identifies newly published chat;
IDs disambiguate multiple messages with the same timestamp. This state survives
ordinary cron calls but not a Render restart or redeploy. A fresh instance sets
its baseline to its startup run, deliberately ignoring older chat history.

## Endpoints

- `GET /health` — process status, with no secrets
- `POST /tick` — starts a job when authorized with
  `Authorization: Bearer $CRON_SHARED_SECRET`

## Environment

See `render.yaml`. `YOUTUBE_STREAM_ID` must identify a reusable stream and
`YOUTUBE_BROADCAST_ID` must identify a non-completed broadcast bound to it.
