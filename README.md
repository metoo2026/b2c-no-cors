# Multi-Account Backblaze B2 Cloudflare Worker (Without CORS)

A Cloudflare Worker for accessing multiple Backblaze B2 accounts through a single endpoint using the S3-compatible API. Account credentials are stored securely in Cloudflare Worker secrets, while cache settings can be configured using environment variables.

## Features

- **Multiple B2 accounts:** Configure as many Backblaze B2 accounts as required.
- **S3-compatible access:** Connect to Backblaze B2 using S3-compatible endpoints and application keys.
- **Dynamic account configuration:** Store account configurations in a single `B2_ACCOUNTS` secret.
- **Cloudflare edge caching:** Configure cache duration using `CACHE_TTL`.
- **Secure credential storage:** Keep account credentials out of the source code and Git repository.
- **Google Colab deployment:** Manage and deploy the Worker from Google Colab using Wrangler.
- **Configurable cache duration:** Change the cache TTL without modifying `index.js`.

## Requirements

- A Cloudflare account with Workers access.
- One or more Backblaze B2 accounts.
- An S3-compatible application key for each B2 account.
- Google Colab or a local environment with Node.js and Wrangler.
- A Cloudflare API token with the permissions required to deploy Workers and manage Worker secrets.

## Project Structure

```text
b2-multi-account-worker/
├── index.js
├── wrangler.toml
├── .gitignore
└── README.md
```

## Configuration

### 1. Configure `wrangler.toml`

Example configuration:

```toml
name = "b2-multi-account-worker"
main = "index.js"
compatibility_date = "2026-10-15"

[vars]
CACHE_TTL = "86400"

[secrets]
required = ["B2_ACCOUNTS"]
```

`CACHE_TTL` is measured in seconds.

| Value | Cache duration |
|---:|---|
| `3600` | 1 hour |
| `21600` | 6 hours |
| `43200` | 12 hours |
| `86400` | 24 hours |
| `604800` | 7 days |
| `0` | Disable caching in the Worker logic |

If `CACHE_TTL` is not configured, `index.js` uses `86400` seconds as its default.

**Important:** The `CACHE_TTL` value in `wrangler.toml` should match the value you intend to use when deploying with Wrangler.

### 2. Configure B2 Accounts

The Worker reads account configuration from the `B2_ACCOUNTS` secret.

Example structure (use placeholder credentials only in documentation):

```json
{
  "account01": {
    "endpoint": "s3.us-west-004.backblazeb2.com",
    "region": "us-west-004",
    "keyId": "KEY_ID_1",
    "applicationKey": "APP_KEY_1",
    "label": "Account 01"
  },
  "account02": {
    "endpoint": "s3.us-west-004.backblazeb2.com",
    "region": "us-west-004",
    "keyId": "KEY_ID_2",
    "applicationKey": "APP_KEY_2",
    "label": "Account 02"
  },
  "account03": {
    "endpoint": "s3.us-west-004.backblazeb2.com",
    "region": "us-west-004",
    "keyId": "KEY_ID_3",
    "applicationKey": "APP_KEY_3",
    "label": "Account 03"
  },
  "account04": {
    "endpoint": "s3.us-west-004.backblazeb2.com",
    "region": "us-west-004",
    "keyId": "KEY_ID_4",
    "applicationKey": "APP_KEY_4",
    "label": "Account 04"
  },
  "account05": {
    "endpoint": "s3.us-west-004.backblazeb2.com",
    "region": "us-west-004",
    "keyId": "KEY_ID_5",
    "applicationKey": "APP_KEY_5",
    "label": "Account 05"
  }
}
```

Replace the example endpoints and credentials with the actual values for each account. Use the endpoint and region specified for your Backblaze B2 S3-compatible configuration.

You can add more accounts by adding more entries to the JSON object. Each account needs a unique account name.

**Security:** Do not commit real application keys, account credentials, or the contents of `B2_ACCOUNTS` to GitHub.

### 3. Set the `B2_ACCOUNTS` Secret

Using Wrangler:

```bash
npx wrangler secret put B2_ACCOUNTS
```

Paste the complete JSON configuration when prompted.

When updating this secret, provide the complete configuration for all accounts you want to retain. Updating the secret replaces its previous value.

### 4. Set the Cloudflare API Token

For Google Colab, store your Cloudflare API token in a protected environment variable or enter it securely when prompted.

Example:

```python
import os
import getpass

os.environ["CLOUDFLARE_API_TOKEN"] = getpass.getpass(
    "Enter Cloudflare API Token: "
)
```

Avoid printing the token or saving it in a notebook that will be shared publicly.

## Deployment

### Deploy with Wrangler

Install Wrangler if necessary:

```bash
npm install -g wrangler
```

Authenticate using your Cloudflare API token and deploy:

```bash
wrangler deploy
```

Make sure the required `B2_ACCOUNTS` secret is configured for the Worker before deploying.

### Deploy from Google Colab

1. Clone your repository.
2. Install or configure Wrangler.
3. Set the Cloudflare API token securely.
4. Configure `B2_ACCOUNTS` as a Worker secret.
5. Set `CACHE_TTL` in `wrangler.toml`.
6. Run `wrangler deploy`.

Example:

```python
!git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
%cd YOUR_REPOSITORY
!npm install -g wrangler
!wrangler deploy
```

Replace the repository URL with your own GitHub repository. If you have already cloned the repository, navigate to its directory instead of cloning it again.

## Changing the Cache TTL

The Worker reads the configured value from:

```javascript
env.CACHE_TTL
```

The fallback in `index.js` is:

```javascript
env.CACHE_TTL || "86400"
```

For example, setting `CACHE_TTL` to `21600` makes the configured cache TTL six hours.

You can change this value in either of these ways:

- **Cloudflare Dashboard:** Open Workers & Pages, select your Worker, then open Settings → Variables and Secrets. Add or update `CACHE_TTL` under Variables.
- **Wrangler configuration:** Edit `CACHE_TTL` in the `[vars]` section of `wrangler.toml` and deploy again.

The JavaScript source does not need to change when only the TTL value changes.

## URL Structure

The Worker supports account-based routes. The exact URL structure depends on the routing implemented in `index.js`.

Typical route patterns are:

```text
https://YOUR_WORKER_DOMAIN/account01/
https://YOUR_WORKER_DOMAIN/account01/BUCKET_NAME/
https://YOUR_WORKER_DOMAIN/account01/BUCKET_NAME/path/to/file.mp4
```

Where supported by the deployed code:

- `/account01/` accesses the bucket-listing route for that account.
- `/account01/BUCKET_NAME/` accesses the object-listing route for that bucket.
- `/account01/BUCKET_NAME/path/to/file.mp4` accesses an object through the Worker.

Replace `YOUR_WORKER_DOMAIN` and the example account, bucket, and object names with your actual values.

## Caching Behaviour

The Worker uses Cloudflare's Cache API for eligible object GET requests when caching is enabled.

- Successful eligible object responses can be cached.
- Cache duration is controlled by `CACHE_TTL`.
- Conditional requests and range requests bypass the Worker cache in the current implementation.
- Bucket listings are not cached by this Worker logic.
- Setting `CACHE_TTL` to `0` disables caching through the TTL check.

Cache behaviour may also depend on Cloudflare's platform rules and the response headers returned by the origin.

## Security Best Practices

- Never commit B2 application keys or Cloudflare API tokens.
- Store `B2_ACCOUNTS` as an encrypted Cloudflare Worker secret.
- Keep `.dev.vars` and other local credential files out of Git.
- Use a Cloudflare API token with only the permissions necessary for deployment and secret management.
- Rotate credentials if they are accidentally exposed.
- Do not publish private bucket credentials or sensitive object URLs in public issues.
- Review your Worker routes and access controls before exposing private files publicly.

## Troubleshooting

### Missing `B2_ACCOUNTS`

Confirm that the secret exists for the same Worker configured in `wrangler.toml`.

### Incorrect account or bucket

Check the account identifier, bucket name, endpoint, region, and application key.

### Cache duration does not change

Confirm that `CACHE_TTL` is configured as a regular variable, has the intended numeric value, and is deployed to the correct Worker.

### Access denied or authentication errors

Verify the B2 application key permissions and S3-compatible endpoint configuration.

### Changes are not reflected after deployment

Ensure that you are deploying from the correct repository directory and that the Worker name in `wrangler.toml` matches the intended deployment.

## License

Choose a license before distributing this project. If you intend to allow others to use, modify, and redistribute the code, consider adding an appropriate open-source license file.
