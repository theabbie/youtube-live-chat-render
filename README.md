# Resumable YouTube Live chat AI on Render

A cron-triggered Render web service that reconnects to one reusable YouTube
ingest stream, atomically claims one unanswered live-chat message per run
through Firebase Realtime Database, answers it through the Exa demo endpoint,
and renders the question and answer into a generated 720p video feed.

Each authenticated `POST /tick` claims at most one message, generates its
answer, reconnects only long enough to show that response card, and stops ingest
without completing the YouTube broadcast. With no unanswered message it exits
without starting FFmpeg. Buffering between cron calls is expected.

If a tick arrives while an older run is still working, the newer invocation
signals the old run to stop and takes the single encoder lock as soon as the old
RTMP connection closes. This gives the newest cron invocation precedence
without opening two simultaneous connections to the same YouTube stream key.

## State and concurrency

YouTube resource IDs and OAuth credentials are Render environment variables.
Message state lives at
`/youtubeLiveChatAI/<broadcast-id>/messages` in Firebase Realtime Database.
Transactional claims prevent overlapping workers from answering the same
message. Interrupted claims return to `pending`; abandoned `processing` claims
can be reclaimed after five minutes.

## Endpoints

- `GET /health` — process status, with no secrets
- `POST /tick` — starts a job when authorized with
  `Authorization: Bearer $CRON_SHARED_SECRET`

## Environment

See `render.yaml`. `YOUTUBE_STREAM_ID` must identify a reusable stream and
`YOUTUBE_BROADCAST_ID` must identify a non-completed broadcast bound to it.
`FIREBASE_SERVICE_ACCOUNT` is the service-account JSON string.
