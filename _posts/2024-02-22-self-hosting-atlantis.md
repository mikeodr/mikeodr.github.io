---
layout: post
title: Self Hosting Atlantis
date: 2024-02-22 21:24 -0500
categories: ["Home lab", "IAC"]
tags: ["atlantis", "self hosted", "IAC", "terraform", "github", "cloudflare", "tunnel"]
---

I've been interacting with [Atlantis](https://www.runatlantis.io/){:target="_blank"}
at work. But I am not responsible for its deployment or configuration,
it was a bit of a magic black box.

Considering I use some terraform for managing my personal Cloudflare DNS settings,
I figured I may as well try and get Atlantis setup for my own use!

> Not every step will be documented here, this is not intended to be a
> full walkthrough.  
> I will attempt to link the relevant documentation I used.
{: .prompt-info }

## The problem

So there were a few requirements I had to meet before setting this into the wild.

1. I must self host the Atlantis service
2. GitHub is my go to repo host, it must be able to [reach Atlantis](https://www.runatlantis.io/docs/deployment.html#routing)
3. No exposing Atlantis completely [to the entire internet](https://www.runatlantis.io/docs/security.html)

## Attempt one

I attempted to first try using [Tailscale Funnel](https://tailscale.com/kb/1223/funnel)
and mapping atlantis to Tailscale in docker from [this](https://tailscale.com/blog/docker-tailscale-guide)
[example](https://github.com/tailscale-dev/docker-guide-code-examples/blob/main/04-ts-mealie/compose.yaml)

This worked great, I self hosted Atlantis and can access it externally! ✅

Unfortunately as much as Tailscale is _awesome_ the [ACLs](https://tailscale.com/kb/1018/acls)
do not support controlled access to services hosted with funnel. ❌

## Attempt Two (Success)

Revisiting the toolbox of poking safe holes through my firewall, and since
I am already using Cloudflare for my DNS, I looked at their [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
offering.

After [creating a tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/)
I was quickly able to access to Atlantis over the open internet. ✅

Thankfully unlike Tailscale, Cloudflare does have a [WAF](https://developers.cloudflare.com/waf/custom-rules/)
that works alongside the tunnel to control access. ✅

Grabbing the webhook CIDR blocks from [GitHub Meta API](https://api.github.com/meta)
I quickly put together a WAF rule to block access:

```plaintext
(not ip.src in {192.30.252.0/22 185.199.108.0/22 140.82.112.0/20 143.55.64.0/20} and http.host eq "atlantis.domain.ca")
```

## Putting it together

Following
the [Atlantis GitHub app](https://www.runatlantis.io/docs/access-credentials.html#github-app)
docs I was able to meet all my personal requirements.

I ended up with the following `docker-compose.yml` that binds
the Atlantis containers networking to the `cloudflared` container.

```yaml
---
version: "2.2"
services:
  atlantis:
    image: ghcr.io/runatlantis/atlantis:latest
    restart: unless-stopped
    volumes:
      - ${PWD}/config:/config
      - ${PWD}/data:/data
    environment:
      ATLANTIS_CONFIG: /config/atlantis.yml
      ATLANTIS_DATA_DIR: /data
      AWS_ACCESS_KEY_ID: <Access Key ID>
      AWS_SECRET_ACCESS_KEY: <Secret Key>
      AWS_REGION: us-west-2
      CLOUDFLARE_API_TOKEN: <Cloudflare Access Token>
    networks:
      - atlantis

  tunnel:
    image: cloudflare/cloudflared:latest
    restart: unless-stopped
    command: tunnel run
    environment:
      - TUNNEL_TOKEN=<Your CF Tunnel Token>
    networks:
      - atlantis

volumes:
  tailscale-data-atlantis:

networks:
  atlantis:
```

Most importantly, I can now safely access the Atlantis UI locally, and
the webhooks from GitHub work great!
Now I can `atlantis plan` and `atlantis apply` on my terraform PRs for my
personal cloud infrastructure! ✅

![Atlantis](assets/img/atlantis-jobs.png)
_The Atlantis UI is safely accessible_

![GitHub](assets/img/github-atlantis.png)
_GitHub interactions are automated and work great!_

### Extra notes

My terraform code is private, so I will not be sharing that.
Additionally Atlantis specifically calls out [not to use public repos](https://www.runatlantis.io/docs/security.html#don-t-use-on-public-repos).
Setting up terraform is an exercise for the reader.
