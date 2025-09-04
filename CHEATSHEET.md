![Banner](repository-assets/banner.png)

# Installation

## Sign up for an account

Create a free account: https://ngrok.com/signup

## Download ngrok

- **MacOS**: `brew install ngrok`
- **Linux**: `snap install ngrok`
- **Windows**: `winget install ngrok -s msstore`

For more agents and SDKs: https://ngrok.com/downloads/

Get your authtoken: https://dashboard.ngrok.com/

## Add your authtoken

```bash
ngrok config add-authtoken <token>
```

## Start an endpoint

```bash
ngrok http 80
```

# Expose different kinds of servers

- API Service (Port 8080): `ngrok http 8080`
- Web App (Port 3000): `ngrok http 3000`
- SSH Server (Port 22): `ngrok tcp 22`
- Postgres Server: `ngrok tcp 5432`
- Any service on a different machine: `ngrok http http://192.168.1.50:8080`

# Troubleshoot

```bash
ngrok diagnose
```

# Add authentication

- Basic Authentication: `ngrok http 8080 --basic-auth "user:pass"`
- OAuth Authentication: `ngrok http 8080 --oauth github`

List of Providers: https://ngrok.com/docs/traffic-policy/actions/oauth/#supported-providers

# Add webhook verification

```yaml
on_http_request:
  - actions:
      - type: verify-webhook
        config:
          provider: gitlab
          secret: secret!
      - type: custom-response
        config:
          status_code: 200
          headers:
            content-type: text/plain
          body: GitLab webhook verified
```

List of Providers: https://ngrok.com/docs/traffic-policy/actions/verify-webhook/#supported-providers

# Presenting Traffic Policy

ngrok's Traffic Policy is a configuration language that offers you the flexibility to filter, match, manage and orchestrate traffic to your endpoints.

With Traffic Policy you can:

- Validate incoming traffic
- Block malicious traffic
- Rewrite URLs
- Respond with custom content
- Forward traffic to your agents running across the globe
  
  & more.

List of Traffic Policy Actions: https://ngrok.com/docs/traffic-policy/actions/

## Create a Policy File

```bash
nano policy.yaml
```

## Example: Using the `verify-webhook` action

```yaml
# policy.yaml
on_http_request:
  - actions:
      - type: verify-webhook
        config:
          provider: $PROVIDER # Example: Slack
          secret: $PROVIDER_TOKEN
```

## Start a Public Endpoint

```bash
# Using a Traffic Policy file:
ngrok http 8080 --url https://baz.ngrok.dev --traffic-policy-file policy.yaml
```

```bash
# Using a Traffic Policy URL:
ngrok http 8080 --url https://baz.ngrok.dev policy --traffic-policy-url https://example.com/policy.yml
```

# Load Balance

```bash
# Between two different ports:
ngrok http 8080 --url https://api.example.com --pooling-enabled
ngrok http 8081 --url https://api.example.com --pooling-enabled
```

Read more about Endpoint Pooling: https://ngrok.com/docs/universal-gateway/endpoint-pooling/

# What *else* can I do with ngrok?
- Use ngrok Kubernetes Operator: https://ngrok.com/docs/k8s/
- Get started with Traffic Observability: https://ngrok.com/docs/obs/
- Look into Identity and Access Management: https://ngrok.com/docs/iam/
- Read ngrok's Security Best Practices: https://ngrok.com/docs/guides/security-dev-productivity/
