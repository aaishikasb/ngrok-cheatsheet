Getting started with ngrok is almost suspiciously easy. Which is why the question we hear most often isn’t "*where do I start?*" but rather “*what else can I do with ngrok?*” For over a decade, we’ve been serving millions of developers and, through them, millions of users.

Sure, our [documentation](https://ngrok.com/docs/) has all the answers, every CLI flag, every Traffic Policy configuration neatly laid out, but what about the developers who are always on the lookout for a **TL;DR**? What if you too are not looking for a full solution or a specific use case for your next project, but just want to know… what else *you* can do with ngrok?

Presenting ngrok's new cheatsheet that walks you through some of our most interesting offerings, designed to scratch that itch of not just serving, but also securing endpoints in as little as two steps. Read on, or download the [PDF format](), or a crisp [two-pager printable]().

# [Installation](https://ngrok.com/downloads/)

- Sign up for a free account: https://ngrok.com/signup
- Keep your authtoken handy: https://dashboard.ngrok.com/get-started/your-authtoken

## [macOS](https://ngrok.com/downloads/mac-os)

```bash
# Install via Homebrew
brew install ngrok

# Add your authtoken
ngrok config add-authtoken <token>

# Start an endpoint
ngrok http 80
```

## [Linux](https://ngrok.com/downloads/linux)

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

## [Windows](https://ngrok.com/downloads/windows)

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

## [Kubernetes](https://ngrok.com/downloads/kubernetes)

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

## [Docker](https://ngrok.com/downloads/docker)

```bash
# Install via Docker
docker pull ngrok/ngrok

# Run ngrok via Docker
docker run --net=host -it -e NGROK_AUTHTOKEN=xyz ngrok/ngrok:latest http 80
```

## [SDKs](https://ngrok.com/downloads/)

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

# [Expose different kinds of servers](https://ngrok.com/docs/agent/#example-usage)

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

# [Troubleshoot](https://ngrok.com/docs/agent/#troubleshooting-connectivity)

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

# [Add Authentication with Traffic Policy](https://ngrok.com/docs/traffic-policy/actions/oauth/)

## [Create a Traffic Policy file `policy.yaml`](https://ngrok.com/docs/traffic-policy/getting-started/agent-endpoints/cli/)

```bash
nano policy.yaml
```

## [Add the OAuth Action with Google](https://ngrok.com/docs/integrations/google/oauth/)

List of Providers: https://ngrok.com/docs/traffic-policy/actions/oauth/#supported-providers

```yaml
# policy.yaml
on_http_request:
  - actions:
      - type: oauth
        config:
          provider: google # OAuth available with Amazon, Facebook, GitHub, GitLab, Google, LinkedIn, Microsoft, Twitch
```

## [Run your endpoint with the Traffic Policy file](https://ngrok.com/docs/traffic-policy/getting-started/agent-endpoints/cli/#2-apply-your-traffic-policy)

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

# [Verify your webhooks](https://ngrok.com/docs/traffic-policy/actions/verify-webhook/)

## [Add the `verify-webhook` action for Slack](https://ngrok.com/docs/integrations/slack/webhooks/)

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

## [Replace `provider` for any supported provider](https://ngrok.com/docs/traffic-policy/actions/verify-webhook/)
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

# [Do even more with internal endpoints](https://ngrok.com/docs/universal-gateway/internal-endpoints/)

## [Create a Cloud Endpoint](https://ngrok.com/docs/traffic-policy/actions/forward-internal/)

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

## [Add path-based routing](https://ngrok.com/docs/universal-gateway/cloud-endpoints/routing-and-policy-decentralization/)

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

## [Route traffic by anything](https://ngrok.com/docs/traffic-policy/actions/forward-internal/#examples)

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

## [Multiplex to Internal Services from a Single Domain](https://ngrok.com/docs/universal-gateway/examples/multiplex/)

```yaml
# policy.yaml
on_http_request:
  - actions:
      - type: forward-internal
        config:
          url: https://${req.host.split(".$NGROK_DOMAIN")[0]}.internal
```

## [Add rate limiting](https://ngrok.com/docs/traffic-policy/actions/rate-limit/)

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

## [Block search and AI bots](https://ngrok.com/docs/traffic-policy/examples/block-unwanted-requests/#how-do-i-deny-traffic-from-bots-and-crawlers-with-a-robotstxt)

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

## [Add headers](https://ngrok.com/docs/traffic-policy/actions/add-headers/)

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

## [Restrict access by IPs](https://ngrok.com/docs/traffic-policy/actions/restrict-ips/)

```bash
# Allow only 203.0.113.0/24; deny others
ngrok http 8080 --cidr-allow 203.0.113.0/24

# Or explicitly deny CIDRs
ngrok http 8080 --cidr-deny 0.0.0.0/0
```

## [Block all the potentially bad things](https://ngrok.com/docs/traffic-policy/actions/owasp-crs-response/)

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

# What *else* can I do with ngrok?
- Use ngrok Kubernetes Operator: https://ngrok.com/docs/k8s/
- Get started with Traffic Observability: https://ngrok.com/docs/obs/
- Look into Identity and Access Management: https://ngrok.com/docs/iam/
- Read ngrok's Security Best Practices: https://ngrok.com/docs/guides/security-dev-productivity/
- Check out Gateway Examples Gallery: https://ngrok.com/docs/universal-gateway/examples/

# Ending Notes

The first iteration of the ngrok cheatsheet was created by [Keith Casey](https://ngrok.com/blog-author/danger), who served on the Product/GTM Team at ngrok. You can read the now-deprecated version [here]().