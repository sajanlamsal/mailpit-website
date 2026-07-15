---
title: Separate mailboxes
description: View captured mail grouped by the authenticated SMTP username, as a separate switchable mailbox per user
section: usage
keywords: [mailbox, mailboxes, username, multi-user, separate, inbox, smtp auth, switch]
weight: 3
---

Mailpit stores all received mail in a single database, but when [SMTP authentication](../../configuration/smtp/#adding-smtp-authentication) is enabled it records the authenticated username on every message. Each distinct username then appears as its own **mailbox** in the web UI, allowing several services to share one Mailpit instance while viewing each one's mail separately.

This requires no additional configuration beyond enabling SMTP authentication — the mailbox switcher appears automatically once authenticated mail has been received.

## Setup

### 1. Enable SMTP authentication with a username per service

Configure [SMTP authentication](../../configuration/smtp/#adding-smtp-authentication) using a [password file](../../configuration/passwords/), giving each service its own username, for example:

```text
service-a:$2y$...bcrypt-hash...
service-b:$2y$...bcrypt-hash...
```

Start Mailpit with the credentials:

```shell
mailpit --smtp-auth-file ./smtp-auth.txt
```

{{< tip "warning" >}}
Enabling SMTP authentication means **all** senders must authenticate — unauthenticated mail is rejected. For testing over an unencrypted connection you can add the `--smtp-auth-allow-insecure` flag. See [Adding SMTP authentication](../../configuration/smtp/#adding-smtp-authentication) for the full details.
{{< /tip >}}

### 2. Point each service at its own username

Configure each application to authenticate with its own credentials, for example:

```text
MAIL_HOST=<mailpit-host>
MAIL_PORT=1025
MAIL_USERNAME=service-a
MAIL_PASSWORD=<service-a password>
```

### 3. Switch between mailboxes

Once each service has sent mail, use the **Mailbox** dropdown in the web UI sidebar to switch between `All mail` and each username. Selecting a mailbox shows only the messages sent by that authenticated user.

The selected mailbox acts as a persistent scope: [tags](../tagging/) and [searches](../search-filters/) apply *within* the current mailbox rather than across all mail. You can also filter directly using the [`username:` search filter](../search-filters/) (e.g. `username:service-a`), or open a mailbox by URL at `/mailbox/service-a`.

## Trusted vs. untrusted usernames

When authentication is configured with a [password file](../../configuration/passwords/), the supplied password is verified against the stored hash, so the mailbox label reflects a sender that proved it owns those credentials.

{{< option flag="smtp-auth-accept-any" env="MP_SMTP_AUTH_ACCEPT_ANY" default="false" >}}
Accept any SMTP username and password (or none). Any username is accepted **without a password check**, so mail is filed under whatever username the sender claims, but the mailbox label cannot be trusted. Mail sent with no authentication at all has no username and only appears under `All mail`.
{{< /option >}}

## Relation to username tagging

This feature is independent of [username auto-tagging](../tagging/#username-auto-tagging) (`--tags-username`). The mailbox switcher reads the authenticated username stored on each message directly, so you do **not** need to enable `--tags-username` to use it. Enable username tagging as well only if you also want each message labelled with a clickable username tag.
