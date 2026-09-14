# Desktop + self-host rollout guide

This guide is for teams rolling out a Linux self-hosted Huly deployment plus
Huly Desktop on macOS and Linux workstations, while wiring the platform into
external systems such as **M3ta-0s**, **Qu3bii**, and **A.D.A.M**.

> [!IMPORTANT]
> Use [huly-selfhost](https://github.com/hcengineering/huly-selfhost) for the
> production self-hosted server deployment. This monorepo is the source tree
> for platform development and a reference for service expectations such as
> `FRONT_URL`, `ACCOUNTS_URL`, desktop update settings, and optional feature
> flags.

## 1. Inventory the boxes

Before rollout, record each target box and assign exactly one primary role:

- **Linux self-host server** — runs the Huly stack from `huly-selfhost`
- **macOS desktop client** — runs Huly Desktop and connects to the shared URL
- **Linux desktop client** — runs Huly Desktop and connects to the shared URL
- **Integration peer** — M3ta-0s, Qu3bii, or A.D.A.M acting as an identity,
  automation, or data endpoint

For each box, capture:

- host name
- operating system
- DNS name
- TLS certificate owner/source
- whether it needs autostart on boot
- whether it stores secrets locally
- which Huly workspace(s) it should access

## 2. Standardize the connection contract

All boxes should use the same canonical Huly entrypoints:

- **Front-end URL** (`FRONT_URL`) — the main web/Desktop URL
- **Accounts URL** (`ACCOUNTS_URL`) — the account/auth endpoint
- **Storage and mail settings** — match the server-side deployment
- **Desktop update source** (`DESKTOP_UPDATES_URL`) — defaults to
  `https://dist.huly.io`
- **Desktop update channel** (`DESKTOP_UPDATES_CHANNEL`) — keep one default
  channel policy per fleet

In this repository, the local dev stack shows how the platform expects these
settings to line up across services, and the desktop app persists the selected
server URL for each workstation.

## 3. Deploy the Linux self-hosted server

Use `huly-selfhost` as the operational source of truth for the Linux host.

During setup, make sure the server-side configuration is aligned with your
fleet contract:

- public `FRONT_URL`
- public `ACCOUNTS_URL`
- storage settings
- mail/OTP delivery
- backup and restore procedure
- optional disabled features via `DISABLED_FEATURES`

After deployment, verify that the self-hosted environment is reachable over the
same URLs that the desktop clients will use.

For migration and restore workflows, also see
[backup-restore.en.md](./backup-restore.en.md).

## 4. Roll out Huly Desktop to macOS and Linux

Published desktop update metadata is available at:

- macOS: `https://dist.huly.io/latest-mac.yml`
- Linux: `https://dist.huly.io/latest-linux.yml`

For each workstation:

1. Install the matching desktop build for the operating system.
2. Point the app to the canonical self-hosted `FRONT_URL`.
3. Confirm the workstation can authenticate against the shared
   `ACCOUNTS_URL`.
4. Apply the local desktop preferences you want standardized across the fleet,
   such as auto-launch or minimize-to-tray behavior.
5. Verify login, workspace access, notifications, downloads, and update checks.

Keep the desktop fleet on one update policy unless you intentionally split
channels for staged rollouts.

## 5. Wire M3ta-0s, Qu3bii, and A.D.A.M into Huly

Treat each external system as an explicit integration peer and record:

- role: identity provider, automation peer, or data/integration target
- direction: inbound to Huly, outbound from Huly, or bidirectional
- secrets and token ownership
- callback/webhook URLs
- workspace scope
- failure and retry expectations

Minimum rollout target:

- **M3ta-0s** — defined integration role and validated connection path
- **Qu3bii** — defined integration role and validated connection path
- **A.D.A.M** — defined integration role and validated connection path

Validate one end-to-end flow per peer before expanding scope.

## 6. Persist box-specific operating procedures

For the Linux self-host server, document:

- restart procedure
- upgrade procedure
- backup/restore owner
- secret rotation owner

For each macOS/Linux desktop workstation, document:

- install source
- server URL
- update channel
- launch-at-login policy
- local recovery steps

## 7. Verify the full mesh

The rollout is ready only when all of the following are true:

- the Linux self-hosted server is reachable from every client box
- macOS desktop clients can sign in and open the expected workspaces
- Linux desktop clients can sign in and open the expected workspaces
- M3ta-0s integrations succeed
- Qu3bii integrations succeed
- A.D.A.M integrations succeed
- backup/restore steps are documented and have been exercised at least once
