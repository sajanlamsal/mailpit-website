---
title: Link check
description: Link check will test all message links
section: usage
weight: 5
---

Link check scans your message (HTML and text) for up to 100 unique links, images, and linked stylesheets. It then makes an HTTP `HEAD` request to each link to test whether the link, image, or stylesheet exists.

{{< tip "warning" >}}
For security reasons, link check will (by default) **block** all HTTP requests to internal (non-public) networks, including loopback, private, unicast & multicast addresses. This is to prevent SSRF (Server-Side Request Forgery) attacks, which can be used to access internal services or resources that are not intended to be exposed. If you trust all traffic to your Mailpit instance, or use authentication to restrict access to your Mailpit instance, you can disable this using `--allow-internal-http-requests` (or the `MP_ALLOW_INTERNAL_HTTP_REQUESTS=true` environment setting).
{{< /tip >}}

{{< tip "warning" >}}
Link check will, by default, require valid HTTPS certificates for any HTTPS links. You can disable this using `--allow-untrusted-tls` (or the `MP_ALLOW_UNTRUSTED_TLS=true` environment setting).
{{< /tip >}}

{{< tip "warning" >}}
Link check will only scan up to 100 unique links per email, and rate limits requests to prevent abuse. [See docs](#rate-limiting) for more information.
{{< /tip >}}

## What are "301" and "302" links?

These are links that redirect you to another URL. For example, newsletters often use redirect links to track user clicks.

By default, Link check will not follow these links; however, you can turn this on via the settings, and Link check will "follow" those redirects.

## Rate limiting

The link checker uses a **per-domain rate limiter** to prevent hammering external servers during a check.

**Domain grouping** - Links are grouped by registered domain (eTLD+1), so `images.example.com` and `click.example.com` share the same limiter as `example.com`. This closes the loophole of bypassing limits via subdomains.

**Token bucket** - Each domain gets a token bucket that starts full at 100 tokens and refills at 1 token per second. Checking a link consumes one token. A fresh check of an email with up to 100 links to the same domain completes immediately; beyond that, requests are paced to one per second.

**Concurrency cap** - Independently of the token bucket, at most 2 requests to the same domain run at the same time. This limits connection load even when tokens are plentiful.

**Result cache** - Each checked URL is cached for 60 seconds. Re-checking the same email within that window skips the network request entirely and returns the cached status, so the rate limiter is not drained twice.

**Disabling** - Passing `--disable-link-check-rate-limit` (or setting `MP_DISABLE_LINK_CHECK_RATE_LIMIT=true`) skips the token bucket, the concurrency cap, and the result cache entirely. Use with caution.

## Some links return an error but work in my browser?

This may be due to various reasons, for instance:

- The Mailpit server cannot resolve (DNS) the hostname of the URL.
- Mailpit is not allowed to access the URL.
- The webserver is blocking requests that don't come from authenticated web browsers.
- The webserver does not allow HTTP HEAD requests.
