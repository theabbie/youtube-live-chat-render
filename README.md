# Resumable YouTube Live chat AI on Render

A cron-triggered Render web service that reconnects to one reusable YouTube
ingest stream, atomically claims one unanswered live-chat message per run
through Firebase Realtime Database, answers it through the Exa demo endpoint,
and renders the question and answer into a generated 720p video feed.

Each authenticated `POST /tick` claims at most one message, generates its
answer, reconnects long enough to show that response card, and stops ingest
without completing the YouTube broadcast. With no unanswered message it
re-streams the latest card as a keepalive. Buffering between cron calls is
expected.

If a tick arrives while an older run is still working, the newer invocation
signals the old run to stop and takes the single encoder lock as soon as the old
RTMP connection closes. This gives the newest cron invocation precedence
without opening two simultaneous connections to the same YouTube stream key.

## State and concurrency

YouTube resource IDs and OAuth credentials are Render environment variables.
Firebase stores the last answered message ID and publication timestamp, the
single latest question/answer card, and a temporary processing lease while a
response is in flight. Older chat content is not retained. Transactional claims
prevent overlapping workers from answering the same message; abandoned leases
can be reclaimed after five minutes.

## Endpoints

- `GET /health` — process status, with no secrets
- `POST /tick` — starts a job when authorized with
  `Authorization: Bearer $CRON_SHARED_SECRET`
- `POST /tick?delay=30` — schedules the same job 30 seconds later, used with a
  second whole-minute cron schedule to produce a 90-second cadence

## Environment

See `render.yaml`. `YOUTUBE_STREAM_ID` must identify a reusable stream and
`YOUTUBE_BROADCAST_ID` must identify a non-completed broadcast bound to it.
`FIREBASE_SERVICE_ACCOUNT` is the service-account JSON string.
