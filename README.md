![Banner](repository-assets/banner.png)

- [Getting Started](#getting-started)
- [Installation](#installation)
  - [macOS](#macos)
  - [Linux](#linux)
  - [Windows](#windows)
  - [Kubernetes](#kubernetes)
  - [Docker](#docker)
  - [SDKs](#sdks)
- [Exposing different kinds of servers](#exposing-different-kinds-of-servers)
  - [API service](#api-service)
  - [Web app](#web-app)
  - [SSH server](#ssh-server)
  - [Postgres server](#postgres-server)
  - [Any service or server on a different machine](#any-service-or-server-on-a-different-machine)
- [Troubleshooting](#troubleshooting)
- [Add Authentication with Traffic Policy](#add-authentication-with-traffic-policy)
  - [Create a Traffic Policy file `policy.yaml`](#create-a-traffic-policy-file-policyyaml)
  - [Add the OAuth Action with Google](#add-the-oauth-action-with-google)
  - [Run your endpoint with the Traffic Policy file](#run-your-endpoint-with-the-traffic-policy-file)
  - [Restrict OAuth to specific emails](#restrict-oauth-to-specific-emails)
  - [Restrict OAuth to specific domains](#restrict-oauth-to-specific-domains)
- [Verify your webhooks](#verify-your-webhooks)
  - [Add the `verify-webhook` action for Slack](#add-the-verify-webhook-action-for-slack)
  - [CLI Alternative](#cli-alternative)
  - [Replace `provider` for any supported provider](#replace-provider-for-any-supported-provider)
- [Do even more with internal endpoints](#do-even-more-with-internal-endpoints)
  - [Create a Cloud Endpoint](#create-a-cloud-endpoint)
  - [Example Traffic Policy File](#example-traffic-policy-file)
  - [Start Public Endpoint with `forward-internal` action](#start-public-endpoint-with-forward-internal-action)
- [Manage traffic in other ways](#manage-traffic-in-other-ways)
  - [Add path-based routing](#add-path-based-routing)
  - [Route traffic by anything](#route-traffic-by-anything)
  - [Multiplex to Internal Services from a Single Domain](#multiplex-to-internal-services-from-a-single-domain)
  - [Add rate limiting](#add-rate-limiting)
  - [Block search and AI bots](#block-search-and-ai-bots)
  - [Add headers](#add-headers)
    - [Via CLI](#via-cli)
    - [Via Traffic Policy](#via-traffic-policy)
  - [Restrict access by IPs](#restrict-access-by-ips)
  - [Block all the potentially bad things](#block-all-the-potentially-bad-things)
- [CLI Flags](#cli-flags)
  - [`url`](#url)
  - [`traffic-policy-file`](#traffic-policy-file)
  - [`traffic-policy-url`](#traffic-policy-url)
  - [`pooling-enabled`](#pooling-enabled)

# Getting Started

- Sign up for a free account: https://ngrok.com/signup
- Grab your authtoken: https://dashboard.ngrok.com/get-started/your-authtoken

# Installation

## macOS

```bash
# Install via Homebrew
brew install ngrok

# Add your authtoken
ngrok config add-authtoken <token>

# Start an endpoint
ngrok http 80
```

## Linux

```bash
# Install via Apt
curl -sSL https://ngrok-agent.s3.amazonaws.com/ngrok.asc \
  | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null \
  && echo "deb https://ngrok-agent.s3.amazonaws.com bookworm main" \
  | sudo tee /etc/apt/sources.list.d/ngrok.list \
  && sudo apt update \
  && sudo apt install ngrok
# OR
# Install via Snap
snap install ngrok

# Add your authtoken
ngrok config add-authtoken <token>

# Start an endpoint
ngrok http 80
```

## Windows

```bash
# Install via WinGet
winget install ngrok -s msstore
# OR
# Install via Scoop
scoop install ngrok

# Add your authtoken
ngrok config add-authtoken <token>

# Start an endpoint
ngrok http 80
```

## Kubernetes

```bash
# Add ngrok Kubernetes Operator to Helm
helm repo add ngrok https://charts.ngrok.com

# Add ngrok API key and authtoken
export NGROK_AUTHTOKEN=YOUR_NGROK_AUTHTOKEN
export NGROK_API_KEY=YOUR_NGROK_API_KEY

helm install ngrok-operator ngrok/ngrok-operator \
  --namespace ngrok-operator \
  --create-namespace \
  --set credentials.apiKey=$NGROK_API_KEY \
  --set credentials.authtoken=$NGROK_AUTHTOKEN
```

## Docker

```bash
# Install via Docker
docker pull ngrok/ngrok

# Run ngrok via Docker
docker run --net=host -it -e NGROK_AUTHTOKEN=xyz ngrok/ngrok:latest http 80
```

## SDKs

```bash
# Node.js: https://ngrok.com/downloads/node-js
npm install @ngrok/ngrok

# Go: https://ngrok.com/downloads/go
go get golang.ngrok.com/ngrok/v2

# Python: https://ngrok.com/downloads/python
python3 -m pip install ngrok

# Rust: https://ngrok.com/downloads/rust
# Install ngrok-rust package and the required dependencies
cargo add ngrok -F axum && cargo add axum && cargo add tokio -F rt-multi-thread -F macros
```

# Exposing different kinds of servers

## API service

```bash
# Example: API service on localhost:8080
ngrok http 8080
```

## Web app

```bash
# Example: On localhost:3000
ngrok http 3000
```

## SSH server

```bash
# Example: On Port 22
ngrok tcp 22
```

## Postgres server

```bash
ngrok tcp 5432
```

## Any service or server on a different machine

```bash
ngrok http http://192.168.1.50:8080
```

# Troubleshooting

```bash
ngrok diagnose

# To test IPv6 connectivity
ngrok diagnose --ipv6 true

# To test connectivity between the ngrok agent and all ngrok points of presence
ngrok diagnose --region all

# For a verbose report
ngrok diagnose -w out.txt #OR
ngrok diagnose --write-report out.txt
```

# Add Authentication with Traffic Policy

Read more: https://ngrok.com/docs/traffic-policy/actions/oauth/

## Create a Traffic Policy file `policy.yaml`

```bash
nano policy.yaml
```

## Add the OAuth Action with Google

```yaml
# policy.yaml
on_http_request:
  - actions:
      - type: oauth
        config:
          provider: google # OAuth available with Amazon, Facebook, GitHub, GitLab, Google, LinkedIn, Microsoft, Twitch
```

## Run your endpoint with the Traffic Policy file

```bash
ngrok http 8080 --traffic-policy-file=policy.yaml
```

## Restrict OAuth to specific emails

```yaml
# policy.yaml
on_http_request:
  - actions:
      - type: oauth
        config:
          provider: google
  - expressions:
      - "!(actions.ngrok.oauth.identity.email in ['alice@example.com','bob@example.com'])"
    actions:
      - type: deny
```

## Restrict OAuth to specific domains

```yaml
# policy.yaml
on_http_request:
  - actions:
      - type: oauth
        config:
          provider: google
  - expressions:
      - "!(actions.ngrok.oauth.identity.email.endsWith('@example.com'))"
    actions:
      - type: deny
```

# Verify your webhooks

## Add the `verify-webhook` action for Slack

```yaml
# policy.yaml
on_http_request:
  - actions:
      - type: verify-webhook
        config:
          provider: slack
          secret: $SLACK_TOKEN
```

## CLI Alternative

```bash
ngrok http 3000 \
  --verify-webhook=slack \
  --verify-webhook-secret=$SLACK_TOKEN
```

## Replace `provider` for any supported provider
List of Supported Providers: https://ngrok.com/docs/traffic-policy/actions/verify-webhook/

```yaml
# policy.yaml
on_http_request:
  - actions:
      - type: verify-webhook
        config:
          provider: $PROVIDER # Example: GitHub
          secret: $PROVIDER_TOKEN
```

# Do even more with internal endpoints

Read more: https://ngrok.com/docs/traffic-policy/actions/forward-internal/

## Create a Cloud Endpoint

```bash
# Create a private, internal agent endpoint only reachable via forward-internal
ngrok http 8080 --binding=internal --url https://api.internal
```

## Example Traffic Policy File

```yaml
# policy.yaml
# Forward to an internal endpoint from a public endpoint
on_http_request:
  - actions:
      - type: forward-internal
        config:
          url: https://api.internal
```

## Start Public Endpoint with `forward-internal` action

```bash
ngrok http 8080 --url forward-internal-example.ngrok.app --traffic-policy-file policy.yml
```

# Manage traffic in other ways

## Add path-based routing

```yaml
# policy.yaml
# Route /api/* to api.internal, /app/* to app.internal
on_http_request:
  - expressions:
      - "req.path.startsWith('/api/')"
    actions:
      - type: forward-internal
        config:
          url: https://api.internal
  - expressions:
      - "req.path.startsWith('/app/')"
    actions:
      - type: forward-internal
        config:
          url: https://app.internal
```

## Route traffic by anything

```yaml
# policy.yaml
# Host-based and header-based dynamic forwarding to internal endpoints via forward-internal action
on_http_request:
  - expressions:
      - "req.host == 'api.example.com'"
    actions:
      - type: forward-internal
        config: { url: https://api.internal }
  - expressions:
      - "getReqHeader('X-Tenant') != ''"
    actions:
      - type: forward-internal
        config: { url: https://tenant.internal }
```

## Multiplex to Internal Services from a Single Domain

```yaml
# policy.yaml
on_http_request:
  - actions:
      - type: forward-internal
        config:
          url: https://${req.host.split(".$NGROK_DOMAIN")[0]}.internal
```

## Add rate limiting

```yaml
# policy.yaml
# 10 requests per 60s window per client IP -> 429 on limit
on_http_request:
  - actions:
      - type: rate-limit
        config:
          name: per-ip-60s
          algorithm: sliding_window
          capacity: 10
          rate: "60s"
          bucket_key:
            - conn.client_ip
```

## Block search and AI bots

```yaml
# policy.yaml
# Send a robots.txt denying crawlers
on_http_request:
  - expressions:
      - "req.path == '/robots.txt'"
    actions:
      - type: custom-response
        config:
          status_code: 200
          headers:
            Content-Type: "text/plain"
          body: |
            User-agent: *
            Disallow: /
```

```yaml
# policy.yaml
# Deny common bot/AI user agents
on_http_request:
  - expressions:
      - "req.user_agent.raw.matches('(?i)(gptbot|chatgpt-user|ccbot|bingbot|googlebot)')"
    actions:
      - type: deny
```

```yaml
# policy.yaml
# Also add X-Robots-Tag to all responses
on_http_response:
  - actions:
      - type: add-headers
        config:
          headers:
            X-Robots-Tag: "noindex, nofollow, noai, noimageai"
```

## Add headers

### Via CLI

```bash
# Common security headers
ngrok http 8080 \
  --request-header-add "X-Frame-Options: DENY" \
  --response-header-add "Referrer-Policy: no-referrer"
```

### Via Traffic Policy

```yaml
# policy.yaml
# Add headers on request/response
on_http_request:
  - actions:
      - type: add-headers
        config:
          headers:
            X-Frame-Options: "DENY"
on_http_response:
  - actions:
      - type: add-headers
        config:
          headers:
            Referrer-Policy: "no-referrer"
```

## Restrict access by IPs

```bash
# Allow only 203.0.113.0/24; deny others
ngrok http 8080 --cidr-allow 203.0.113.0/24

# Or explicitly deny CIDRs
ngrok http 8080 --cidr-deny 0.0.0.0/0
```

## Block all the potentially bad things

```yaml
# policy.yaml
# Apply OWASP Core Rule Set on requests/responses
on_http_request:
  - actions:
      - type: owasp-crs-request
on_http_response:
  - actions:
      - type: owasp-crs-response
```

# CLI Flags

## `url`

```bash
# Choose a URL instead of random assignment
ngrok http 8080 --url https://baz.ngrok.dev
```

## `traffic-policy-file`

```bash
# Manipulate traffic to your endpoint with a traffic policy file
ngrok http 8080 --url https://baz.ngrok.dev --traffic-policy-file policy.yaml
```

## `traffic-policy-url`

```bash
# Manipulate traffic to your endpoint with a traffic policy URL
ngrok http 8080 --url https://baz.ngrok.dev policy --traffic-policy-url https://example.com/policy.yml
```

## `pooling-enabled`

```bash
# Load Balance (different ports)
ngrok http 8080 --url https://api.example.com --pooling-enabled
ngrok http 8081 --url https://api.example.com --pooling-enabled
```