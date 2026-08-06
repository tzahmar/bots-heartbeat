# bots-heartbeat

An external heartbeat for two Telegram bots running on a Hostinger shared-hosting
account. This repository contains **no application code and no credentials** —
only the scheduled workflow.

## Why it exists

The bots are supervised on the server by a small always-on script (`keepalive.sh`)
that restarts them within 60 seconds if they crash. That handles crashes, but it
cannot restart *itself*: a reboot or a hosting-side process reaper kills the
supervisor along with everything else.

That gap is normally closed by cron. This hosting account has no working cron —
there is no `crontab`/`crond` binary, `/var/spool/cron` is empty, and a monitoring
script placed there weeks earlier had never executed once. So the heartbeat has to
come from outside the machine.

Every 10 minutes this workflow opens an SSH session and relaunches the supervisor.
Both the supervisor and the watchdog it drives are idempotent — when everything is
already healthy the run changes nothing — which is what makes it safe to repeat
indefinitely.

## Why public

GitHub Actions minutes are unmetered for public repositories, so a 10-minute
heartbeat costs nothing. A private repository at this frequency would consume
roughly twice the entire monthly free allowance.

Nothing sensitive is exposed: the SSH key, host, and Telegram credentials live in
encrypted repository secrets, and the key is dedicated to this workflow alone.

## Required secrets

| Secret | Purpose |
|---|---|
| `SSH_KEY` | Private key (ed25519), used only by this workflow |
| `SSH_HOST` / `SSH_PORT` / `SSH_USER` | Connection details |
| `SSH_KNOWN_HOSTS` | Pinned host key, so the job cannot be redirected to another host |
| `TG_TOKEN` / `TG_CHAT` | Telegram alert on failure |

## Failure behaviour

Ordinary crash-and-restart is reported by the server-side watchdog. This workflow
only alerts when the server is unreachable or a bot will not come up — the cases
the server cannot resolve on its own.
